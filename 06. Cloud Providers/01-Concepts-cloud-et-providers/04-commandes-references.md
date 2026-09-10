# Référence rapide — Leçon 1 : Concepts cloud & AWS CLI

> Bloc 6 · Leçon 1 — Aide-mémoire.

## Concepts clés
- **Cloud provider** : loue puissance de calcul + stockage à la demande (AWS, Azure, GCP, Alibaba).
- **Virtualisation** : un serveur physique partagé en plusieurs VM isolées.
- **VM (Virtual Machine)** : un « serveur virtuel » (OS complet).
- **IaaS** : louer l'infrastructure (serveurs, réseau, stockage) → l'essentiel DevOps.
- **PaaS** : infrastructure + cadre applicatif prêt pour l'app.
- **SaaS** : logiciel complet, on utilise seulement.
- **AWS en profondeur** ; Azure/GCP/Alibaba = notions + traduction.

## Mini-tableau de traduction
| Concept | AWS | GCP | Azure |
|---------|-----|-----|-------|
| Machine virtuelle | EC2 | Compute Engine | Virtual Machines |
| Stockage de fichiers | S3 | Cloud Storage | Blob Storage |
| Base de données managée | RDS | Cloud SQL | Azure SQL Database |

## Installation AWS CLI
```bash
python3 -m pip install --user --upgrade awscli
aws --version
# si "aws introuvable" :
# 1. ajoute au PATH dans ~/.bashrc :
# export PATH="$HOME/.local/bin:$PATH"
```

## Configuration AWS
```bash
aws configure  # Access Key ID + Secret Key (Leçon 6/IAM) + région + format de sortie
```
- Clés stockées dans `~/.aws/credentials` et config dans `~/.aws/config`.
- ⚠️ **Ne jamais mettre les clés dans Git** (Bloc 5 — secrets).

## Bonnes pratiques
- Compte gratuit AWS, ressources temporaires, suppression après test.
- Pas de `sudo` sauf besoin ; `pip --user` pour ton compte.
- Apprendre la logique, « traduire » plutôt que mémoriser 3 clouds.