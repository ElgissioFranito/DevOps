# Référence rapide — Leçon 2 : Terraform, workflow et blocs HCL

> Bloc 8 · Leçon 2 — Aide-mémoire.

## Les 4 commandes du workflow

| Commande | Rôle | Quand |
|----------|------|-------|
| `terraform init` | Télécharge les providers, prépare le dossier | 1 fois, et à chaque ajout de provider |
| `terraform plan` | Prévisualise les changements (`+` créer, `~` modifier, `-` supprimer) | **Avant chaque apply** |
| `terraform apply` | Exécute (confirmation `yes`) | Après avoir lu le plan |
| `terraform destroy` | Supprime tout ce que le projet a créé | Pour démonter |

```bash
terraform init
terraform plan
terraform apply      # tape yes
terraform destroy    # tape yes
```

- `terraform apply -auto-approve` : saute la confirmation — **réservé aux pipelines CI/CD**.
- `terraform show` : affiche le contenu du state en lisible.
- `terraform state list` : liste les ressources gérées.

## Les blocs HCL (analogie : maquette de ville)

```hcl
# La notice : version de Terraform + providers à télécharger.
terraform {
  required_version = ">= 1.5.0"
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.0"   # ~> = cette version ou une mineure supérieure
    }
  }
}

# La plateforme (le "traducteur").
provider "local" {}

# Un bouton de réglage.
variable "nom_projet" {
  description = "Nom du projet"
  type        = string
  default     = "atelier"
}

# Une brique concrète à créer.
resource "local_file" "carte" {
  content  = "Projet : ${var.nom_projet}"   # interpolation : insère la valeur
  filename = "carte-${var.nom_projet}.txt"
}

# Une étiquette d'infos à la fin de l'apply.
output "fichier_cree" {
  value = local_file.carte.filename
}
```

## Fichiers du projet

| Fichier | Rôle | Git ? |
|---------|------|-------|
| `versions.tf` | versions + providers | ✅ oui |
| `main.tf` | resources | ✅ oui |
| `variables.tf` | variables | ✅ oui |
| `outputs.tf` | outputs | ✅ oui |
| `.terraform/` | plugins téléchargés | ❌ non |
| `terraform.tfstate` | le state (registre) | ❌ non — protégé (Leçon 3) |
| `terraform.tfstate.backup` | copie de secours du state | ❌ non |

## Rituels de sécurité

- Toujours **`plan` → lecture → `apply`**.
- Ne jamais éditer à la main ce que Terraform gère (**drift**).
- Ne jamais éditer/supprimer `terraform.tfstate` à la main.