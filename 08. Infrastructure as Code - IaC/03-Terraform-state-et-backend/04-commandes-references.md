# Référence rapide — Leçon 3 : le state et son backend

> Bloc 8 · Leçon 3 — Aide-mémoire.

## Le cycle déclaratif (le rôle du state)

```
Code (ce que je veux) → State (ce que je gère) → Réel (ce qui existe)
                              ↑ rafraîchi à chaque plan/apply
```

## Inspecter le state (jamais l'éditer à la main !)

```bash
terraform state list                  # liste des ressources gérées
terraform state show random_string.secret   # détails d'une ressource
terraform show                        # tout le state, lisible
terraform state pull > sauvegarde-state-$(date +%F).tfstate   # sauvegarde manuelle
```

## Les 4 problèmes du state local

| Problème | Conséquence |
|----------|-------------|
| Pas partagé | Doublons à chaque collègue |
| Pas verrouillé | Corruption si apply simultanés |
| Pas sauvegardé | Ressources orphelines si perdu |
| Secrets en clair | Fuite si versionné dans Git |

## .gitignore d'un projet Terraform

```
.terraform/
*.tfstate
*.tfstate.*
```

## Backend S3 (activé en Leçon 5)

```hcl
terraform {
  backend "s3" {
    bucket         = "MON-NOM-unique-de-compartiment"
    key            = "projet/terraform.tfstate"  # un chemin PAR environnement
    region         = "eu-west-3"
    dynamodb_table = "terraform-locks"           # le verrou
    encrypt        = true                        # secrets en clair dans le state !
  }
}
```

```bash
terraform init   # après ajout du backend → propose la migration → tape yes
```

## Règles d'or

- Le **code** va dans Git ; le **state** va dans le backend. Jamais l'inverse, jamais ensemble.
- `sensitive = true` masque l'**affichage**, pas le **stockage** dans le state.
- Un state par projet **et par environnement**.