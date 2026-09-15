# Leçon 4 — Modules et environnements : organiser son code

> **Bloc 8 · Infrastructure as Code (IaC)** — Leçon 4 sur 9
> 🧭 **Pont depuis la Leçon 3** : ton state est protégé (backend préparé, `.gitignore` en place). Mais ton code a un défaut d'organisation : tout vit dans un seul niveau de dossiers. Si tu veux la même infrastructure en **dev** (petite, jetable) et en **prod** (grande, sérieuse), le réflexe du débutant est le **copier-coller** — et chaque correction devient double. Cette leçon transpose au langage Terraform ce que tu connais déjà en programmation : les **fonctions réutilisables** (modules) et la **séparation des environnements** (dev/staging/prod), avec la gestion des secrets qui va avec.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Expliquer** ce qu'est un **module** Terraform et pourquoi il évite le copier-coller.
2. **Créer** un module (dossier + variables + resources + outputs) et l'**appeler** depuis un projet racine avec `source`.
3. **Organiser** un dépôt en un module partagé + des dossiers d'environnements (`dev`, `prod`).
4. **Utiliser** des fichiers `.tfvars` pour régler chaque environnement sans toucher au code.
5. **Expliquer** pourquoi les secrets ne vont **jamais** dans le code ni dans Git, et citer des alternatives.

---

## 2. Explication simple

### 2.1 Le « pourquoi » : le copier-coller est le pire ami de l'IaC

Imagine la suite du parcours : en Leçon 5, ton code créera le réseau, la machine, la base — une centaine de lignes. Tu veux maintenant la même chose pour **tester** (dev) et pour **la vraie vie** (prod). Avec le copier-coller :

- tu corriges un bug **deux fois** (dev ET prod) ;
- les deux copies **divergent** silencieusement (celle de prod a un réglage que la dev n'a pas — le fameux « ça marche en dev mais pas en prod ») ;
- ton code devient illisible.

> 💡 **Analogie** : c'est la **recette photocopiée**. Tu tapes la recette du gâteau une fois, tu la mets dans ton livre de cuisine, et tu écris ensuite seulement « gâteau, mais avec 3 œufs » ou « gâteau, mais version famille ». Le module, c'est la recette du livre ; les environnements, ce sont les « versions » de la recette.

C'est exactement le principe de **fonction** que tu connais en programmation (Java, Python, Bash du Bloc 3) : on encapsule une logique, on la **paramètre**, et on l'appelle autant de fois que nécessaire. Terraform a le même outil : le **module**.

### 2.2 Le « comment » : un module = un dossier réutilisable

Un **module Terraform**, c'est simplement **un dossier** contenant des fichiers `.tf` (les mêmes que depuis la Leçon 2 : `variables.tf`, `main.tf`, `outputs.tf…`). Ce dossier devient réutilisable dès qu'un **autre code** l'appelle :

```hcl
module "nom_local" {
  source = "./modules/mon-module"   # où trouver le module (chemin relatif, ou dépôt Git)
  variable_1 = valeur               # les "arguments" de la fonction
}
```

Les conventions à retenir :

- Le module **déclare ses variables** (comme une fonction déclare ses paramètres) et **retourne ses outputs** (comme une fonction retourne une valeur).
- Le projet qui **appelle** le module est le **projet racine** (*root*). Il fournit les valeurs.
- Chaque appel crée ses propres ressources : deux appels du même module = **deux infrastructures parallèles**, avec des noms différents (`module.nom_local`).
- Un module doit faire **une chose cohérente** (« créer le journal »), pas tout faire.

### 2.3 Le « quand » : les environnements dev / staging / prod

Les **environnements** sont des copies de l'infrastructure, destinées à des usages différents :

| Environnement | Rôle | Analogie |
|---------------|------|----------|
| **dev** (développement) | La base d'essai : on y casse les choses, c'est fait pour | Le **chantier d'essai** |
| **staging** (pré-production) | Une copie proche de la prod, pour les derniers tests | La **répétition générale** |
| **prod** (production) | Le vrai service, avec les vrais utilisateurs | La **représentation devant public** |

Pourquoi séparer ? Parce qu'une erreur en dev **ne doit jamais pouvoir toucher la prod**. La séparation se fait à **trois niveaux**, et ils s'ajoutent :

1. **Du code** : des dossiers `envs/dev/` et `envs/prod/`, chacun appelant le même module avec des réglages différents.
2. **Du state** : un state (donc un chemin de backend) **par environnement** — la règle posée en Leçon 3. Une erreur de apply en dev ne peut jamais corrompre le registre de la prod.
3. **Des secrets** : la prod a ses mots de passe, la dev les siens — jamais mélangés (§ 3.4).

Dans ce bloc, on pratique **dev et prod** (staging suit exactement le même modèle : tu sauras l'ajouter toi-même).

### 2.4 Les fichiers `.tfvars` : les réglages par environnement

Un fichier **`.tfvars`** fournit des valeurs aux variables, **sans toucher au code** :

```hcl
# terraform.tfvars — lu automatiquement par terraform plan/apply
longueur_secret = 40
```

On utilise couramment des fichiers dédiés par environnement (`dev.tfvars`, `prod.tfvars`), passés avec l'option `-var-file` :

```bash
terraform apply -var-file="dev.tfvars"   # -var-file = "lis ces valeurs-ci"
```

Et la règle de sécurité qui découle de la Leçon 3 : **un `.tfvars` qui contient des secrets ne va jamais dans Git** (on l'ajoute au `.gitignore`). Le code (Git) décrit la forme ; les valeurs sensibles restent à part — on citera les alternatives professionnelles en § 3.4.

---

## 📖 Vocabulaire / Abréviations

| Terme | Définition (une ligne) | Où |
|-------|------------------------|-----|
| **Module** | Un dossier de code Terraform réutilisable, appelé depuis le projet racine | § 2.2 |
| **Projet racine (root)** | Le dossier qui appelle les modules et déclare les providers | § 2.2 |
| **`source`** | L'argument qui dit où trouver le module (chemin relatif, ou dépôt Git) | § 3.2 |
| **Environnement** | Une copie de l'infrastructure avec un usage dédié (dev/staging/prod) | § 2.3 |
| **Dev / Staging / Prod** | Développement / pré-production (répétition) / production (le vrai service) | § 2.3 |
| **`.tfvars`** | Fichier qui fournit des valeurs aux variables, lu par plan/apply | § 2.4 |
| **`-var-file`** | Option qui indique quel fichier de valeurs utiliser | § 2.4 |
| **`sensitive = true`** | Masque la valeur à l'affichage (mais pas dans le state — Leçon 3) | § 3.2 |
| **Heredoc `<<-EOT`** | Syntaxe pour un texte sur plusieurs lignes (vue en Leçon 2) | § 3.1 |
| **Registry (registre public)** | Le dépôt de modules partagés de HashiCorp — mention en § 4 | § 4 |

---

## 3. Exemples concrets

On reconstruit le projet de la Leçon 2 en version professionnelle : **un module** (la recette) + **deux environnements** (dev, prod). Toutes les commandes sont commentées ligne par ligne.

### 3.1 L'arborescence visée

```
organise/
├── modules/
│   └── journal/            ← LE module (la recette)
│       ├── variables.tf
│       ├── main.tf
│       └── outputs.tf
└── envs/
    ├── dev/                ← UN projet racine par environnement
    │   ├── versions.tf     (providers : déclarés ici, une fois)
    │   ├── main.tf         (appelle le module)
    │   └── dev.tfvars
    └── prod/
        ├── versions.tf
        └── main.tf
```

Pourquoi cette forme ? Chaque environnement a **son state** (Leçon 3 : un state par environnement) — et un state par dossier de projet. Le module, lui, est **partagé** : corriger la recette une fois profite aux deux environnements.

### 3.2 Le module — `modules/journal/`

```hcl
# variables.tf — les paramètres de la "fonction".
variable "nom_env" {
  description = "Nom de l'environnement (dev, prod…)"
  type        = string
  # PAS de "default" : l'appelant est OBLIGÉ de fournir la valeur.
}

variable "taille_secret" {
  description = "Longueur du secret de cet environnement"
  type        = number
  default     = 10      # valeur raisonnable si l'appelant ne précise rien
}
```

```hcl
# main.tf — les ressources du module. Aucun bloc "provider" ici :
# le projet racine les déclare (un module reste un composant générique).
resource "random_string" "secret" {
  length  = var.taille_secret
  special = true
}

resource "local_file" "journal" {
  # Heredoc vu en Leçon 2 : texte sur plusieurs lignes.
  content  = <<-EOT
    Environnement : ${var.nom_env}
    Taille secret : ${var.taille_secret}
    Secret        : ${random_string.secret.result}
  EOT
  filename = "journal-${var.nom_env}.txt"
}
```

```hcl
# outputs.tf — les valeurs "retournées" par la fonction.
output "chemin" {
  description = "Chemin du fichier journal créé"
  value       = local_file.journal.filename
}

output "secret" {
  description = "Le secret généré pour cet environnement"
  value       = random_string.secret.result
  sensitive   = true   # masqué à l'affichage (mais PAS dans le state — Leçon 3)
}
```

### 3.3 Les projets racine — `envs/dev/` puis `envs/prod/`

```hcl
# envs/dev/main.tf — le projet racine qui APPELLE le module.
module "journal" {
  source        = "../../modules/journal"  # chemin relatif vers le module
  nom_env       = "dev"                    # obligatoire (pas de défaut)
  taille_secret = 8                        # un secret court pour la dev
}
```

```hcl
# envs/prod/main.tf — même module, réglages sérieux.
module "journal" {
  source        = "../../modules/journal"
  nom_env       = "prod"
  taille_secret = 32
}
```

Chaque environnement contient aussi le `versions.tf` (copie du projet de la Leçon 2 : les providers `local` et `random` y sont déclarés — c'est le **projet racine** qui les déclare).

### 3.4 Exécuter, comparer, démanteler

```bash
# L'application de la dev (depuis son dossier).
cd ~/organise/envs/dev
terraform init
terraform apply

# L'application de la prod (depuis son dossier).
cd ../prod
terraform init
terraform apply

# Vérification : deux fichiers, deux secrets de longueurs différentes.
cat ~/organise/envs/dev/journal-dev.txt
cat ~/organise/envs/prod/journal-prod.txt
```

**Et les secrets ?** On ne les met jamais dans le code. Les options professionnelles, par ordre de simplicité :

1. **Variables d'environnement** du shell (`TF_VAR_nom`) — non vues par Git ;
2. **Fichiers `.tfvars` hors dépôt** (dans `.gitignore`, gardés localement ou dans un coffre) ;
3. **Gestionnaires de secrets** (AWS Secrets Manager, Vault) — la voie professionnelle, dont tu as aperçu le concept au Bloc 5 (secrets) et au Bloc 6 (IAM) ; on l'utilisera en Leçon 5 avec l'IAM AWS.

> 🔑 **À retenir** : le code décrit la **forme** (un secret de N caractères) ; la **valeur** reste hors de Git.

> 🟡 **À mentionner, définir, ne pas creuser** — le **Registry** de HashiCorp : un catalogue public de modules partagés (réseau AWS, base de données…) déjà écrits et maintenus par la communauté. On les utilise avec `source = "terraform-aws-modules/vpc/aws"` + `version`. En production, on réutilise plutôt ces modules éprouvés que de tout écrire soi-même — mais pour apprendre, on écrit les nôtres.

---

## 4. Bonnes pratiques modernes (2025-2026)

- **Un module = une responsabilité** : « créer le journal », pas « créer le journal + le réseau + la base ». Les modules se composent ensuite comme des briques.
- **Variables sans défaut pour l'essentiel** : si une valeur doit forcément être choisie (nom d'env), ne mets pas de `default` — l'erreur se voit à l'écriture, pas en production.
- **Un projet racine par environnement** : `envs/dev`, `envs/prod`… avec chacun **son state** (backend `key` différent) — la règle de la Leçon 3.
- **Toujours `sensitive = true`** sur les outputs qui exposent un secret.
- **Descriptions partout** : `description` sur chaque variable, output et module — c'est la documentation vivante.
- **Versionner les modules partagés** : quand un module vit dans son propre dépôt Git, on appelle une **référence précise** (`?ref=v1.2.0`) pour que les environnements ne changent pas d'un seul coup.
- **Staging entre dev et prod** : le même modèle avec un troisième dossier — copie de la prod pour la répétition générale des déploiements.

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi c'est dangereux/inefficace | ✅ Version correcte |
|-----------------|--------------------------------------|---------------------|
| Copier-coller les resources entre dev et prod | Corrections en double, copies qui divergent, « ça marche en dev mais pas en prod » | **Un module** appelé deux fois avec des variables |
| Un seul state pour dev et prod | Un apply raté en dev touche la prod (même registre !) | Un state (backend `key`) **par environnement** |
| Secrets en dur dans le code (`mot_de_passe = "azerty"`) | Versionné dans Git = publié pour toujours (Bloc 5 — secrets) | `sensitive` + variables d'env / `.tfvars` hors Git / gestionnaire de secrets |
| Un module « fourre-tout » de 300 lignes | Impossible de réutiliser une seule partie | Modules petits et composables |
| Sortir de `envs/dev` et appliquer en pensant être ailleurs | Mauvais environnement = mauvais state = catastrophe possible | Toujours vérifier le dossier courant (`pwd`) avant `apply` |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`**, la correction dans **`03-correction.md`**.

**Énoncé court** : crée l'arborescence `modules/journal` + `envs/dev` + `envs/prod`, transforme le code de la Leçon 2 en module (`nom_env` sans défaut, `taille_secret` avec défaut, output `sensitive`), appelle le module depuis chaque environnement, applique les deux, détruis la dev sans toucher la prod, et vérifie que les registres sont bien isolés.

---

## 7. Correction détaillée de l'exercice

> La correction complète est dans **`03-correction.md`** : code HCL final, commandes commentées, et réponses aux 3 questions (isolement des states, `.gitignore` au bon niveau, où le secret reste en clair).

---

## 8. Checklist de validation

- [ ] J'explique ce qu'est un module et pourquoi il remplace le copier-coller.
- [ ] Je crée un module (variables sans/avec défaut, resources, outputs `sensitive`) et je l'appelle avec `source`.
- [ ] Je distingue dev / staging / prod et je sais pourquoi on les sépare.
- [ ] J'applique le trio de séparation : dossiers de code + **states séparés** + secrets hors Git.
- [ ] J'utilise `.tfvars` et `-var-file` pour régler un environnement sans toucher au code.
- [ ] J'ai vérifié la dév/déprod : détruire la dev n'affecte pas la prod.

---

🧭 **Pont vers la suite** — Tu maîtrises maintenant **le langage** (Leçon 2), **la mémoire** (Leçon 3) et **l'organisation** (Leçon 4). Le prochain pas est le grand saut : remplacer les providers `local`/`random` par le provider **AWS** et reconstruire en code l'architecture que tu as apprise au Bloc 6 (VPC, machine, stockage, base) — d'abord en **prévisualisant** (`plan`) sans rien créer, puis en créant pour de vrai dans le cadre du free tier. C'est la **Leçon 5**.

---

*Prochaine étape :* Leçon 5 — **Terraform chez AWS : reconstruire l'architecture du Bloc 6** dans `05-Terraform-AWS-architecture-reelle/`.