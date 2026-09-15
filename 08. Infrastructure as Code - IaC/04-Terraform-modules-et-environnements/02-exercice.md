# Exercice — Leçon 4 : modules et environnements

> **Bloc 8 · Leçon 4** — Exercice en autonomie, **100 % local et gratuit** : tu transformes le code de la Leçon 2 en **module réutilisable**, puis tu le déploies en deux environnements (`dev` et `prod`) avec des réglages distincts.

---

## Contexte

Jusqu'ici, ton code n'existait qu'en un seul exemplaire. Pour préparer la vraie infrastructure de la Leçon 5 (réseau + machine + base en dev ET en prod), il faut savoir **réutiliser** et **paramétrer**. L'exercice reconstruit le projet `atelier-securise` sous la forme professionnelle : 1 module + 2 environnements.

---

## Énoncé

> 📌 **Rappels d'options** : `mkdir -p a/b` = créer le dossier `a/b` (et ses parents) ; `terraform apply -var-file=FICHIER` = appliquer avec les valeurs de ce fichier ; `terraform destroy -var-file=FICHIER` = détruire en tenant compte de ce fichier.

### Étape 1 — Construire l'arborescence

```bash
# Nouveau dossier projet, avec la structure module + environnements.
mkdir -p ~/organise/modules/journal ~/organise/envs/dev ~/organise/envs/prod

# Entre dans le dossier du module et copie les fichiers de la Leçon 2 comme point de départ.
cd ~/organise/modules/journal
cp ~/atelier-securise/versions.tf .
```

### Étape 2 — Écrire le module `journal`

Dans `~/organise/modules/journal/`, crée (inspire-toi de la Leçon 2, mais **renomme les variables**) :

1. `variables.tf` : deux variables — `nom_env` (string, **sans défaut** : l'appelant est obligé de la fournir) et `taille_secret` (number, défaut `10`).
2. `main.tf` : la resource `random_string` (longueur = `taille_secret`, `special = true`) et la `local_file` : fichier `journal-<nom_env>.txt`, contenu sur 3 lignes (nom d'env, taille, secret).
3. `outputs.tf` : deux outputs — `chemin` (le chemin du fichier) et `secret` (marqué `sensitive = true`).

> 💡 **Astuce** : dans un module, on ne déclare **pas** de bloc `terraform { required_providers … }` avec les sources : les providers sont déclarés **une fois dans le projet racine**. (Une version minimale `required_version` reste possible.)

### Étape 3 — Appeler le module depuis `dev` et `prod`

Dans `~/organise/envs/dev/`, crée `main.tf` :

```hcl
# modules/journal : APPEL du module (comme un appel de fonction).
module "journal" {
  source        = "../../modules/journal"   # où trouver le module (chemin relatif)
  nom_env       = "dev"                     # argument n°1
  taille_secret = 8                         # argument n°2 : un secret court pour la dev
}
```

Puis dans `~/organise/envs/prod/`, le même appel avec `nom_env = "prod"` et `taille_secret = 32`.

Enfin, copie le `versions.tf` de l'étape 1 dans **chaque** environnement (`envs/dev/` et `envs/prod/`) — c'est le projet racine qui déclare les providers (rappel Leçon 3 : chaque environnement a SON state, donc son propre dossier de projet).

### Étape 4 — Les `.tfvars` (et la règle des secrets)

Dans `envs/dev/`, crée `dev.tfvars` :

```hcl
taille_secret = 8          # valeur fournie sans toucher au code
```

Puis **applique les deux environnements** :

```bash
cd ~/organise/envs/dev
terraform init
terraform apply -var-file="dev.tfvars"    # tape yes

cd ../prod
terraform init
terraform apply    # ici pas de tfvars : les défauts du code sont utilisés
```

Vérifie : deux fichiers `journal-dev.txt` et `journal-prod.txt`, avec des secrets de longueurs différentes.

```bash
# Démonte la dev proprement (la prod reste debout !).
cd ../dev && terraform destroy -var-file="dev.tfvars"   # tape yes
```

### Étape 5 — Vérifier l'isolement des états

```bash
# Depuis envs/dev : le registre est vide (on a détruit la dev).
cd ~/organise/envs/dev && terraform state list

# Depuis envs/prod : la prod est toujours là.
cd ../prod && terraform state list
```

Questions (dans `notes-exercice-04.md`) :
1. Pourquoi chaque environnement a-t-il son propre registre ? Quel risque couvert ?
2. Où placer le `.gitignore` pour qu'il couvre tout le dépôt `~/organise` ?
3. Ton output `secret` est marqué `sensitive = true` : où la valeur du secret apparaît-elle **quand même** en clair ? (Réponse en Leçon 3…)

---

## Livrable

- L'arborescence complète `~/organise/` (module + 2 environnements), fonctionnelle.
- `notes-exercice-04.md` : les 3 réponses + la sortie des deux `state list`.

Correction détaillée dans **`03-correction.md`**.