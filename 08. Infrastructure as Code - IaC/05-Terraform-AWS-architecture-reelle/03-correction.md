# Correction — Leçon 5 : Terraform chez AWS, l'architecture réelle

> **Bloc 8 · Leçon 5** — Correction pas à pas. (Exercice à réaliser sur un compte AWS free tier ; si tu n'as pas de compte, lis la logique — la Leçon 6 et le projet final s'enchaînent sans problème.)

---

## Étape 0 — Les garde-fous

```bash
aws configure get region
# eu-west-3   ← doit correspondre à var.region du code
```

**Explication** : le budget à 1 $ (console → Budgets → créer, rappel Bloc 6 Leçon 8) est la ceinture de sécurité : si quelque chose coûte, tu es prévenu(e) par e-mail dans l'heure. La région, elle, doit être **cohérente partout** : `aws configure`, `var.region`, et le backend.

## Étape 1 — Le backend à la main (l'œuf avant la poule)

```bash
aws s3api create-bucket \
  --bucket prenom-devops-states-2026 \
  --region eu-west-3 \
  --create-bucket-configuration LocationConstraint=eu-west-3

aws s3api put-bucket-versioning \
  --bucket prenom-devops-states-2026 \
  --versioning-configuration Status=Enabled

aws dynamodb create-table \
  --table-name terraform-locks \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST
```

**Explications ligne par ligne** :
- `s3api create-bucket` : le mode « API » de la CLI (plus explicite que `s3 mb`) ; le `\` en fin de ligne continue la commande (Bloc 2) ; `LocationConstraint` est **obligatoire** hors de la région par défaut `us-east-1` — sans lui, erreur `InvalidLocationConstraint`.
- `put-bucket-versioning Status=Enabled` : active le **versionnage** — chaque ancien state est conservé (règle de la Leçon 3 : restaurer un état corrompu).
- `dynamodb create-table` : une table minimaliste (une seule clé `LockID`, type texte `S`) ; `PAY_PER_REQUEST` = facturation à l'usage — pour un verrou qui dort 99 % du temps, le coût est quasi nul.

## Étape 2 — Le projet

Les 5 fichiers sont dans la leçon (§ 3.1 à 3.3). Points de vigilance corrigés ici :

1. **`versions.tf` contient DEUX providers** (`aws` et `random`) : le `random_password` du `main.tf` a besoin du provider `random` (Leçon 2) — oublié, l'erreur est `Missing required provider`.
2. **Le nom de bucket S3 de documents doit être unique au monde** : remplace `${ton-prefixe-unique}` par un identifiant perso (ex. `elgissio-atelier-documents`). Sinon : erreur `BucketAlreadyExists` (les noms de buckets sont un espace de noms mondial).
3. **Le security group SSH** : remplace `TON.IP.PUBLIQUE/32` par ton IP réelle (taper « mon ip » dans un moteur de recherche). Le `/32` signifie « exactement cette adresse » — une ouverture à `0.0.0.0/0` sur le port 22 serait scannée par des robots en quelques minutes.
4. **Le secret** : `export TF_VAR_mot_de_passe_bdd="..."` (option 1 de la Leçon 4). Il ne quitte pas ta machine ; Terraform le transmet à RDS lors de l'apply.

## Étape 4 — Le rituel, avec les sorties attendues

```bash
terraform init
```

Sortie attendue (extraits) : `Initializing the backend...` puis `Do you wish to proceed with the corresponding state migration?` → **yes** → `Successfully initialized!`. **Explication** : Terraform a détecté le bloc `backend "s3"` (Leçon 3) et propose de **migrer** le state local vers S3. C'est la migration annoncée en Leçon 3 — dorénavant, le registre vit dans ton compartiment (vérifiable avec `aws s3 ls s3://prenom-devops-states-2026/`).

```bash
terraform plan -out=tfplan
```

Les **4 points de contrôle** (exercice, étape 4) :
1. `provider.aws.region = "eu-west-3"` en en-tête de plan ;
2. `instance_type = "t3.micro"` et `instance_class = "db.t3.micro"`, `allocated_storage = 20` ;
3. les AMI via `data.aws_ami.ubuntu` (pas d'ID figé) ;
4. résumé du type : `Plan: 13 to add, 0 to change, 0 to destroy.` — VPC + 3 subnets + IGW + route table + association + SG + EC2 + bucket + subnet group + mot de passe + RDS.

```bash
terraform apply tfplan    # tape yes
```

**À observer** : Terraform crée dans l'ordre des dépendances (VPC → subnets → IGW → route → SG → EC2/S3 → RDS en dernier, plusieurs minutes). Puis les outputs s'affichent : `ip_publique`, `nom_bucket`, `endpoint_bdd` — **note-les**, la Leçon 6 (Ansible) utilisera l'IP, et le projet final réutilisera tout.

## Étape 5 — Vérifier

```bash
terraform state list
# aws_vpc.principal
# aws_subnet.public  aws_subnet.prive_a  aws_subnet.prive_b
# aws_internet_gateway.principal
# aws_route_table.public
# aws_route_table_association.public
# aws_security_group.web
# aws_instance.app
# aws_s3_bucket.documents
# aws_db_subnet_group.prive
# aws_db_instance.bibliotheque
# random_password.mdp_bdd
```

**Explication** : le registre liste les 13 ressources — **et il vit dans S3** (pas de `terraform.tfstate` local : vérifie avec `ls`, il n'y en a plus). Dans la console : VPC (1 réseau, 3 subnets, 1 IGW), EC2 (1 instance `t3.micro` en cours d'exécution), RDS (1 base PostgreSQL, disponible après quelques minutes), S3 (1 bucket). Tout correspond au code : **c'est le drift inverse — zéro drift, l'infra EST le code**.

## Étape 6 — Détruire

```bash
terraform destroy    # tape yes
```

Sortie attendue : la liste des 13 suppressions (dans l'ordre inverse des dépendances : la base d'abord, le VPC en dernier), puis `Destroy complete! Resources: 13 destroyed.` **Vérification console obligatoire** : EC2 → 0 instance ; RDS → 0 base ; VPC → rien de ton préfixe ; S3 → pas de bucket de documents. Le compartiment du backend et `terraform-locks` **restent** : c'est voulu (Leçons suivantes), coût quasi nul.

> 🔑 **Le critère du bloc avance** : tu viens de démontrer « je peux créer puis supprimer une infrastructure entière **depuis le code** ». En Leçon 8 et au projet final, tu enchaîneras destruction **et** reconstruction complète.

---

## Checklist de validation (leçon 5)

- [ ] J'explique comment Terraform trouve les clés AWS (`~/.aws/credentials`) et pourquoi elles ne vont jamais dans le code.
- [ ] Je traduis VPC/subnet/IGW/SG/EC2/S3/RDS en resources Terraform, et je sais pourquoi RDS exige un subnet group de 2 zones.
- [ ] Je connais la différence `resource` (créer) vs `data` (chercher), et pourquoi l'AMI se cherche à la volée.
- [ ] J'ai activé le backend S3 et je comprends le « problème de l'œuf et de la poule ».
- [ ] J'ai appliqué mes garde-fous (budget, lecture du plan, destroy) et tout est bien détruit.
- [ ] Je peux détruire ET reconstruire l'architecture du Bloc 6 depuis le code — le critère du bloc est en marche.

---

## 🧠 Conseils pour la suite

- **Garde ton compartiment S3 et ta table de verrou** : ils serviront au projet final (Leçon 9) pour démontrer destroy/rebuild avec un state partagé.
- **Reruns** : si tu veux rejouer l'apply plusieurs fois, attends 5-10 min après un destroy — les noms RDS ont une « période de veille » : une erreur `DBInstanceAlreadyExists` après un destroy récent est normale.
- **Leçon 6** : garde la sortie `terraform output ip_publique` de ta dernière création si tu veux faire l'exercice Ansible sur une vraie EC2 — sinon tout se pratique sur `localhost`.