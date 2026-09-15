# Exercice — Leçon 5 : Terraform chez AWS, l'architecture réelle

> **Bloc 8 · Leçon 5** — ⚠️ **Exercice sur AWS réel (compte free tier), avec garde-fous**. Tout ce qui est créé sera **détruit à la fin**. Si tu n'as pas encore de compte AWS (gratuit), tu peux faire **l'Étapes 1-3 (préparation + plan)** et reporter l'apply — mais lis tout, la logique est la même.

---

## Contexte

C'est la consécration de tout le bloc : le code des Leçons 2-4 devient **une vraie infrastructure AWS** — l'architecture du Bloc 6 (VPC + EC2 + S3 + RDS) décrite en HCL. Tu actives aussi le **backend S3** préparé en Leçon 3. Discipline imposée : `plan` → **lecture** → `apply` → vérification → **`destroy`**.

---

## Énoncé

> 📌 **Rappels d'options** : `aws s3api create-bucket` = créer un compartiment S3 ; `aws dynamodb create-table` = créer une table ; `--profile` = utiliser un profil nommé de `~/.aws/credentials` ; `terraform plan -out=FILE` = sauvegarder le plan dans un fichier.

### Étape 0 — Garde-fous (5 minutes, à ne pas sauter)

1. **Budget** : dans la console AWS, crée une alerte de budget à **1 $** (Bloc 6, Leçon 8 — FinOps). Le free tier couvre tout ce que l'exercice crée, l'alerte est la ceinture de sécurité.
2. **Vérifie ta région** : `aws configure get region` doit renvoyer `eu-west-3` (Paris) — la même que dans le code.
3. **Note l'heure** du début : tout l'exercice doit tenir en moins d'une heure ; le `destroy` arrive à la fin.

### Étape 1 — Préparer le backend (le poulailler avant les poules)

Le state doit vivre dans S3… mais le compartiment doit **exister avant** que Terraform ne s'en serve. On le crée **une fois, à la main** (réflexe du Bloc 6) :

```bash
# Crée le compartiment S3 qui hébergera les states.
# --create-bucket-configuration LocationConstraint : obligatoire hors région us-east-1.
aws s3api create-bucket \
  --bucket mon-nom-unique-terraform-states \
  --region eu-west-3 \
  --create-bucket-configuration LocationConstraint=eu-west-3

# Active le versionnage du compartiment : chaque ancien état est conservé (Leçon 3).
aws s3api put-bucket-versioning \
  --bucket mon-nom-unique-terraform-states \
  --versioning-configuration Status=Enabled

# Crée la table DynamoDB qui sert de verrou (Leçon 3).
aws dynamodb create-table \
  --table-name terraform-locks \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST
```

> 📌 Remplace `mon-nom-unique-terraform-states` par un nom **unique au monde** (ex. `prenom-devops-states-2026`). Note-le dans tes notes : tu le réutiliseras dans `backend.tf`.

### Étape 2 — Écrire le projet

Reprends l'arborescence de la Leçon 4 (`envs/dev/`), et ajoute :

1. **`versions.tf`** : provider `aws` (`~> 5.0`), plus les providers déjà connus si besoin.
2. **`backend.tf`** : celui de la Leçon 3, avec TON nom de compartiment et `key = "atelier-aws/terraform.tfstate"`.
3. **`variables.tf`** : `region` (défaut `eu-west-3`), `nom_projet` (défaut `atelier`), `mot_de_passe_bdd` (**sensitive**, sans défaut — voir étape 3).
4. **`main.tf`** : l'architecture du Bloc 6 en code — la version complète et commentée est dans la leçon (§ 3.3) : VPC, 1 subnet public, 2 subnets privés, IGW, route table + association, security group web, data AMI, instance EC2, bucket S3, subnet group + instance RDS.
5. **`outputs.tf`** : IP publique de l'EC2, nom du bucket, **endpoint de la base** (utile en Leçon 6 pour Ansible).

### Étape 3 — Les secrets, comme convenu (Leçon 4)

Le mot de passe de la base **ne va jamais dans le code**. Option simple pour cet exercice :

```bash
# Exporte la variable dans ton shell : TF_VAR_ + nom de la variable.
export TF_VAR_mot_de_passe_bdd="UnMotDePasseSolide123!"
```

(Variables d'environnement : option n°1 de la Leçon 4. Elles ne quittent pas ta machine.)

### Étape 4 — Le rituel, version cloud

```bash
terraform init      # télécharge le provider AWS ET tente la migration vers le backend → tape yes
terraform plan -out=tfplan   # LECTURE OBLIGATOIRE : compte les "+", vérifie la région et le type d'instance
terraform apply tfplan       # applique le plan SAUVEGARDÉ (pas de nouvelle surprise)
```

**Dans le plan, vérifie ces 4 points avant d'aller plus loin :**
1. La région affichée est bien `eu-west-3` ;
2. Le type d'instance est `t3.micro` et la classe de base `db.t3.micro` (**free tier**) ;
3. Aucun `data "aws_ami"` ne fait référence à une autre région ;
4. Le nombre total de ressources à créer est ~13 (pas 50).

### Étape 5 — Vérifier (la console ne sert qu'à OBSERVER)

```bash
# Liste les ressources dans le registre (backend S3 désormais !).
terraform state list

# L'IP publique de la machine (output).
terraform output ip_publique

# Le compartiment existe bien.
aws s3 ls
```

Puis dans la **console web** (observation seulement) : VPC → ton VPC ; EC2 → ton instance ; RDS → ta base ; S3 → ton bucket. Tout correspond au plan ? C'est l'architecture du Bloc 6, **créée par du code**.

### Étape 6 — Détruire proprement (le réflexe FinOps)

```bash
terraform destroy    # tape yes → tout disparaît : VPC, EC2, S3, RDS
```

Vérifie dans la console que **rien ne reste** (sinon, facture !). Le compartiment S3 du backend et la table DynamoDB **restent** : ils hébergeront les states des futures leçons — leur coût est quasi nul (quelques Ko, la table ne coûte que l'usage).

---

## Livrable

- Le projet `envs/dev/` complet (fonctionnel si tu as fait l'apply).
- `notes-exercice-05.md` : ton nom de compartiment, les 4 vérifications du plan (coche), le résumé du plan final, la liste des ressources dans `state list`, et la confirmation du destroy.

Correction détaillée dans **`03-correction.md`**.