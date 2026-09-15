# Référence rapide — Leçon 4 : modules et environnements

> Bloc 8 · Leçon 4 — Aide-mémoire.

## Arborescence type d'un dépôt Terraform organisé

```
depot/
├── modules/
│   └── journal/           ← module réutilisable (variables, main, outputs)
└── envs/
    ├── dev/               ← 1 projet racine par environnement
    ├── staging/
    └── prod/
```

## Appeler un module

```hcl
module "journal" {
  source        = "../../modules/journal"   # chemin relatif (ou Git + ?ref=v1.2.0)
  nom_env       = "dev"      # variable SANS default → obligatoire
  taille_secret = 8          # variable avec default → facultative
}
```

- Le module déclare **variables** (paramètres) et **outputs** (retours) — comme une fonction.
- Les `provider` se déclarent dans le **projet racine**, pas dans le module.
- Adresse des ressources d'un module : `module.journal.random_string.secret`.

## Valeurs par environnement

```hcl
# dev.tfvars (hors Git si secrets)
taille_secret = 8
```

```bash
terraform apply -var-file="dev.tfvars"   # appliquer avec ces valeurs
```

Alternatives aux secrets dans le code : variables d'environnement `TF_VAR_*`, `.tfvars` hors dépôt, gestionnaires de secrets (AWS Secrets Manager, Vault).

## Séparation des environnements — les 3 niveaux

| Niveau | Ce qui est séparé | Pourquoi |
|--------|-------------------|----------|
| Code | `envs/dev/`, `envs/prod/` appellent le même module | Corriger une fois, régler deux fois |
| State | backend `key` différent par env | Une erreur en dev ne touche pas la prod |
| Secrets | un jeu de secrets par env | La dev n'a pas les clés de la prod |

## Rituels

- `pwd` avant chaque `apply` : le dossier courant = l'environnement.
- `terraform state list` : vérifier que tu es sur le registre attendu.
- `sensitive = true` sur tout output qui expose un secret.