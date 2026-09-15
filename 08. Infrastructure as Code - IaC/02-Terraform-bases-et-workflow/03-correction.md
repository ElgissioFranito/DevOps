# Correction — Leçon 2 : Terraform, ton premier projet

> **Bloc 8 · Leçon 2** — Correction pas à pas.

---

## Étape 1 — Nouveau projet

```bash
# mkdir = crée le dossier ; && = exécute la commande suivante seulement si la 1re réussit.
mkdir ~/atelier-securise && cd ~/atelier-securise
```

## Étape 2 — Les 4 fichiers

### `versions.tf`

```hcl
# Réglages du projet : version de Terraform + liste des providers.
terraform {
  required_version = ">= 1.5.0"    # toute version à partir de 1.5.0

  required_providers {
    local = {
      source  = "hashicorp/local"  # provider des fichiers locaux
      version = "~> 2.0"           # 2.x, jamais 3.x
    }
    random = {
      source  = "hashicorp/random" # provider des valeurs aléatoires
      version = "~> 3.0"
    }
  }
}
```

### `variables.tf`

```hcl
variable "nom_projet" {
  description = "Nom du projet, réutilisé dans les fichiers créés"
  type        = string          # une chaîne de caractères
  default     = "securise"      # utilisée si on ne fournit rien
}

variable "longueur_secret" {
  description = "Longueur du secret aléatoire"
  type        = number          # un nombre
  default     = 20
}

variable "activer_journal" {
  description = "Créer le fichier journal ?"
  type        = bool            # true ou false
  default     = true
}
```

**Explication** : le type `bool` est nouveau par rapport à la leçon — il ne peut valoir que `true` ou `false`. On l'utilise avec un bloc `count` ou une condition ; ici on l'explorera simplement dans le contenu du journal.

### `main.tf`

```hcl
# Providers déclarés vides : aucun réglage spécial nécessaire.
provider "local" {}
provider "random" {}

# Ressource 1 : le secret aléatoire.
resource "random_string" "secret" {
  length  = var.longueur_secret   # lit la variable (20 par défaut)
  special = true                  # caractères spéciaux AUTORISÉS (contrairement à la leçon)
}

# Ressource 2 : le fichier journal.
resource "local_file" "journal" {
  # ${...} = interpolation ; \n = retour à la ligne ; var.X = variable.
  content  = <<-EOT               # <<-EOT : texte sur plusieurs lignes (fini par EOT seul)
    Projet        : ${var.nom_projet}
    Longueur      : ${var.longueur_secret}
    Secret        : ${random_string.secret.result}
    Journal actif : ${var.activer_journal}
  EOT
  filename = "journal-${var.nom_projet}.txt"
}
```

**Explications des choix techniques** :
- `<<-EOT … EOT` (le **heredoc**) : une syntaxe Terraform pour écrire un texte **sur plusieurs lignes** sans répéter `\n`. Le nom après `<<-` est libre (`EOT` = *End Of Text*, par convention) ; le bloc se termine par cette même marque seule sur sa ligne. C'est plus lisible que des `\n` partout — et en Leçon 7 (Ansible), tu reverras cette idée pour les templates.
- `${random_string.secret.result}` : on **interpole la sortie d'une autre ressource**. C'est le principe du « maillon » : une ressource produit une valeur, une autre la consomme.
- Le nom du fichier dépend de la variable `nom_projet` : changer la variable **change le nom du fichier**, donc Terraform détruira l'ancien et créera le nouveau (tu l'as vu au plan : `1 to add, 1 to destroy`).

### `outputs.tf`

```hcl
# Info affichée à la fin de l'apply : le secret généré.
output "secret_genere" {
  description = "Le secret aléatoire généré"
  value       = random_string.secret.result
}

# Info : le chemin du fichier journal.
output "chemin_journal" {
  description = "Le chemin du fichier journal créé"
  value       = local_file.journal.filename
}
```

---

## Étape 3 — Le rituel complet, avec les sorties attendues

```bash
terraform init     # "Terraform has been successfully initialized!"
terraform plan     # Plan: 2 to add, 0 to change, 0 to destroy.
terraform apply    # tape yes → "Resources: 2 added, 0 changed, 0 destroyed."
terraform apply    # "No changes. Your infrastructure matches the configuration."
```

**Explications** :
- Le **premier apply** crée les 2 ressources (le secret et le journal) et affiche tes 2 outputs à la fin.
- Le **re-apply** affiche `No changes` : c'est l'**idempotence** (Leçon 1) vérifiée en pratique — relancer ne duplique rien.
- Après le changement de `longueur_secret` (20 → 30), le plan montre `~ random_string.secret will be updated in-place` : le secret est **regénéré sur place**, et le journal suit (son contenu dépend du secret). Résumé typique : `Plan: 0 to add, 2 to change, 0 to destroy.` Le fichier **garderait son nom** ici (on n'a pas changé `nom_projet`), donc pas de destruction.

```bash
terraform destroy  # tape yes → "Resources: 2 destroyed."
```

**Explication** : `destroy` supprime **tout ce que ce projet a créé** (les 2 ressources), mais **ne touche jamais à ton code** ni aux fichiers `.tf`. D'où le critère du bloc : *supprimer l'infrastructure et la reconstruire depuis le code* — tu peux le vérifier : relance `apply` et tout revient.

## Étape 4 — Questions de lecture de plan

1. `~ ... updated in-place` → la ressource **existe déjà** ; Terraform va **modifier certaines propriétés sans la supprimer** (comme repeindre une pièce sans la démolir).
2. `- ... will be destroyed` → la ressource sera **supprimée** (ex. : son nom dépendait d'une variable modifiée, ou tu l'as retirée du code).
3. `+ ... will be created` → la ressource **n'existe pas encore** ; elle sera créée.

> 🔑 **Réflexe à graver** : avant chaque `apply`, lis le nombre de `-` (destroy). Un `destroy` imprévu = ressource qui va disparaître = toujours une alerte à éclaircir.

---

## Checklist de validation (leçon 2)

- [ ] Je crée un projet Terraform et j'explique le rôle de chaque bloc (`terraform`, `provider`, `resource`, `variable`, `output`).
- [ ] J'exécute le workflow complet `init` → `plan` → `apply` → `destroy` et j'explique chaque étape.
- [ ] Je lis une sortie de `terraform plan` et je connais les symboles `+`, `~`, `-`.
- [ ] J'ai **vérifié l'idempotence** : un re-apply sans changement affiche `No changes`.
- [ ] Je sais ce que sont `terraform.tfstate`, `.terraform/` et `terraform.tfstate.backup`.
- [ ] Je modifie une variable et le plan montre une mise à jour **ciblée**, pas un chaos.

---

## 🧠 Conseils pour la suite

- **Refais l'exercice de mémoire** demain sans regarder la correction : c'est le meilleur test. La mécanique `init/plan/apply/destroy` doit devenir réflexe, car la Leçon 5 l'appliquera au cloud réel où les erreurs coûtent.
- **`-auto-approve` : pas à la main.** Dans les corrections futures, tu verras l'option apparaître dans des pipelines — mais toujours après un plan déjà validé.
- **Ne versionne pas le state** : ajoute dès maintenant `.terraform/` et `*.tfstate*` dans un fichier `.gitignore` (rappel du Bloc 4 : le fichier qui liste ce que Git doit ignorer). La Leçon 3 explique **pourquoi** et **où** mettre le state à la place.
