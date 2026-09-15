# Leçon 5 — Terraform chez AWS : reconstruire l'architecture du Bloc 6

> **Bloc 8 · Infrastructure as Code (IaC)** — Leçon 5 sur 9
> 🧭 **Pont depuis la Leçon 4** : tu maîtrises le langage (Leçon 2), la mémoire (Leçon 3) et l'organisation (Leçon 4) — le tout en local avec les providers `local`/`random`. Le grand pas est là : remplacer le « traducteur » par le **provider AWS** et décrire en code **l'architecture exacte que tu as apprise au Bloc 6** (VPC, sous-réseaux public/privé, machine, stockage, base managée). Tu retrouveras tous les concepts du Bloc 6 — cette fois, ils deviennent des **blocs HCL**. Et tu activeras enfin le **backend S3** préparé en Leçon 3. ⚠️ **Cette leçon se pratique sur un compte AWS réel (free tier) avec des garde-fous stricts** : tout est annoncé avant d'être créé, et tout est détruit à la fin.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Configurer** les accès AWS pour Terraform (via `aws configure`, créé au Bloc 6 — Leçon 6) et expliquer comment Terraform trouve les identifiants.
2. **Traduire** chaque concept du Bloc 6 (VPC, subnet, IGW, security group, EC2, S3, RDS) en **resource Terraform**.
3. **Utiliser** une **source de données** (`data "aws_ami"`) pour trouver l'image d'un système sans coder d'ID en dur.
4. **Préparer** le backend distant : créer le compartiment S3 et la table de verrou, activer `backend.tf`.
5. **Appliquer** l'architecture réelle dans le cadre du free tier, **vérifier**, puis **détruire** proprement.

---

## 2. Explication simple

### 2.1 Le « pourquoi » : rendre le Bloc 6 reproductible

Au Bloc 6, tu as (réellement ou en simulation) construit l'architecture pas à pas : le réseau privé, la machine publique, la base privée, le stockage. Mais ce sont des **actions**, pas du **code** : refaire la même chose = refaire les mêmes clics ou les mêmes commandes. Aujourd'hui, on **traduit cette architecture en fichiers HCL** — et le bénéfice immédiat est celui de la Leçon 1 : recréable à volonté, révisable dans Git, automatisable.

La correspondance concept ↔ code est directe — c'est le cœur de cette leçon :

| Concept du Bloc 6 | Resource Terraform | Analogie filée |
|---------------------|--------------------|----------------|
| VPC (réseau privé) | `aws_vpc` | Le **terrain clôturé** |
| Subnet public / privé | `aws_subnet` | Les **quartiers** ouvert / fermé sur l'extérieur |
| Internet Gateway | `aws_internet_gateway` | La **porte d'entrée** du terrain |
| Route table | `aws_route_table` (+ `association`) | Le **plan de circulation** |
| Security group | `aws_security_group` | Le **portier** de chaque quartier |
| Machine EC2 | `aws_instance` | La **maison** |
| Stockage S3 | `aws_s3_bucket` | Le **garde-meubles** |
| Base RDS | `aws_db_instance` (+ `aws_db_subnet_group`) | La **bibliothèque gérée** |
| AMI (image système) | `data "aws_ami"` (voir § 2.3) | Le **catalogue de maisons prêtes** |

### 2.2 Le « comment » : les identifiants et le provider

Terraform ne « sait » pas qui tu es : il utilise **les mêmes identifiants que l'AWS CLI** (Bloc 6, Leçon 6) — les clés stockées par `aws configure` dans `~/.aws/credentials`. Rien de nouveau à créer si tu as déjà configuré l'AWS CLI : c'est le même utilisateur **IAM** (rappel : *Identity and Access Management*, le service AWS qui gère qui peut faire quoi) qui parle à la fois à `aws` et à Terraform.

```hcl
# Le provider AWS : le "traducteur" vers les API d'AWS.
provider "aws" {
  region = var.region   # la région où tout sera créé (ex. eu-west-3)
}
```

> 🔑 **Règle de sécurité** (rappel du Bloc 5 — secrets) : **jamais de clés dans le code**. Le code dit seulement `region = var.region` ; les clés viennent de `~/.aws/credentials`, jamais d'un fichier `.tf` versionné dans Git.

### 2.3 Le « comment » (suite) : la source de données `data`

Un bloc **`resource`** dit « **crée-moi** cette chose ». Un bloc **`data`** dit « **va chercher** une chose qui existe déjà ». Cas d'usage n°1 : l'**AMI** (rappel Bloc 6 : l'AMI est l'image système préinstallée pour créer une EC2). Il existe des centaines d'AMI, chacune avec un identifiant propre à chaque région (`ami-0abc123…`). **Coder cet ID en dur est fragile** : il change selon la région et finit par disparaître. La bonne pratique est de **chercher l'AMI à la volée** :

```hcl
# data = "va chercher" (lecture seule), ne crée rien.
data "aws_ami" "ubuntu" {
  most_recent = true            # la plus récente qui correspond aux filtres
  owners      = ["099720109477"]  # l'ID du compte officiel Canonical (éditeur d'Ubuntu)

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}
```

> 💡 **Analogie** : plutôt que d'écrire dans ton plan « utilise le permis n° 42B » (qui peut être révoqué), tu écris « prends la dernière maison **du catalogue officiel Canonical** qui corresponde au modèle 22.04 ». Le plan reste valable des années.

### 2.4 Le « quand » : le backend S3 activé, et les garde-fous du free tier

Deux sujets se rejoignent ici, vus aux Leçons 3 et au Bloc 6 :

**Le backend S3** : la configuration écrite en Leçon 3 s'active aujourd'hui. Il y a une subtilité logique à connaître (le « problème de l'œuf et de la poule ») : le state doit vivre dans S3, mais le compartiment S3 doit **exister avant** que Terraform ne s'en serve. Solution du marché : on crée le compartiment et la table de verrou **une fois à la main** (avec l'AWS CLI du Bloc 6), et ensuite Terraform y range le state à chaque projet. C'est le seul objet de tout le bloc que tu créeras à la main — et il servira pour les leçons suivantes.

**Les garde-fous du free tier** (rappel Bloc 6, Leçon 8 — FinOps) : le free tier couvre une machine `t3.micro` ~750 h/mois et une base `db.t3.micro` ~750 h/mois — largement le temps d'un exercice. Les règles sont simples :
1. **Budget + alerte à 1 $** créé avant de commencer ;
2. **`plan` lu attentivement** avant chaque apply (c'est le réflexe de la Leçon 2, maintenant vital) ;
3. **`destroy` à la fin** de chaque session ;
4. jamais d'option `-auto-approve` à la main.

---

## 📖 Vocabulaire / Abréviations

| Terme | Définition (une ligne) | Où |
|-------|------------------------|-----|
| **Provider AWS** | Le plugin qui traduit le HCL en appels aux API d'AWS | § 2.2 |
| **IAM** | Le service AWS « qui peut faire quoi » (Bloc 6, Leçon 6) | § 2.2 |
| **`data`** | Bloc de **lecture** d'une ressource existante (à l'inverse de `resource` qui crée) | § 2.3 |
| **AMI** | L'image système préinstallée d'une EC2 (Bloc 6, Leçon 3) | § 2.3 |
| **Canonical** | L'éditeur d'Ubuntu (compte officiel propriétaire des AMI Ubuntu) | § 2.3 |
| **CIDR** | La notation d'un bloc d'adresses IP (`10.0.0.0/16`) — Bloc 5, Leçon 2 | § 3.3 |
| **IGW** | *Internet Gateway* : la porte d'entrée Internet d'un VPC | § 3.3 |
| **AZ (Availability Zone)** | Une zone de disponibilité : l'un des data centers isolés d'une région (ex. eu-west-3a) | § 3.3 |
| **Route table** | Le plan de circulation des paquets d'un subnet | § 3.3 |
| **`skip_final_snapshot`** | Option RDS : « détruis sans garder une dernière sauvegarde » | § 3.3 |
| **Subnet group (RDS)** | La liste des subnets privés où la base peut s'installer (2 zones minimum) | § 3.3 |
| **Free tier** | Le palier gratuit d'AWS : quotas mensuels offerts | § 2.4 |
| **`-out=tfplan`** | Option de `plan` qui sauvegarde le plan pour l'appliquer tel quel | § 3.5 |

*(Rappels cités : VPC/subnet/SG = Bloc 6 Leçon 2 · EC2 = Bloc 6 Leçon 3 · S3 = Bloc 6 Leçon 4 · RDS = Bloc 6 Leçon 5 · IAM = Bloc 6 Leçon 6 · FinOps = Bloc 6 Leçon 8.)*

---

## 3. Exemples concrets

On écrit maintenant le code complet, dans l'arborescence module + environnement de la Leçon 4. **Toutes les commandes sont commentées ligne par ligne** ; chaque bloc HCL renvoie au concept du Bloc 6.

### 3.1 L'arborescence du projet AWS

```
atelier-aws/
├── modules/
│   └── (les modules arriveront avec l'expérience — ici, un seul environnement)
└── envs/
    └── dev/
        ├── versions.tf     (provider AWS)
        ├── backend.tf      (le state dans S3 — Leçon 3)
        ├── variables.tf    (région, nom de projet, mot de passe)
        ├── main.tf         (l'architecture du Bloc 6)
        ├── outputs.tf      (IP publique, endpoint…)
        └── .gitignore      (les 3 lignes de la Leçon 3)
```

### 3.2 Les fichiers de configuration

**`versions.tf`** — un seul nouveau provider :

```hcl
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"   # le "traducteur" vers les API d'AWS
      version = "~> 5.0"          # 5.x : la génération actuelle
    }
    random = {
      source  = "hashicorp/random"  # pour générer le mot de passe de la base (Leçon 2)
      version = "~> 3.0"
    }
  }
}
```

**`backend.tf`** — celui de la Leçon 3, complété avec TON compartiment :

```hcl
terraform {
  backend "s3" {
    bucket         = "mon-nom-unique-terraform-states"  # créé à la main (exercice, étape 1)
    key            = "atelier-aws/terraform.tfstate"    # un chemin par projet
    region         = "eu-west-3"
    dynamodb_table = "terraform-locks"                  # le verrou (Leçon 3)
    encrypt        = true                               # le state contient des secrets
  }
}
```

**`variables.tf`** :

```hcl
variable "region" {
  description = "Région AWS où tout sera créé"
  type        = string
  default     = "eu-west-3"    # Paris — cohérent avec aws configure
}

variable "nom_projet" {
  description = "Préfixe de nommage de toutes les ressources"
  type        = string
  default     = "atelier"
}

variable "mot_de_passe_bdd" {
  description = "Mot de passe du compte admin de la base RDS"
  type        = string
  # PAS de défaut : fourni par variable d'environnement TF_VAR_… (Leçon 4)
  sensitive   = true           # masqué dans les sorties Terraform
}
```

**`outputs.tf`** :

```hcl
# L'adresse IP publique de la machine — utile en Leçon 6 pour Ansible.
output "ip_publique" {
  description = "IP publique de l'instance EC2"
  value       = aws_instance.app.public_ip
}

# Le nom du compartiment créé.
output "nom_bucket" {
  description = "Nom du compartiment S3 de documents"
  value       = aws_s3_bucket.documents.id
}

# L'endpoint de la base — l'adresse DNS stable (Bloc 6, Leçon 5).
output "endpoint_bdd" {
  description = "Endpoint de la base RDS"
  value       = aws_db_instance.bibliotheque.endpoint
  sensitive   = false   # l'endpoint n'est pas un secret (le mot de passe, lui, est à part)
}
```

### 3.3 `main.tf` — première moitié : le réseau (le VPC du Bloc 6 en code)

```hcl
provider "aws" {
  region = var.region   # où tout sera créé — vient de variables.tf
}

# ============ 1. LE VPC (Bloc 6, Leçon 2 : le terrain clôturé) ============
resource "aws_vpc" "principal" {
  cidr_block           = "10.0.0.0/16"   # le bloc d'adresses du réseau (Bloc 5, Leçon 2)
  enable_dns_support   = true            # la résolution de noms interne (DNS)
  enable_dns_hostnames = true            # les noms d'hôtes pour les ressources

  tags = { Name = "${var.nom_projet}-vpc" }   # nommage : indispensable pour se repérer
}

# ============ 2. LES SUBNETS (les quartiers) ============
# Le subnet PUBLIC : les machines y reçoivent une IP publique (map_public_ip…).
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.principal.id       # rattache le subnet au VPC
  cidr_block              = "10.0.1.0/24"              # un quartier : 256 adresses
  availability_zone       = "${var.region}a"           # une zone de disponibilité (AZ = Availability Zone :
                                                       # l'un des data centers isolés d'une région, ex. eu-west-3a)
  map_public_ip_on_launch = true                       # IP publique automatique ici
  tags = { Name = "${var.nom_projet}-public" }
}

# Deux subnets PRIVÉS : la base y vivra. Pourquoi DEUX ? RDS exige d'avoir
# au moins deux zones de disponibilité (repli en cas de panne d'une zone —
# le Multi-AZ aperçu au Bloc 6, Leçon 5). Une infra privée sans Internet.
resource "aws_subnet" "prive_a" {
  vpc_id            = aws_vpc.principal.id
  cidr_block        = "10.0.2.0/24"
  availability_zone = "${var.region}a"
  tags = { Name = "${var.nom_projet}-prive-a" }
}

resource "aws_subnet" "prive_b" {
  vpc_id            = aws_vpc.principal.id
  cidr_block        = "10.0.3.0/24"
  availability_zone = "${var.region}b"
  tags = { Name = "${var.nom_projet}-prive-b" }
}

# ============ 3. LA PORTE D'ENTRÉE (Internet Gateway) ============
resource "aws_internet_gateway" "principal" {
  vpc_id = aws_vpc.principal.id
}

# ============ 4. LE PLAN DE CIRCULATION (route table) ============
# Le subnet public a le droit de sortir vers Internet (via l'IGW)…
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.principal.id

  route {
    cidr_block = "0.0.0.0/0"              # "tout le reste du monde"
    gateway_id = aws_internet_gateway.principal.id
  }
  tags = { Name = "${var.nom_projet}-public" }
}

# …et on branche le subnet public sur cette route table.
resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}

# Les subnets privés n'ont AUCUNE route vers Internet : c'est le but (le portier
# du Bloc 5 appliqué au cloud). La base n'est joignable que depuis le VPC.

# ============ 5. LE PORTIER (security group de la machine web) ============
# Règle par défaut : tout est interdit, on n'OUVRE que le nécessaire
# (défaut-deny — le réflexe du Bloc 5, Leçon 3).
resource "aws_security_group" "web" {
  vpc_id     = aws_vpc.principal.id
  name       = "${var.nom_projet}-sg-web"
  description = "Autorise SSH depuis ta connexion et HTTP pour le web"

  # Entrée : HTTP (port 80) — depuis n'importe qui (c'est le but d'un site web).
  ingress {
    description = "HTTP depuis Internet"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"                   # TCP : le protocole fiable (Bloc 5, Leçon 1)
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Entrée : SSH (port 22) — depuis TON IP uniquement (remplace par ton IP publique :
  # cherche "mon ip" ; garde le /32 = une seule adresse, jamais 0.0.0.0/0 !).
  ingress {
    description = "SSH depuis ton poste"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["TON.IP.PUBLIQUE/32"]
  }

  # Sortie : tout autorisé (la machine doit télécharger ses paquets).
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"                    # -1 = tous les protocoles
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = { Name = "${var.nom_projet}-sg-web" }
}

# ============ 6. LE CATALOGUE (data AMI — § 2.3) ============
data "aws_ami" "ubuntu" {
  most_recent = true              # la plus récente correspondante
  owners      = ["099720109477"]  # le compte officiel Canonical (Ubuntu)

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}

# ============ 7. LA MAISON (instance EC2 — Bloc 6, Leçon 3) ============
resource "aws_instance" "app" {
  ami                    = data.aws_ami.ubuntu.id   # l'image trouvée à la volée
  instance_type          = "t3.micro"               # LE type du free tier
  subnet_id              = aws_subnet.public.id     # dans le quartier public
  vpc_security_group_ids = [aws_security_group.web.id]  # le portier de § 5

  tags = { Name = "${var.nom_projet}-app" }
}

# ============ 8. LE GARDE-MEUBLES (stockage S3 — Bloc 6, Leçon 4) ============
# Le nom du bucket doit être UNIQUE AU MONDE : garde un préfixe perso.
resource "aws_s3_bucket" "documents" {
  bucket = "${var.nom_projet}-documents-${ton-prefixe-unique}"
  tags   = { Name = "${var.nom_projet}-documents" }
}

# ============ 9. LA BIBLIOTHÈQUE GÉRÉE (RDS — Bloc 6, Leçon 5) ============
# D'abord le "subnet group" : la liste des quartiers privés où la base peut vivre.
resource "aws_db_subnet_group" "prive" {
  name       = "${var.nom_projet}-db-subnets"
  subnet_ids = [aws_subnet.prive_a.id, aws_subnet.prive_b.id]   # 2 zones = exigence RDS
}

# Le mot de passe de la base : généré aléatoirement (provider random — Leçon 2).
resource "random_password" "mdp_bdd" {
  length  = 24
  special = true
}

# Puis la base elle-même.
resource "aws_db_instance" "bibliotheque" {
  identifier        = "${var.nom_projet}-bibliotheque"
  engine            = "postgres"       # le moteur vu au Bloc 7
  engine_version    = "16"
  instance_class    = "db.t3.micro"    # LE type du free tier
  allocated_storage = 20               # 20 Go : le minimum RDS (couvert par le free tier)
  db_name           = "bibliotheque"   # la base du fil rouge du Bloc 7 !

  username = "admin_biblio"
  password = random_password.mdp_bdd.result   # le mot de passe généré ci-dessus

  db_subnet_group_name = aws_db_subnet_group.prive.name
  # PAS de security group dédié ici : sans lui, la base reste inatteignable
  # depuis Internet — exactement le comportement privé voulu. (En production,
  # on ajouterait un SG autorisant seulement le subnet public sur le port 5432.)

  skip_final_snapshot = true   # pour l'exercice : détruis SANS garder une dernière sauvegarde
  # (en production : false, avec un identifiant de snapshot — Bloc 7, Leçon 4)

  tags = { Name = "${var.nom_projet}-bibliotheque" }
}
```

> 🔑 **Le sens de la dépendance** : regarde les références (`aws_subnet.public.id`, `random_password.mdp_bdd.result`) : Terraform calcule tout seul l'**ordre de création** (le VPC avant les subnets, les subnets avant la base…) grâce au graphe de dépendances qu'il déduit des références. C'est un autre superpouvoir du déclaratif : tu ne gères pas l'ordre, le code le décrit.

### 3.4 Le rituel, version cloud

```bash
# Déclare le mot de passe de la base dans ton shell (option 1 des secrets, Leçon 4).
# TF_VAR_ + nom de la variable = la valeur est lue automatiquement par Terraform.
export TF_VAR_mot_de_passe_bdd="UnMotDePasseSolide123!"

# Prépare le projet : télécharge les providers AWS + random, ET propose
# la migration du state vers le backend S3 (celui de la Leçon 3 !).
terraform init
# → "Do you wish to proceed?" → tape yes : le state part vivre dans S3.

# Prévisualise et SAUVEGARDE le plan (-out = écrit le plan dans un fichier).
terraform plan -out=tfplan

# Lis le plan. Vérifie (exercice, étape 4) : région, types free tier, ~13 ressources.

# Applique le plan SAUVEGARDÉ : rien d'autre ne sera fait que ce que tu as lu.
terraform apply tfplan
```

À la fin de l'apply : les **outputs** s'affichent (`ip_publique`, `nom_bucket`, `endpoint_bdd`). La création prend plusieurs minutes — surtout la base RDS (compte tes blessures d'impatience : c'est normal, RDS est lent).

```bash
# La liste des ressources gérées — note le registre vit DANS S3 désormais.
terraform state list

# Les outputs à la demande (utile pour les leçons suivantes).
terraform output ip_publique
terraform output endpoint_bdd
```

**Vérification dans la console (observation seulement)** : VPC, EC2, S3, RDS — tout correspond au code. C'est le moment « waouh » : **l'architecture du Bloc 6 créée en ~13 blocs HCL**, refaisable à volonté.

### 3.5 Détruire proprement

```bash
# Détruit TOUT ce que ce projet a créé (tape yes).
terraform destroy

# Vérifie dans la console : plus rien (sinon, facture !).
# Le compartiment S3 du backend et la table DynamoDB restent — quasi gratuits,
# ils serviront aux leçons suivantes (Ansible, projet final).
```

> 💡 **Pourquoi le `destroy` est ici sans danger ?** Parce que les ressources sont de test. En production, le même `destroy` serait une catastrophe — c'est pour ça qu'on lit TOUJOURS son plan, même en destruction.

---

## 4. Bonnes pratiques modernes (2025-2026)

- **Un utilisateur IAM dédié à Terraform** (Bloc 6, Leçon 6) : au moindre privilège (ici : VPC, EC2, S3, RDS — pas tout le compte), ses clés dans `~/.aws/credentials` ; jamais les clés d'un admin généraliste.
- **Nommer via `tags` systématiquement** : `Name` + `projet` + `environnement` — c'est ainsi qu'on retrouve ses ressources et qu'on suit les coûts (Bloc 6, Leçon 8).
- **`plan -out=tfplan` puis `apply tfplan`** : le plan appliqué est **exactement** celui lu — pas de glissement entre les deux (par exemple si un collègue a modifié le code entre-temps).
- **L'AMI par `data`, jamais en dur** : le code reste valable quand les anciennes AMI disparaissent.
- **Variables sensibles (`sensitive = true`) + secrets hors Git** : le mot de passe de la base ne se retrouve ni dans le code, ni dans les logs.
- **Le backend S3 + verrou actif dès le premier apply** : c'est fait aujourd'hui, plus jamais de state local pour ce projet.

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi c'est dangereux/inefficace | ✅ Version correcte |
|-----------------|--------------------------------------|---------------------|
| Clés AWS en dur dans le code | Git les publierait pour toujours (Bloc 5 — secrets) | `aws configure` + `~/.aws/credentials`, clés jamais dans les `.tf` |
| `apply` sans lire le plan (surtout en cloud) | Créations/suppressions imprévues = facture ou outage | `plan -out=tfplan` → **lecture** → `apply tfplan` |
| Oublier le `destroy` en fin de session | Ressources payées pour rien (le piège n°1 du débutant cloud) | Budget + alerte, `destroy` en fin d'exercice, vérification console |
| Security group SSH ouvert à `0.0.0.0/0` | Des robots scannent Internet : essais de connexion permanents | SSH limité à `TON.IP/32` (et clé SSH, pas mot de passe — Bloc 2, Leçon 5) |
| AMI codée en dur (`ami-0abc123`) | ID propre à une région, qui disparaît avec le temps | `data "aws_ami"` avec filtres |
| Région incohérente (code ≠ `aws configure`) | Ressources créées ailleurs que prévu, mystère total en console | `aws configure get region` + `var.region` alignés |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`**, la correction dans **`03-correction.md`**.

**Énoncé court** : garde-fous (budget 1 $, région), création du compartiment S3 du backend + table de verrou à la main, écriture du projet complet (versions, backend, variables, main, outputs), secret par variable d'environnement, puis le rituel `init` (migration du state vers S3) → `plan -out` → lecture → `apply` → vérification console (observation) → **`destroy`**.

---

## 7. Correction détaillée de l'exercice

> La correction complète est dans **`03-correction.md`** : sorties attendues de chaque étape, les 4 points de contrôle du plan, et le déroulé de la migration du state vers le backend.

---

## 8. Checklist de validation

- [ ] J'explique comment Terraform trouve les clés AWS (`~/.aws/credentials`) et pourquoi elles ne vont jamais dans le code.
- [ ] Je traduis VPC/subnet/IGW/SG/EC2/S3/RDS en resources Terraform, et je sais pourquoi RDS exige un subnet group de 2 zones.
- [ ] Je connais la différence `resource` (créer) vs `data` (chercher), et pourquoi l'AMI se cherche à la volée.
- [ ] J'ai activé le backend S3 et je comprends le « problème de l'œuf et de la poule ».
- [ ] J'ai appliqué mes garde-fous (budget, lecture du plan, destroy) et tout est bien détruit.
- [ ] Je peux détruire ET reconstruire l'architecture du Bloc 6 depuis le code — le critère du bloc est en marche.

---

🧭 **Pont vers la suite** — Regarde ce que Terraform vient de créer : une **machine Ubuntu neuve, vide**. Mais une machine vide ne sert à rien : il faut installer Nginx, créer l'utilisateur `deploy`, déposer l'application — exactement ce que tu as fait **à la main** aux Blocs 2 et 7. C'est le rôle d'**Ansible**, la deuxième moitié du duo IaC : il parle aux **machines** (par SSH, rappel du Bloc 2, Leçon 5) pour les configurer. Première étape : les bases — inventaire, commandes ad-hoc et ton premier **playbook** idempotent. C'est la **Leçon 6**.

---

*Prochaine étape :* Leçon 6 — **Ansible : bases, inventaire et premier playbook** dans `06-Ansible-bases-inventaire-playbooks/`.