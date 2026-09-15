# Correction — Leçon 4 : modules et environnements

> **Bloc 8 · Leçon 4** — Correction pas à pas.

---

## Étape 1 — L'arborescence

```bash
# -p : crée le dossier et tous ses parents au besoin.
mkdir -p ~/organise/modules/journal ~/organise/envs/dev ~/organise/envs/prod

cd ~/organise/modules/journal
cp ~/atelier-securise/versions.tf .   # point de départ : les providers
```

Résultat visé :

```
~/organise/
├── modules/journal/     (le module : la recette)
└── envs/dev/  envs/prod/   (les projets racine : les versions)
```

## Étape 2 — Le module `journal`

**`modules/journal/variables.tf`** :

```hcl
variable "nom_env" {
  description = "Nom de l'environnement (dev, prod…)"
  type        = string
  # SANS default : l'appelant DOIT la fournir (erreur immédiate sinon).
}

variable "taille_secret" {
  description = "Longueur du secret de cet environnement"
  type        = number
  default     = 10
}
```

**Explication** : la variable **sans défaut** est le bon réflexe pour ce qui ne peut pas être deviné — le module refuse de s'appliquer tant que l'appelant n'a pas décidé. C'est la validation la plus tôt possible : à l'écriture, pas en production.

**`modules/journal/main.tf`** :

```hcl
resource "random_string" "secret" {
  length  = var.taille_secret   # paramétrable par l'appelant
  special = true
}

resource "local_file" "journal" {
  content  = <<-EOT
    Environnement : ${var.nom_env}
    Taille secret : ${var.taille_secret}
    Secret        : ${random_string.secret.result}
  EOT
  filename = "journal-${var.nom_env}.txt"   # un fichier PAR environnement
}
```

**Explication** : **aucun bloc `provider`** dans le module. Le module reste générique : c'est le projet racine qui déclare qu'on travaille chez `local`/`random`. Le module dit « quoi » ; le racine dit « où/avec quoi ».

**`modules/journal/outputs.tf`** :

```hcl
output "chemin" {
  description = "Chemin du fichier journal créé"
  value       = local_file.journal.filename
}

output "secret" {
  description = "Le secret généré pour cet environnement"
  value       = random_string.secret.result
  sensitive   = true   # masqué à l'affichage : pas de secret qui s'affiche en clair
}
```

**Explication** : `sensitive = true` masque la valeur à l'écran (elle s'affiche `<sensitive>`), tout en restant **en clair dans le state** (Leçon 3) — le masque protège les logs, pas le stockage.

---

## Étape 3 — Les appels depuis `dev` et `prod`

**`envs/dev/main.tf`** :

```hcl
module "journal" {
  source        = "../../modules/journal"
  nom_env       = "dev"
  taille_secret = 8
}
```

**`envs/prod/main.tf`** :

```hcl
module "journal" {
  source        = "../../modules/journal"
  nom_env       = "prod"
  taille_secret = 32
}
```

**Explications** :
- `../../modules/journal` : un **chemin relatif** — deux niveaux au-dessus du dossier de l'environnement (`envs/dev` → `envs` → `organise`), puis `modules/journal`. (Chemins relatifs, pas absolus : le code reste portable d'une machine à l'autre — rappel du Bloc 2.)
- Chaque environnement contient aussi le **`versions.tf`** copié de la Leçon 2 (les providers `local` + `random`). Pourquoi ici et pas dans le module ? Parce que le bloc `terraform { required_providers }` du racine est celui qui télécharge les plugins (via `init`) ; les enfants héritent de ce choix. (Tu peux y laisser aussi `required_version`.)
- **Adresse des ressources créées** : elles ne s'appellent plus `random_string.secret` mais `module.journal.random_string.secret` — tu le verras dans `terraform state list`. C'est logique : deux appels du même module doivent cohabiter, chacun dans « son tiroir ».

## Étape 4 — Exécuter, comparer, démanteler

```bash
cd ~/organise/envs/dev
terraform init                                   # prépare le projet (télécharge les providers)
terraform apply -var-file="dev.tfvars"           # tape yes
# → outputs : chemin = journal-dev.txt, secret = <sensitive>
```

**Explication** : `-var-file="dev.tfvars"` lit le fichier de valeurs ; l'output `secret` s'affiche **masqué** grâce à `sensitive = true`.

```bash
cd ../prod
terraform init
terraform apply                                  # tape yes → défauts du code (taille 32)

cat ../dev/journal-dev.txt    # Environnement: dev, Taille: 8
cat journal-prod.txt          # Environnement: prod, Taille: 32, secret plus long
```

**Vérification attendue** : deux fichiers, deux secrets de longueurs différentes — **le même module**, deux réglages. Zéro copier-coller de resources.

```bash
cd ../dev
terraform destroy -var-file="dev.tfvars"         # tape yes → "Resources: 2 destroyed."
```

**Explication** : `destroy` ne touche **que** le state du dossier courant (la dev). La prod, dans un autre dossier avec un autre registre, n'est pas au courant — c'est exactement l'isolement voulu.

## Étape 5 — Vérifier l'isolement

```bash
cd ~/organise/envs/dev && terraform state list
# (vide — la dev a été détruite)

cd ../prod && terraform state list
# module.journal.random_string.secret
# module.journal.local_file.journal
```

**Réponses aux questions** :

1. **Pourquoi un registre par environnement ?** Parce que le state EST la frontière de responsabilité de Terraform : avec des registres séparés, un apply raté en dev ne peut **physiquement pas** lire/écrire les ressources de prod. C'est le niveau 2 de la séparation (code / state / secrets) posée en § 2.3.
2. **Où placer le `.gitignore` ?** À la **racine du dépôt** (`~/organise/.gitignore`) : les règles s'appliquent à tous les sous-dossiers. Une seule ligne `*.tfstate` suffit — les motifs avec `*` sont relatifs à chaque dossier concerné.
3. **Où le secret apparaît-il quand même en clair ?** Dans le **state** (`terraform.tfstate` — Leçon 3) : `sensitive = true` masque l'affichage à l'écran et dans les logs, pas le stockage. D'où : state chiffré dans un backend, jamais dans Git.

---

## Checklist de validation (leçon 4)

- [ ] J'explique ce qu'est un module et pourquoi il remplace le copier-coller.
- [ ] Je crée un module (variables sans/avec défaut, resources, outputs `sensitive`) et je l'appelle avec `source`.
- [ ] Je distingue dev / staging / prod et je sais pourquoi on les sépare.
- [ ] J'applique le trio de séparation : dossiers de code + **states séparés** + secrets hors Git.
- [ ] J'utilise `.tfvars` et `-var-file` pour régler un environnement sans toucher au code.
- [ ] J'ai vérifié l'isolement : détruire la dev n'affecte pas la prod.

---

## 🧠 Conseils pour la suite

- **Reprends cette arborescence pour la suite du bloc** : en Leçon 5, le module `journal` deviendra (mentalement) un module « infrastructure AWS » ; garde le réflexe module + envs.
- **L'erreur d'environnement est l'erreur n°1 en IaC** : le réflexe `pwd` avant chaque `apply` n'est pas de la paranoïa, c'est la discipline qui évite « j'ai détruit la prod en croyant être en dev ».
- En Leçon 5, tu reprendras le **même rituel** (`init`, `plan`, lecture, `apply`) — mais chaque erreur coûtera des centimes et demandera un `destroy` : d'où l'importance de la lecture du plan, habituelle depuis la Leçon 2.