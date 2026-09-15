# Référence rapide — Leçon 5 : Terraform + AWS

> Bloc 8 · Leçon 5 — Aide-mémoire.

## Traduction Bloc 6 → Terraform

| Concept (Bloc 6) | Resource Terraform | Notation clé |
|------------------|--------------------|--------------|
| VPC | `aws_vpc` | `cidr_block = "10.0.0.0/16"` |
| Subnet | `aws_subnet` | `vpc_id`, `cidr_block`, `availability_zone` |
| Subnet public | `aws_subnet` | `map_public_ip_on_launch = true` |
| IGW | `aws_internet_gateway` | `vpc_id` |
| Route table | `aws_route_table` + `aws_route_table_association` | `route { cidr_block, gateway_id }` |
| Security group | `aws_security_group` | `ingress` / `egress` (défaut-deny) |
| EC2 | `aws_instance` | `ami`, `instance_type`, `subnet_id` |
| AMI | `data "aws_ami"` | `most_recent`, `owners`, `filter` |
| S3 | `aws_s3_bucket` | nom unique au monde |
| RDS | `aws_db_subnet_group` + `aws_db_instance` | 2 subnets privés de zones différentes |

## Les clés

- Terraform lit `~/.aws/credentials` (créé par `aws configure`, Bloc 6 Leçon 6).
- Jamais de clés dans les `.tf` ; un utilisateur IAM **dédié** au moindre privilège.
- `aws configure get region` doit = `var.region`.

## resource vs data

```hcl
resource "aws_instance" "app" { ... }     # CRÉE la chose
data "aws_ami" "ubuntu" { ... }           # CHERCHE la chose (lecture seule)
```

## Backend (activé ici)

```bash
# Une fois à la main (le poulailler avant les poules) :
aws s3api create-bucket --bucket MON-NOM --region eu-west-3 --create-bucket-configuration LocationConstraint=eu-west-3
aws s3api put-bucket-versioning --bucket MON-NOM --versioning-configuration Status=Enabled
aws dynamodb create-table --table-name terraform-locks --attribute-definitions AttributeName=LockID,AttributeType=S --key-schema AttributeName=LockID,KeyType=HASH --billing-mode PAY_PER_REQUEST

# Puis dans le projet :
terraform init      # propose la migration du state → tape yes
```

## Le rituel version cloud

```bash
export TF_VAR_mot_de_passe_bdd="..."   # secret hors Git (Leçon 4)
terraform plan -out=tfplan             # LECTURE OBLIGATOIRE
terraform apply tfplan                 # applique exactement ce plan
terraform state list                   # vérifier le registre
terraform output ip_publique           # récupérer une valeur
terraform destroy                      # FinOps : tout détruire en fin de session
```

## Garde-fous free tier

- Budget + alerte à 1 $ AVANT de commencer.
- Types gratuits : `t3.micro` (EC2), `db.t3.micro` (RDS, 20 Go).
- `destroy` en fin de session + vérification console.
- Jamais `-auto-approve` à la main.