# Correction — Leçon 1 : Concepts cloud et panorama des providers

> **Bloc 6 · Leçon 1** — Correction pas à pas.

---

## Étape 1 — Vérifier Python et pip

```bash
python3 --version        # ex. : Python 3.12.x
python3 -m pip --version # ex. : pip 24.x from ... (python 3.x)
```
**Explication** : on vérifie que l'outil `python3` (le programme) et `pip` (le gestionnaire de paquets) existent avant d'installer quoi que ce soit. Si `pip` manque : `sudo apt update && sudo apt install -y python3-pip`.

## Étape 2 — Installer l'AWS CLI

```bash
python3 -m pip install --user --upgrade awscli
aws --version
```
**Explication** : `pip install` télécharge et installe le paquet. `--user` limite l'installation à ton compte (pas global, plus sûr). `--upgrade` garantit la dernière version. `aws --version` affiche par ex. `aws-cli/2.x ...`. Si `aws` n'est pas trouvé, c'est souvent que le dossier `pip --user` (`~/.local/bin`) n'est pas dans le `PATH` ; on peut y ajouter dans `~/.bashrc` :
```bash
export PATH="$HOME/.local/bin:$PATH"
```
*(À relire dans le bloc 2/3, « variables d'environnement ».)*

## Étape 3 — Lancer la configuration (interruptible)

```bash
aws configure
```
**Explication** : l'assistant demande 4 choses : **Access Key ID**, **Secret Access Key**, **région** (ex. `eu-west-3`), **format de sortie** (ex. `json`). Les clés se créent à la Leçon 6 (IAM). Ici, on peut faire **Ctrl+C** après observation ; ces valeurs seront stockées dans `~/.aws/credentials` et `~/.aws/config`.

## Étape 4 — Réflexion

### 1. Définitions avec analogies

| Terme | Définition | Analogie |
|-------|------------|----------|
| **VM** | Un « serveur virtuel » : un OS complet sur un serveur physique partagé | Un **appartement** dans un immeuble |
| **IaaS** | On loue l'infrastructure (serveurs, réseau, stockage) | Un **terrain nu** où tu construis |
| **PaaS** | On loue l'infra **+** un cadre prêt pour l'application | Une **maison prête à décorer** |
| **SaaS** | Logiciel complet loué, on utilise seulement | Un **appartement meublé** |

### 2. Tableau de traduction cloud

| Concept | AWS | GCP | Azure |
|---------|-----|-----|-------|
| Machine virtuelle | **EC2** | **Compute Engine** | **Virtual Machines** |
| Stockage de fichiers | **S3** | **Cloud Storage** | **Blob Storage** |
| Base de données managée | **RDS** | **Cloud SQL** | **Azure SQL Database** |

**Explication** : les **mêmes concepts** existent partout sous d'autres noms. Retiens la **logique** (une VM, un stockage, une base gérée) plus que les noms exacts : savoir « traduire » est plus utile que tout mémoriser.

---

## Checklist de validation (leçon 1)

- [ ] J'explique un cloud provider et la location à la demande.
- [ ] Je définis VM + virtualisation avec une analogie.
- [ ] Je distingue IaaS / PaaS / SaaS avec un exemple.
- [ ] Je situe AWS, Azure, GCP, Alibaba.
- [ ] Je traduis VM / stockage / base entre AWS, GCP, Azure.
- [ ] AWS CLI installée et vérifiée (`aws --version`).

---

## 🧠 Conseils pour la suite

- Ne **pas** lancer de service payant « pour voir » — le compte gratuit suffit pour apprendre, et on **supprime** après les tests.
- Garde `~/.local/bin` dans ton `PATH` pour que `aws` soit trouvable.
- En Leçon 2, tu vas **construire le réseau** (VPC) ; les concepts du bloc 5 (IP, subnet, pare-feu, gateway, load balancer) reviennent à l'échelle cloud.