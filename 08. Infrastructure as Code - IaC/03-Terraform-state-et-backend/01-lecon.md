# Leçon 3 — Le state : le registre vital de Terraform

> **Bloc 8 · Infrastructure as Code (IaC)** — Leçon 3 sur 9
> 🧭 **Pont depuis la Leçon 2** : tu as exécuté le workflow complet (`init`, `plan`, `apply`, `destroy`) et tu as croisé un fichier que tu n'as jamais écrit : `terraform.tfstate`. La Leçon 2 l'a mentionné, sa checklist t'a prévenu(e) : « Terraform "oublie" ce qu'il gère si on le perd ». Cette leçon explique **pourquoi** ce fichier est vital, **comment** Terraform s'en sert, ce qui le rend **dangereux** (secrets, collisions), et comment le déporter dans un **backend distant** — le standard professionnel.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Expliquer** le rôle du **state** dans le cycle déclaratif de Terraform (code ↔ state ↔ réel).
2. **Inspecter** le state avec les commandes `terraform state list`, `terraform state show` et `terraform show`.
3. **Observer** le drift en action : détruire une ressource à la main et voir le plan la détecter.
4. **Expliquer** pourquoi le state local ne convient pas à une équipe, et **décrire** un backend distant (S3 + verrou).
5. **Protéger** le state : hors de Git, sauvegardé, chiffré, verrouillé.

---

## 2. Explication simple

### 2.1 Le « pourquoi » : Terraform a besoin de la mémoire

Reprends le cycle de la Leçon 2 (§ 2.3) : Terraform compare **ton code** et **la réalité**, puis calcule les écarts. Mais il lui manque une pièce : **comment sait-il quelles ressources sont « les siennes » ?** Le cloud contient peut-être 50 machines virtuelles ; ton code n'en décrit qu'une. Sans mémoire, Terraform ne saurait pas laquelle est à lui.

Le **state** est cette mémoire : un fichier (par défaut `terraform.tfstate`) où Terraform note **ce qu'il a créé et où le trouver** — avec, pour chaque ressource, l'**identifiant réel** chez le provider (l'ID du fichier, de la VM, du réseau…).

> 💡 **Analogie** : le state est le **registre du chantier**. Le plan d'architecte (ton code) dit ce qu'on veut construire ; le registre du chantier note : « la maison 3 a été construite avec le permis n° 42B, livrée le 12 mars ». Si un jour on veut la modifier ou la démolir, on ouvre le registre pour savoir **de quelle maison on parle**.

**Trois rôles concrets du state :**
1. **La cartographie** : relier chaque bloc `resource` de ton code à l'objet réel (par son ID).
2. **Le cache des attributs** : certaines valeurs produites par le provider (un ID, un résultat aléatoire) y sont stockées, pour ne pas les regénérer à chaque fois.
3. **La détection du drift** : en comparant registre ↔ réalité, Terraform repère ce qui a été modifié à la main.

### 2.2 Le « comment » : à quoi ressemble le state ?

Le state est un fichier au format **JSON** (le format de données texte vu au Bloc 3, Leçon 4) : une liste de ressources, chacune avec ses attributs réels. Tu peux le lire avec `terraform show`, qui le présente de façon lisible. Tu ne l'**édites jamais à la main** : Terraform seul y écrit, et le corrompre = perdre la mémoire du projet.

**La détection du drift, concrètement** : à chaque `plan` (et chaque `apply`), Terraform « rafraîchit » son registre — il **interroge la réalité** et met à jour le state avec ce qu'il trouve. Si un fichier créé par Terraform a été modifié ou supprimé à la main, le plan le détecte et propose de le remettre en conformité. Tu vas le vérifier par toi-même dans cette leçon.

### 2.3 Le problème du state local : pourquoi il ne suffit plus

Par défaut, le state vit **dans ton dossier**, à côté du code. Pour apprendre en solo, c'est parfait. Mais dès que le projet devient sérieux, trois problèmes apparaissent :

| Problème | Ce qui se passe | Conséquence |
|----------|-----------------|-------------|
| **Pas partagé** | Ton collègue a son propre `terraform.tfstate` | Il « voit » une infrastructure vide → son apply **recrée des doublons** |
| **Pas verrouillé** | Deux personnes (ou deux pipelines) appliquent **en même temps** | Le state est corrompu ou incomplet → ressources perdues pour Terraform |
| **Pas sauvegardé** | Tu perds le fichier (disque, nettoyage) | Terraform « oublie » tout ce qu'il a créé → ressources **orphelines** impossible à gérer sans réparer le state |

Et un quatrième, plus sournois : le state contient les **valeurs réelles** des ressources — y compris les secrets (un mot de passe généré y est stocké **en clair**). Le pousser dans Git = publier tes secrets dans l'historique du dépôt. C'est pourquoi la Leçon 2 t'a conseillé de le mettre hors de Git via `.gitignore`.

### 2.4 Le « comment » (suite) : le backend distant

La solution aux quatre problèmes : **déporter le state** dans un emplacement partagé et surveillé — c'est le **backend**. Terraform propose plusieurs backends ; le standard avec AWS est :

- **Un compartiment S3** (le stockage objet d'AWS, Bloc 6 Leçon 4) pour héberger le fichier de state : partagé, **chiffré**, avec versionnage (chaque ancien état est conservé).
- **Une table DynamoDB** (une base clé-valeur d'AWS) ou, dans les versions récentes de Terraform, un **fichier de verrou** géré par le backend : pour le **verrouillage** — pendant un `apply`, le state est verrouillé, personne d'autre ne peut l'écrire en même temps.

```
Avant (apprentissage)              Après (équipe / production)
ton dossier/                       ton code (dans Git)
├── main.tf                        backend → S3 (state partagé, chiffré,
└── terraform.tfstate              versionné) + verrou (DynamoDB)
```

> 🔑 **À retenir** : le code, lui, reste **dans Git**. Seul le **state** va dans le backend. On ne partage jamais les deux dans le même endroit : Git versionne le code, S3 héberge l'état.

**Le « quand »** : le state local suffit tant que tu es seul(e) et en apprentissage (c'est notre cas dans ce bloc). Dès qu'un **deuxième humain** ou un **pipeline CI/CD** (Bloc 11) touche au projet, passe au backend distant. Tu écriras la configuration du backend dans cette leçon, et tu l'utiliseras réellement en Leçon 5 quand tu auras des clés AWS.

---

## 📖 Vocabulaire / Abréviations

| Terme | Définition (une ligne) | Où |
|-------|------------------------|-----|
| **State** | Le registre (`terraform.tfstate`) de ce que Terraform a créé, avec les ID réels | § 2.1 |
| **Rafraîchissement** | La lecture du réel par Terraform, faite à chaque plan/apply | § 2.2 |
| **Ressource orpheline** | Une ressource réelle que le state ne connaît plus : ingérable sans réparation | § 2.3 |
| **Backend** | L'emplacement où vit le state (local par défaut, S3 à distance) | § 2.4 |
| **S3** | Le stockage objet d'AWS — héberge le state distant (Bloc 6 Leçon 4) | § 2.4 |
| **DynamoDB** | La base clé-valeur d'AWS — sert de verrou au state | § 2.4 |
| **Verrouillage (lock)** | Pendant un apply, le state est verrouillé contre les écritures concurrentes | § 2.4 |
| **Chiffrement au repos** | Le fichier est chiffré « sur le disque » chez le provider | § 3.4 |
| **`terraform state pull`** | Commande qui affiche/charge le state (utile pour sauvegarder) | § 3.3 |
| **`sensitive = true`** | Option d'output qui masque la valeur à l'écran (mais PAS dans le state) | § 3.4 |

---

## 3. Exemples concrets

La théorie est posée ; passons à la manipulation du registre. On reprend le projet `atelier-securise` de la Leçon 2 — **toutes les commandes sont commentées ligne par ligne**.

### 3.1 Ouvrir le registre

```bash
# Retourne dans le projet de la Leçon 2.
cd ~/atelier-securise

# Liste les ressources gérées par Terraform (le registre).
terraform state list
```

Sortie attendue :

```
local_file.journal
random_string.secret
```

```bash
# Détails d'une ressource précise : adresse "random_string.secret".
terraform state show random_string.secret
```

Sortie (extrait) : le résultat aléatoire, la longueur, l'ID réel du fichier… c'est la **cartographie** de § 2.1. Enfin :

```bash
# Affiche tout le state, de façon lisible (traduit le JSON brut).
terraform show
```

### 3.2 Le drift en action : expérience vécue

C'est l'expérience clé : jouer le rôle du « collègue négligent », puis observer.

```bash
# Supprime À LA MAIN le fichier que Terraform a créé (le drift !).
rm journal-securise.txt

# Demande le plan : Terraform rafraîchit le registre, interroge le réel…
terraform plan
```

Le plan détecte l'écart et propose la **réparation** : `Plan: 1 to add, 0 to change, 0 to destroy.` — le fichier va être **recréé**. Comment Terraform le sait-il ? Grâce au state : il sait qu'une ressource `local_file.journal` existe dans son registre, il va vérifier le réel, constate que le fichier n'existe plus, et propose de le remettre en conformité avec le code.

```bash
# Applique la réparation (tape yes).
terraform apply

# Vérifie : le fichier est revenu, identique.
ls journal-securise.txt
```

> 🔑 **C'est le superpouvoir de l'IaC** : peu importe ce qu'on a cassé à la main, le code + le state **réparent**. Retiens ce trio : code (ce que je veux) → state (ce que je gère) → réel (ce qui existe).

### 3.3 Sauvegarder le registre

```bash
# Sauvegarde manuelle : "state pull" charge le state, on le redirige (> ) dans un fichier daté.
# $(date +%F) insère la date du jour (ex. 2026-09-11) — réflexe vu au Bloc 3.
terraform state pull > sauvegarde-state-$(date +%F).tfstate

# Vérifie que la sauvegarde existe.
ls -lh sauvegarde-state-*
```

Ce réflexe manuel est utile, mais il reste **manuel** — d'où l'intérêt du backend distant, qui versionne automatiquement chaque état.

### 3.4 Le backend distant : la configuration (préparée, activée en Leçon 5)

```hcl
# backend.tf — déclare où vit le state.
terraform {
  backend "s3" {
    bucket         = "MON-NOM-unique-de-compartiment"     # le compartiment S3 qui hébergera le state
    key            = "atelier-securise/terraform.tfstate" # le chemin du fichier dans le compartiment
    region         = "eu-west-3"                          # ta région AWS
    dynamodb_table = "terraform-locks"                    # la table qui gère le verrou
    encrypt        = true                                 # chiffre le state au repos (il contient des secrets !)
  }
}
```

Quand tu l'activeras (Leçon 5), la commande de migration sera :

```bash
# terraform init redétecte la config et demande la migration du state vers le backend.
terraform init
# → "Do you wish to proceed? ... Enter a value:" → tape yes
```

Un dernier mot sur les **outputs sensibles** : dans la Leçon 2, ton output `secret_genere` affichait la valeur. En production, on ajoute `sensitive = true` dans le bloc `output` : la valeur est alors masquée à l'écran (`<sensitive>`) — mais attention, elle reste **en clair dans le state**. Le masque protège l'affichage, pas le stockage : encore une raison de chiffrer le backend.

---

## 4. Bonnes pratiques modernes (2025-2026)

- **State distant dès que l'équipe grandit** : S3 + verrou est le standard AWS ; d'autres backends existent (Azure Blob, Google Cloud Storage, Terraform Cloud/HCP) — le principe (partagé, chiffré, verrouillé) reste le même.
- **Toujours `encrypt = true`** sur le backend : le state contient des secrets en clair.
- **Versionnage du bucket S3 activé** : chaque ancien état est conservé — si un apply corrompt le state, on restaure la version précédente (rappel : « un backup non testé n'est pas un backup », Bloc 7).
- **`.gitignore` systématique** dans tout projet Terraform : `.terraform/`, `*.tfstate`, `*.tfstate.*` (et les fichiers `.tfvars` contenant des secrets, tu les verras en Leçon 4).
- **Ne jamais éditer le state à la main** : les commandes `terraform state` (list, show, mv, rm, pull) existent pour intervenir proprement ; l'édition au couteau suisse corrompt le JSON.
- **Un state par projet (et par environnement)** : on ne partage pas un state entre la dev et la prod (Leçon 4) — une erreur en dev ne doit jamais pouvoir détruire la prod.

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi c'est dangereux/inefficace | ✅ Version correcte |
|-----------------|--------------------------------------|---------------------|
| Pousser `terraform.tfstate` dans Git | Secrets **en clair dans l'historique**, conflits, corruption | `.gitignore` + backend distant chiffré |
| Deux personnes qui `apply` en même temps (state local) | State corrompu, ressources perdues pour Terraform | Backend distant + **verrouillage** |
| Supprimer le state « pour repartir de zéro » | Les ressources réelles deviennent **orphelines** : il faut tout recréer/réparer | Garder le state, le sauvegarder, utiliser le versionnage du backend |
| Éditer le state avec un éditeur de texte | Un JSON mal formé = mémoire perdue | Commandes `terraform state …` officielles |
| Un seul state partagé entre dev et prod | Une erreur en dev peut **détruire la prod** (même registre !) | Un state (backend path) **par environnement** |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`**, la correction dans **`03-correction.md`**.

**Énoncé court** : sur le projet de la Leçon 2 — inspecte le registre (`state list`, `state show`, `show`), **provoque un drift** (supprime le fichier à la main) et regarde le plan le détecter et le réparer, sauvegarde le state avec `state pull`, écris le `.gitignore`, et prépare la configuration du backend S3 (sans l'activer).

---

## 7. Correction détaillée de l'exercice

> La correction complète est dans **`03-correction.md`** : sorties attendues, explication du mécanisme de détection du drift, commentaires des lignes `.gitignore` et lecture du `backend.tf`.

---

## 8. Checklist de validation

- [ ] J'explique le rôle du state dans le cycle code ↔ state ↔ réel.
- [ ] Je sais inspecter le state (`state list`, `state show`, `show`) sans jamais l'éditer à la main.
- [ ] J'ai provoqué un drift et vu le plan le détecter puis le réparer.
- [ ] Je liste les 4 problèmes du state local (partage, verrou, sauvegarde, secrets).
- [ ] Je décris un backend S3 + verrou et je sais pourquoi le state ne va jamais dans Git.
- [ ] J'ai un `.gitignore` correct dans mon projet Terraform.

---

🧭 **Pont vers la suite** — Ton registre est protégé et ton code est propre. Mais regarde ton `main.tf` : si tu veux la même infrastructure en `dev` et en `prod`, tu vas **copier-coller** 40 lignes… et te corriger deux fois à chaque changement. Le remède des développeurs (fonctions, composants réutilisables) existe en Terraform : ce sont les **modules** — et leur complément naturel, l'organisation par **environnements**. C'est la **Leçon 4**.

---

*Prochaine étape :* Leçon 4 — **Modules et environnements : organiser son code** dans `04-Terraform-modules-et-environnements/`.