# Leçon 2 — Terraform : ton premier projet (provider, resource, variable, output, workflow)

> **Bloc 8 · Infrastructure as Code (IaC)** — Leçon 2 sur 9
> 🧭 **Pont depuis la Leçon 1** : tu sais **pourquoi** l'IaC existe (fini les installations à la main du Bloc 7) et tu as installé `terraform`. Mais à quoi ressemble un **projet Terraform complet** ? Cette leçon répond en 4 temps : le vocabulaire du langage **HCL** (provider, resource, variable, output), le **workflow** en 4 commandes (`init`, `plan`, `apply`, `destroy`), un premier aperçu du fichier **state** (approfondi en Leçon 3), et tout se pratique **en local, gratuitement** — tu vas créer de vrais fichiers sur ton disque avec le même code que pour le cloud.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Créer** l'arborescence d'un projet Terraform et **expliquer** le rôle de chaque fichier (`.tf`, `.terraform`, `terraform.tfstate`, `terraform.tfstate.backup`).
2. **Écrire** des blocs `terraform` (versions), `provider`, `resource`, `variable` et `output` en HCL.
3. **Exécuter** le workflow complet — `init`, `plan`, `apply`, `destroy` — et **expliquer** ce que fait chaque étape.
4. **Modifier** ton code et **re-appliquer** pour observer la mise à jour ciblée et l'**idempotence**.
5. **Lire** la sortie de `terraform plan` (« + create », « ~ update in-place », « - destroy »).

---

## 2. Explication simple

### 2.1 Le « quoi » : les 4 briques du vocabulaire Terraform

Un fichier Terraform est composé de **blocs** — des morceaux de texte entre accolades `{ }` — qui jouent chacun un rôle précis. Reprenons-les avec une analogie filée : **construire une maquette de ville**.

| Bloc | Ce qu'il dit | Analogie (maquette de ville) |
|------|--------------|------------------------------|
| `terraform` | Les réglages du projet : version minimale de Terraform et **liste des « plugins »** nécessaires | La **notice générale** de la maquette : quelles boîtes d'outils il faut acheter |
| `provider` | La **plateforme** où créer les ressources | La **marque** de la maquette : « je construis chez Legoville » (AWS) — une autre marque serait un autre provider (GCP) |
| `resource` | **Une ressource concrète** à créer, de tel type, avec telles propriétés | **Un bâtiment** : « une maison, 3 étages, toit rouge » |
| `variable` | Une **valeur réglable** de l'extérieur | Un **bouton de réglage** : « hauteur du bâtiment : 3 étages par défaut » |
| `output` | Une **information affichée** à la fin de l'exécution | L'**étiquette d'infos** collée sur la maquette : « adresse de la maison : … » |

> 💡 **Analogie du provider** : le provider est un **traducteur**. Terraform, lui, ne sait pas parler « AWS », ni « Google », ni « fichiers locaux ». Chaque provider est un **plugin** (une extension installée à part) qui traduit ton code dans les appels réels de la plateforme cible. Changer de provider = changer de traducteur, **sans changer ta façon d'écrire le code** : c'est ce qui rend Terraform multi-cloud.

Deux mots de vocabulaire au passage :

- **HCL** (*HashiCorp Configuration Language*) : le langage des fichiers Terraform. Lisible, pensé pour la configuration — tu ne « programmes » pas, tu **décris**.
- **Extension `.tf`** : tous les fichiers Terraform finissent par `.tf`. Terraform lit **tous** les `.tf` d'un dossier comme un seul ensemble — c'est pourquoi on les sépare par **rôle** (`versions.tf`, `main.tf`, `variables.tf`, `outputs.tf`) pour la lisibilité, pas pour la logique.

### 2.2 Le « comment » : le workflow en 4 commandes

Écrire le code ne crée encore **rien**. Terraform exécute les changements quand tu lui demandes, en 4 commandes qui s'enchaînent logiquement :

```
terraform init    →  prépare le projet : télécharge les providers déclarés
terraform plan    →  prévisualise : "voilà ce que je vais créer/modifier/supprimer"
terraform apply   →  exécute réellement les changements (après ta confirmation)
terraform destroy →  supprime proprement tout ce que ce projet a créé
```

| Commande | Ce qu'elle fait | Analogie |
|----------|-----------------|----------|
| `init` | Télécharge les **providers** (les plugins) et prépare le dossier | **Déballer la boîte d'outils** achetée selon la notice |
| `plan` | Compare code ↔ réalité et **affiche le programme de travaux**, sans rien toucher | La **maquette d'avant-travaux** : « je vais percer ici, peindre là » |
| `apply` | Exécute le programme, **après que tu as confirmé** | Le **chantier** lui-même |
| `destroy` | Supprime ce que ce projet a créé | Le **démontage** complet de la maquette |

**Pourquoi la séparation `plan` / `apply` ?** C'est le filet de sécurité de l'IaC : tu vois **avant** d'exécuter si Terraform va créer, modifier ou — attention — **supprimer** des ressources. Jamais de surprise. En production, on exige toujours la lecture du plan avant l'apply.

### 2.3 Le « quand » : le cycle déclaratif complet

À chaque `apply`, Terraform suit toujours le même cycle, hérité de la Leçon 1 (§ 2.3) :

```
1. Lire ton code          ("ce que je veux")
2. Lire le state          ("ce que j'ai déjà créé" — registre interne)
3. Interroger le réel     ("ce qui existe vraiment")
4. Calculer les écarts    ("+ créer, ~ modifier, - supprimer")
5. Exécuter les écarts    (et mettre à jour le state)
```

Le **state** — le fichier `terraform.tfstate` qui apparaîtra chez toi — est le **registre** de ce que Terraform a créé : le « cahier de suivi du chantier ». Pour l'instant, retiens juste son rôle ; il mérite une leçon entière (Leçon 3) parce qu'il est à la fois **indispensable** et **dangereux** s'il est maltraité.

### 2.4 Pourquoi commencer en local ?

Un provider peut cibler n'importe quoi : le cloud AWS, mais aussi **ta propre machine**. Deux providers officiels suffisent pour apprendre tout le vocabulaire, **gratuitement et sans compte** :

- **`local`** : crée et gère des **fichiers sur ton disque**. Tu verras de vrais fichiers apparaître — c'est de la vraie infrastructure, petite échelle.
- **`random`** : génère des **valeurs aléatoires** (chaînes, nombres) — utile pour simuler des identifiants ou des mots de passe.

Le code HCL sera **strictement identique** en Leçon 5 avec le provider AWS : seul le « traducteur » change. C'est exactement le principe du Bloc 6 : apprendre les concepts une fois, les rejouer ailleurs.

---

## 📖 Vocabulaire / Abréviations

| Terme | Définition (une ligne) | Où |
|-------|------------------------|-----|
| **HCL** | Le langage des fichiers Terraform, lisible, orienté description | § 2.1 |
| **Bloc** | Un morceau de code entre `{ }` avec un type et un nom | § 2.1 |
| **Provider** | Le plugin traducteur vers une plateforme (local, random, AWS…) | § 2.1 |
| **Resource** | Une ressource concrète à créer (fichier, VM, base…) | § 2.1 |
| **Variable** | Une valeur réglable de l'extérieur, avec défaut | § 2.1 |
| **Output** | Une information affichée à la fin de l'exécution | § 2.1 |
| **Interpolation `${...}`** | Insérer la valeur d'une variable/ressource dans un texte | § 3.3 |
| **Jeton (token)** | Une valeur aléatoire, comme un identifiant ou un mot de passe simulé | § 3.3 |
| **Workflow** | L'enchaînement `init` → `plan` → `apply` (→ `destroy`) | § 2.2 |
| **State** | Le registre (`terraform.tfstate`) de ce que Terraform a créé | § 2.3 |
| **`.terraform/`** | Le dossier caché où `init` range les providers téléchargés | § 3.4 |
| **`.tfstate.backup`** | La copie de secours de l'état précédent, faite à chaque apply | § 3.4 |
| **`~>`** | Contrainte de version : « cette version ou une mineure supérieure » | § 3.2 |

---

## 3. Exemples concrets

La théorie est posée ; on construit maintenant le projet pas à pas, chaque fichier jouant le rôle vu en § 2.1. Ouvre un terminal et suis les commandes — toutes commentées ligne par ligne.

### 3.1 Créer le projet

```bash
# Crée un dossier dédié (règle : 1 projet Terraform = 1 dossier).
mkdir ~/terraform-debut

# Entre dedans : Terraform agit TOUJOURS sur le dossier courant.
cd ~/terraform-debut
```

### 3.2 Déclarer les outils nécessaires — `versions.tf`

```bash
# Crée le fichier versions.tf avec nano (éditeur de texte du Bloc 2).
nano versions.tf
```

Colle ce contenu (commenté), puis sauvegarde avec `Ctrl+O`, `Entrée`, et quitte avec `Ctrl+X` :

```hcl
# Bloc "terraform" : les réglages du projet.
terraform {
  # Version minimale de Terraform exigée pour ce code.
  required_version = ">= 1.5.0"

  # La liste des providers (plugins) à télécharger.
  required_providers {
    # Clé locale "local" : ce provider, sa source officielle, sa version.
    local = {
      source  = "hashicorp/local"   # où le télécharger (registre officiel)
      version = "~> 2.0"            # ~> = "2.x, mais pas 3" (épinglé : reproductible)
    }
    # Provider "random" : génération de valeurs aléatoires.
    random = {
      source  = "hashicorp/random"
      version = "~> 3.0"
    }
  }
}
```

### 3.3 Décrire les ressources — `main.tf`

```bash
nano main.tf
```

```hcl
# Les variables d'abord (bloc "variable" = un bouton de réglage).
variable "nom_projet" {
  description = "Nom du projet, réutilisé dans les fichiers créés"
  type        = string
  default     = "atelier"          # valeur utilisée si on ne fournit rien
}

variable "longueur_token" {
  description = "Longueur du jeton aléatoire généré"
  type        = number
  default     = 12
}

# Le provider local : ici, pas de réglage particulier, on le déclare vide.
provider "local" {}

# Le provider random : idem.
provider "random" {}

# Ressource 1 : un jeton aléatoire (random_string = une chaîne de caractères).
# "jeton" (en anglais : token) = une valeur aléatoire, comme un identifiant ou un mot de passe.
resource "random_string" "token" {
  length  = var.longueur_token   # var.NOM = lire la variable déclarée ci-dessus
  special = false                # pas de caractères spéciaux (!, @, #…)
}

# Ressource 2 : un fichier réel sur ton disque.
resource "local_file" "carte_identite" {
  # content = le contenu voulu. ${...} = interpolation : "insère la valeur de…".
  content  = "Projet  : ${var.nom_projet}\nJeton   : ${random_string.token.result}\n"
  # filename = le nom du fichier à créer.
  filename = "carte-identite-${var.nom_projet}.txt"
}

# Bloc "output" : une information à afficher à la fin de l'apply.
output "jeton_genere" {
  description = "Le jeton généré (comme un mot de passe simulé)"
  value       = random_string.token.result
}

output "fichier_cree" {
  description = "Le chemin du fichier créé"
  value       = local_file.carte_identite.filename
}
```

> 💡 **Lecture de `content`** : le `\n` signifie « retour à la ligne » (comme la touche Entrée). L'interpolation `${random_string.token.result}` insère la **valeur produite par une autre ressource** — c'est comme ça que les briques se passent des informations entre elles.

### 3.4 Exécuter le workflow

**Étape 1 — `init` : préparer le projet**

```bash
# Télécharge les providers déclarés dans versions.tf et prépare le dossier.
terraform init
```

Sortie attendue (extrait) : `Initializing provider plugins...` puis `- Installing hashicorp/local v2.x.x ...` et à la fin `Terraform has been successfully initialized!`. Après l'opération, un dossier caché `.terraform/` est apparu : il contient les **plugins téléchargés** (ne le modifie jamais ; s'il est supprimé, relance simplement `init`).

**Étape 2 — `plan` : prévisualiser sans toucher**

```bash
# Affiche le programme de travaux : ce qui va être créé/modifié/supprimé.
terraform plan
```

Lis la fin de la sortie :

```
Plan: 2 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + fichier_cree = "carte-identite-atelier.txt"
  + jeton_genere = <aléatoire>
```

Lecture : **2 to add** (les 2 ressources vont être **créées**), 0 to change, 0 to destroy. Les symboles : `+` = création, `~` = modification « sur place » (*in-place*), `-` = suppression. **Rien n'a encore été créé** — le plan ne touche à rien.

**Étape 3 — `apply` : exécuter réellement**

```bash
# Exécute le plan après ta confirmation.
terraform apply
```

Terraform te redemande : `Do you want to perform these actions? ... Enter a value:` — tape **`yes`** puis Entrée. À la fin, les **outputs** s'affichent :

```
jeton_genere = "aX7bTq2mZp9w"
fichier_cree = "carte-identite-atelier.txt"
Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

Vérifie le résultat réel :

```bash
# Liste les fichiers : le fichier "carte-identite-atelier.txt" existe !
ls -l

# Affiche son contenu : projet et jeton, comme voulu.
cat carte-identite-atelier.txt
```

**Étape 4 — l'idempotence en action : re-appliquer sans rien changer**

```bash
# Re-applique SANS avoir modifié le code.
terraform apply
```

La sortie dit : `No changes. Your infrastructure matches the configuration.` — rien n'est refait. **C'est l'idempotence** de la Leçon 1, vérifiée par toi-même : relancer le code ne crée jamais de doublons.

**Étape 5 — modifier le code, observer la mise à jour ciblée**

Ouvre `main.tf` et change uniquement la valeur par défaut de la variable : `default = "atelier"` → `default = "labo"`. Puis :

```bash
# Le plan montre une mise à jour CIBLÉE, pas une recréation complète.
terraform plan
```

Sortie : `1 to add, 0 to change, 1 to destroy` — l'ancien fichier est **remplacé** (son nom change : Terraform détruit l'ancien, crée le nouveau). Le jeton, lui, **ne change pas** : la ressource `random_string` ne dépend pas de cette variable, donc rien ne la force à se recréer. C'est la **mise à jour ciblée** : Terraform ne touche que ce qui est concerné. Vérifie avec `apply` (tape `yes`), puis `ls` : nouveau fichier, même jeton dedans. **Le state a été mis à jour** en même temps.

**Étape 6 — `destroy` : tout démonter proprement**

```bash
# Supprime tout ce que CE projet a créé (tape "yes" pour confirmer).
terraform destroy

# Le fichier créé n'existe plus ; seul le code reste.
ls
```

> 🔑 **Le rituel à retenir** : `init` une fois (et après chaque ajout de provider), puis le trio **`plan` → lire → `apply`** à chaque changement, et `destroy` quand tu as fini de jouer. En Leçon 5, ce même rituel s'appliquera à du vrai cloud.

### 3.5 Les fichiers du projet après coup

```bash
# ls -a liste aussi les fichiers cachés (ceux qui commencent par un point).
ls -la
```

| Fichier/dossier | C'est quoi ? | Qui l'a créé ? |
|-----------------|--------------|----------------|
| `*.tf` | **Ton code** — versionne-le dans Git | Toi |
| `.terraform/` | Les providers téléchargés — à **ignorer dans Git** | `terraform init` |
| `terraform.tfstate` | Le **state** : le registre de ce que Terraform gère — sensible, à protéger (Leçon 3) | `terraform apply` |
| `terraform.tfstate.backup` | La copie de secours de l'état précédent | `terraform apply` |

---

## 4. Bonnes pratiques modernes (2025-2026)

- **Un fichier par rôle** : `versions.tf` (réglages), `main.tf` (ressources), `variables.tf` (variables), `outputs.tf` (sorties). Terraform fusionne tout de toute façon — c'est pour la **lisibilité de l'équipe**.
- **Toujours `plan` avant `apply`**, et **lire le plan** : c'est le réflexe professionnel n°1. Le `apply` direct sans lecture est l'équivalent d'un « oui » en catimini sans avoir lu ce qu'on confirme.
- **`-auto-approve` réservé aux pipelines** : l'option `-auto-approve` (sauter la confirmation) n'est acceptable que dans une chaîne d'automatisation (CI/CD, Bloc 11) déjà validée par un humain — jamais pour « aller plus vite » à la main.
- **Épingler les versions** des providers (`~> 2.0`) : le code reste reproductible même des années plus tard.
- **Décrire avec des `description`** dans les `variable` et `output` : ton futur toi (ou un collègue) comprendra sans deviner.
- **`.tfstate` hors de Git** : il contient potentiellement des secrets et il est volatile — on l'ignore via `.gitignore` et on le stockera à distance (Leçon 3).

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi c'est dangereux/inefficace | ✅ Version correcte |
|-----------------|--------------------------------------|---------------------|
| Modifier à la main le fichier créé par Terraform (ex. éditer `carte-identite-atelier.txt`) | Crée du **drift local** : le prochain `apply` écrasera ta modification « par surprise » | Tout changement passe par le **code**, puis `apply` |
| Supprimer ou éditer `terraform.tfstate` à la main | Terraform « oublie » ce qu'il gérait : il recréera des doublons ou croira que tout a disparu | Le laisser tranquille ; le sauvegarder et le déporter en backend (Leçon 3) |
| `terraform apply -auto-approve` par paresse | Tu contournes ton propre filet de sécurité : destructions non lues possibles | `plan` → **lecture** → `apply` (confirmation) |
| Un gros `main.tf` de 500 lignes qui mélange tout | Illisible, invivable en équipe | Séparation par rôle (§ 4) — et **modules** en Leçon 4 |
| Oublier `init` après avoir ajouté un provider | Erreur `Provider configuration not found` ou `Missing required provider` | Relancer `terraform init` — c'est prévu pour |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`**, la correction dans **`03-correction.md`**.

**Énoncé court** : reconstruis le projet de la leçon en changeant 3 éléments : une variable `environnement` (`dev`/`prod`), un deuxième fichier généré (un « journal »), et un output supplémentaire. Puis expérimente : changement de variable → lecture du plan → apply → re-apply (idempotence) → destroy. Enfin, réponds à 2 questions de lecture de plan (`+`, `~`, `-`).

---

## 7. Correction détaillée de l'exercice

> La correction complète est dans **`03-correction.md`** : code HCL final commenté, sortie attendue de chaque commande, et réponses aux questions de lecture de plan.

---

## 8. Checklist de validation

- [ ] Je crée un projet Terraform et j'explique le rôle de chaque bloc (`terraform`, `provider`, `resource`, `variable`, `output`).
- [ ] J'exécute le workflow complet `init` → `plan` → `apply` → `destroy` et j'explique chaque étape.
- [ ] Je lis une sortie de `terraform plan` et je connais les symboles `+`, `~`, `-`.
- [ ] J'ai **vérifié l'idempotence** : un re-apply sans changement affiche `No changes`.
- [ ] Je sais ce que sont `terraform.tfstate`, `.terraform/` et `terraform.tfstate.backup`.
- [ ] Je modifie une variable et le plan montre une mise à jour **ciblée**, pas un chaos.

---

🧭 **Pont vers la suite** — Ton projet fonctionne… mais regarde bien : dans le dossier, il y a un fichier que tu n'as jamais écrit, `terraform.tfstate`, que tu n'as **pas le droit de perdre**. Sans lui, Terraform « oublie » tout ce qu'il a créé : des doublons à chaque apply, des ressources fantômes, des secrets en clair. C'est LA notion la plus importante du bloc : la **Leçon 3 — le state et son backend**.

---

*Prochaine étape :* Leçon 3 — **Le state : le registre vital de Terraform** dans `03-Terraform-state-et-backend/`.