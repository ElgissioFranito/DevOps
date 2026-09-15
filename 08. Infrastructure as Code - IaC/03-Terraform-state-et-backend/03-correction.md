# Correction — Leçon 3 : le state et son backend

> **Bloc 8 · Leçon 3** — Correction pas à pas.

---

## Étape 1 — Préparer le terrain

```bash
cd ~/atelier-securise
terraform apply    # tape yes si besoin
```

**Explication** : si tu avais lancé un `destroy` à la fin de la Leçon 2, le registre est vide et le `apply` recrée les ressources. Si le projet est déjà appliqué, la sortie est `No changes` (idempotence, Leçon 2).

## Étape 2 — Ouvrir le registre

```bash
terraform state list
# local_file.journal
# random_string.secret
```

**Réponse Q1** : 2 ressources — exactement celles de `main.tf`.

```bash
terraform state show random_string.secret
```

**Réponse Q2** : la valeur réelle du secret apparaît dans l'attribut `result` du bloc `random_string.secret` affiché. C'est le **cache des attributs** (§ 2.1) : Terraform garde le résultat pour ne pas le regénérer à chaque apply — c'est aussi pour ça qu'un re-apply ne change pas la valeur.

```bash
cat terraform.tfstate | head -30
```

**Réponse Q3** : c'est du **JSON** — des accolades `{ }`, des paires `"clé": valeur`. (Le `| head -30` n'affiche que les 30 premières lignes — réflexe du Bloc 2.)

## Étape 3 — Provoquer et détecter le drift

```bash
rm journal-securise.txt
terraform plan
```

Sortie attendue : `Plan: 1 to add, 0 to change, 0 to destroy.` (avec le détail `+ local_file.journal will be created`).

**Réponse à « comment Terraform sait-il ? »** : le state dit « je gère `local_file.journal`, ID = journal-securise.txt ». Le **rafraîchissement** interroge le réel : le fichier n'existe plus. Écart détecté → le plan propose de **recréer** la ressource pour rattraper le code. Trois enseignements :

1. Le plan ne lit pas seulement ton code : il lit **le réel**, via le state.
2. Une modification à la main n'est jamais invisible : elle crée du **drift** que le prochain plan expose.
3. L'apply « répare » : le code est la source de vérité.

```bash
terraform apply    # tape yes → le fichier revient
ls journal-securise.txt   # présent à nouveau
```

> ⚠️ Nuance attendue : la recréation régénère le contenu (le secret du fichier peut différer de l'ancien si la ressource est recréée). C'est le comportement du provider : quand une ressource « meurt » de façon non contrôlée, on la reconstruit. Sur des données de production, on ne peut pas se le permettre — d'où les sauvegardes (Bloc 7) et le verrouillage du state.

---

## Étape 4 — Sauvegarder et protéger

```bash
terraform state pull > sauvegarde-state-$(date +%F).tfstate
ls -lh sauvegarde-state-*
```

**Explication** : `state pull` affiche le state brut ; la redirection `>` l'écrit dans un fichier dont le nom contient la date ($(date +%F) → ex. 2026-09-11). C'est la sauvegarde **manuelle** ; le backend distant la rend automatique (versionnage).

Le `.gitignore` — trois lignes, chacune commentée :

```
.terraform/     # les plugins téléchargés : volumineux, réinstallables (init)
*.tfstate       # le state : SECRETS en clair + ne doit jamais diverger entre Git et le réel
*.tfstate.*     # la sauvegarde automatique (terraform.tfstate.backup) et le fichier de verrou
```

**Réponses** : `.terraform/` = inutile dans Git (reproductible par `init`, et très lourd) ; `*.tfstate` = secrets en clair + le state ne doit pas être partagé via Git (c'est le rôle du backend) ; `*.tfstate.*` = couvre les fichiers dérivés (backup, verrou).

## Étape 5 — Le backend S3 (écrit, pas activé)

Chaque ligne de `backend.tf` :

| Ligne | Rôle |
|-------|------|
| `backend "s3"` | « le state vit dans S3 » (et non plus en local) |
| `bucket` | le **compartiment** S3 hébergeur — son nom doit être unique mondialement (Bloc 6, Leçon 4) |
| `key` | le « chemin » du fichier dans le compartiment — **un chemin par projet/environnement** |
| `region` | la région AWS où vit le compartiment |
| `dynamodb_table` | la table qui **verrouille** le state pendant les apply |
| `encrypt = true` | chiffre le state au repos — obligatoire, il contient des secrets |

**Questions de synthèse** :

1. **Deux apply simultanés avec state local** : chaque exécution lit et réécrit le fichier en même temps → le state est corrompu ou incomplet ; des ressources deviennent fantômes. **Avec backend + verrou** : le second apply attend (`Acquiring state lock…`) ou échoue proprement — les deux exécutions ne se marchent jamais dessus.
2. **State jamais dans Git** — deux raisons : (a) il contient des **secrets en clair** (même supprimé ensuite, il reste dans l'historique Git — rappel du Bloc 4 : l'historique ne s'efface pas) ; (b) deux collègues qui poussent/tirent le state via Git finissent par le **divorcer du réel** (conflits de fusion, états désynchronisés) — le backend avec verrou règle les deux problèmes.

---

## Checklist de validation (leçon 3)

- [ ] J'explique le rôle du state dans le cycle code ↔ state ↔ réel.
- [ ] Je sais inspecter le state (`state list`, `state show`, `show`) sans jamais l'éditer à la main.
- [ ] J'ai provoqué un drift et vu le plan le détecter puis le réparer.
- [ ] Je liste les 4 problèmes du state local (partage, verrou, sauvegarde, secrets).
- [ ] Je décris un backend S3 + verrou et je sais pourquoi le state ne va jamais dans Git.
- [ ] J'ai un `.gitignore` correct dans mon projet Terraform.

---

## 🧠 Conseils pour la suite

- **Habitude à prendre** : après chaque projet Terraform de ce bloc, vérifie que `.gitignore` contient bien les 3 lignes avant le premier `git add`.
- Le `backend.tf` que tu viens d'écrire sera **réutilisé tel quel** en Leçon 5 — garde-le précieusement.
- La Leçon 4 (modules et environnements) s'appuie sur tout ce que tu sais déjà : le code de la Leçon 2 deviendra un module réutilisable, et la séparation dev/prod s'appuiera sur la règle « un state par environnement » vue ici.