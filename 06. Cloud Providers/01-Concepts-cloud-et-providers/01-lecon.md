# Leçon 1 — Concepts cloud et panorama des providers

> **Bloc 6 · Cloud Providers** — Leçon 1 sur 8
> 🧭 **Pont depuis le Bloc 5 (Réseautage et sécurité)** : tu sais comment les machines communiquent (protocoles), comment on filtre (pare-feu), et comment on protège les échanges (TLS, certificats). Tous ces savoirs vont maintenant **prendre vie à grande échelle** dans le cloud. Cette leçon pose les **fondations** : qu'est-ce qu'un cloud provider, la virtualisation, les modèles de service (IaaS/PaaS/SaaS), le panorama AWS / Azure / GCP / Alibaba, et l'installation de l'outil **AWS CLI** qui servira à piloter AWS en ligne de commande.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Expliquer** ce qu'est un « cloud provider » et le passage du modèle « j'achète mes serveurs » au modèle « je loue à la demande ».
2. **Définir** la **VM** (machine virtuelle) et la **virtualisation**, avec une analogie claire.
3. **Distinguer** les trois modèles de services cloud : **IaaS**, **PaaS**, **SaaS**.
4. **Situer** les principaux providers (AWS, Azure, GCP, Alibaba) et **traduire** les mêmes concepts d'un cloud à l'autre.
5. **Installer et vérifier** l'outil en ligne de commande `aws` (AWS CLI) pour les leçons suivantes.

---

## 2. Explication simple

### 2.1 Le « pourquoi » : pourquoi le cloud existe ?

Historiquement, une entreprise **achetait ses propres serveurs** : des machines physiques installées dans une salle (le « data center »), qu'il fallait acheter, refroidir, surveiller, réparer, remplacer. C'est cher en **argent** (achat) et en **temps** (maintenance).

Le **cloud** inverse le modèle : un **cloud provider** (AWS, Azure, Google…) possède et entretient d'immenses salles de serveurs, et te **loue** une petite partie de cette puissance **à la demande**. Tu paies **ce que tu utilises** (c'est un **coût variable**), sans rien acheter ni entretenir toi-même.

> 💡 **Analogie** : plutôt que **d'acheter une voiture** (capital, entretien, assurance), tu **loues** un véhicule quand tu en as besoin. Le cloud, c'est « louer de la puissance de calcul et du stockage à la demande ».

**À quel moment choisit-on le cloud ?** Dès qu'on veut **démarrer vite**, **évoluer facilement** (monter/diminuer la puissance) et **sans gérer de matériel**. La roadmap le place après le Bloc 5 (réseau & sécurité) car le cloud **réutilise tous les concepts réseau** que tu viens d'apprendre, mais à grande échelle.

### 2.2 Le « comment » : la virtualisation et la VM

Pour louer efficacement, il faut **partager un serveur physique** en plusieurs **serveurs virtuels** isolés : c'est la **virtualisation**.

> 💡 **Analogie** : un grand immeuble (le serveur physique) est découpé en **appartements** (les VM), chacun avec sa porte, son électricité et son compteur, sans que les voisins se gênent.

Une **VM (Virtual Machine)** = un « serveur virtuel » : un système d'exploitation complet (ex. Ubuntu) qui s'exécute sur un serveur physique partagé, isolé des autres.

```
Serveur physique
├── VM Ubuntu   (ton application 1)
├── VM Windows  (ton application 2)
└── VM Ubuntu   (ton application 3)
```

Un seul serveur physique héberge donc plusieurs VM. Chaque VM a l'air d'être une machine à part entière, mais elle est créée et détruite en **quelques minutes** (contrairement à une machine physique).

### 2.3 Le « quand » : les trois modèles de service — IaaS / PaaS / SaaS

Le cloud se décline en **trois niveaux**, selon **ce que le provider gère** à ta place. Plus tu montes vers le haut, moins tu gères, mais moins tu contrôles.

| Modèle | Sigle | Le provider gère | Toi, tu gères | Analogie |
|--------|-------|------------------|----------------|----------|
| **Infrastructure** | **IaaS** = Infrastructure as a Service | Les serveurs, le réseau, le stockage | L'OS, l'application, les données | **Terrain nu** : tu construis ta maison |
| **Plateforme** | **PaaS** = Platform as a Service | Serveurs + cadre applicatif (OS, runtime) | Ton application et ses données | **Maison prête à décorer** : les murs sont là |
| **Logiciel** | **SaaS** = Software as a Service | Tout, même l'application | Utiliser et configurer | **Appartement meublé** : tu emménages |

Exemples concrets :
- **IaaS** : AWS EC2 (une VM louée), Azure VM, GCP Compute Engine → c'est le niveau principal des DevOps.
- **PaaS** : un service qui héberge déjà ton application sans que tu gères le serveur (ex. AWS Elastic Beanstalk).
- **SaaS** : Gmail, Slack, Google Drive — des **logiciels** que tu utilises sans rien installer.

> 🔑 **À retenir** : le bloc Cloud se concentre surtout sur **IaaS** (tu loues des briques et tu construis), car c'est là que le réseau, la sécurité et l'architecture prennent sens.

### 2.4 Le panorama des providers — AWS en profondeur

La roadmap le dit clairement : **« AWS en profondeur + notions Azure/GCP »**. On n'apprend pas trois clouds en profondeur ; on **apprend les concepts une fois, sur AWS**, puis on **les traduit** ailleurs (les concepts sont les mêmes, seuls les noms changent).

| Provider | Propriétaire | C'est quoi | Quand le choisir |
|----------|--------------|------------|------------------|
| **AWS** (Amazon Web Services) | Amazon | Le cloud le plus répandu et le plus riche en services | **Ton choix principal** — le standard de l'industrie |
| **Azure** | Microsoft | Cloud Microsoft, très présent en entreprise (Active Directory, .NET) | Environnements déjà équipés Microsoft |
| **GCP** (Google Cloud Platform) | Google | Cloud de Google, bon en data et en Kubernetes managé | Projets data / Google |
| **Alibaba Cloud** | Alibaba | Cloud chinois | Projets ciblant le marché asiatique |

> 💡 **Traduire d'un cloud à l'autre** : le concept de « VM », de « stockage de fichiers », de « base de données managée » et de « gestion des accès » existe partout. Ex. chez AWS on dit **S3**, chez GCP **Cloud Storage**, chez Azure **Blob Storage** — c'est le **même type de service** sous un autre nom.

### 2.5 L'outil de travail : l'AWS CLI

Pour piloter AWS **en ligne de commande** (comme tu pilotes Linux en terminal), on utilise l'**AWS CLI** (Command Line Interface, « interface en ligne de commande ») : l'outil officiel, souvent nommé **`aws`**. La plupart des leçons de ce bloc l'utiliseront. On l'installe en Python (via `pip`) car c'est la méthode universelle et simple.

> ⚠️ **Prérequis réaliste** : certaines commandes AWS (créer une vraie VM, un stockage) exigent un **compte AWS** gratuit à l'inscription, mais certaines ressources sont payantes (même peu). Chaque leçon proposera donc des **alternatives simulables en local gratuitement** quand c'est possible, avec les commandes réelles à exécuter le jour où tu créerás ton compte.

---

## 📖 Vocabulaire / Abréviations

> Définitions d'une ligne pour ne jamais être perdu(e), à consulter avant les exemples.

- **Cloud provider** : entreprise qui loue de la puissance de calcul et du stockage (AWS, Azure, Google, Alibaba).
- **Data center** : bâtiment qui abrite des centaines de serveurs physiques.
- **Virtualisation** : découpage d'un serveur physique en plusieurs serveurs virtuels isolés.
- **VM (Virtual Machine)** : un « serveur virtuel » (un OS complet s'exécutant sur un serveur physique partagé).
- **Coût variable** : ce qu'on paie selon l'utilisation (vs un achat fixe).
- **IaaS** (Infrastructure as a Service) : on loue l'infrastructure (serveurs, réseau, stockage).
- **PaaS** (Platform as a Service) : on loue l'infrastructure + un cadre applicatif.
- **SaaS** (Software as a Service) : logiciel complet loué, on ne fait qu'utiliser.
- **AWS** (Amazon Web Services) : le cloud provider d'Amazon, le plus répandu.
- **Azure** : le cloud provider de Microsoft.
- **GCP** (Google Cloud Platform) : le cloud provider de Google.
- **AWS CLI / `aws`** : le programme en ligne de commande qui pilote AWS.
- **`pip`** : le gestionnaire de paquets de Python, qui permet d'installer des outils Python.

---

## 3. Exemples concrets

> On prépare ici l'outil `aws` pour la suite. Toutes les commandes sont commentées ligne par ligne.

### 3.1 Vérifier Python et `pip`

```bash
# Affiche la version de Python (python3 = la version 3 du langage Python).
python3 --version

# Vérifie que pip (le gestionnaire de paquets Python) est présent.
python3 -m pip --version

# Si pip manque, on l'installe sur Ubuntu/Debian (sudo = super utilisateur).
# sudo apt update
# sudo apt install -y python3-pip
```

### 3.2 Installer l'AWS CLI

```bash
# -m pip = "utilise le module pip de ce Python" (la commande pip).
# --user  = installe pour ton compte (pas le système entier).
# --upgrade = met à jour s'il existe déjà.
python3 -m pip install --user --upgrade awscli
```

### 3.3 Vérifier l'installation

```bash
# Affiche la version de aws : l'installation a réussi.
aws --version
```

### 3.4 Lancer la configuration (interactif, à la demande)

```bash
# Lance l'assistant qui demande tes clés d'accès et ta région.
# (Les clés se créent à la Leçon 6 — IAM. Tu peux interrompre avec Ctrl+C.)
aws configure
```

> 💡 Pas de panique si `aws configure` te demande des clés : la Leçon 6 (IAM) expliquera précisément ce que sont ces clés et comment les créer en toute sécurité.

---

## 4. Bonnes pratiques modernes (2025-2026)

- **AWS en profondeur, les autres en notions** : maîtriser un cloud donne la logique ; savoir « traduire » vers Azure/GCP suffit (la roadmap le recommande).
- **Toujours le moindre coût pour apprendre** : compte gratuit, ressources temporaires, suppression dès que l'exercice est fini.
- **AWS CLI plutôt que la console web** : en DevOps, on automatise en ligne de commande et en scripts, pas en cliquant.
- **Ne jamais committer de clé** : les clés d'accès AWS (vues en Leçon 6) ne vont **jamais** dans Git (rappel Bloc 5 — secrets).
- **Installer via `pip --user`** sur une machine partagée pour ne pas toucher au système global.

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi c'est dangereux/inefficace | ✅ Version correcte |
|----------------|-------------------------------------|---------------------|
| Apprendre AWS + Azure + GCP en profondeur en même temps | Surcharge, on ne maîtrise rien | **AWS en profondeur**, Azure/GCP en notions |
| Lancer des services payants « pour voir » | Facture inattendue | Compte gratuit + **suppression** après test |
| Installer AWS CLI avec `sudo` sans raison | Risque sur le système, version figée | `pip install --user` pour ton compte |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`**, la correction dans **`03-correction.md`**.

**Énoncé court** : vérifie que Python et `pip` sont là, installe l'AWS CLI, vérifie avec `aws --version`, et lance `aws configure` (interruptible). Puis rédige dans `notes-exercice-01.md` : les définitions de **VM**, **IaaS**, **PaaS**, **SaaS** avec une analogie chacune, et le tableau de traduction AWS ↔ GCP ↔ Azure (VM, stockage de fichiers, base de données).

---

## 7. Correction détaillée de l'exercice

> La correction complète est dans **`03-correction.md`**. On y confirme les commandes, et surtout le tableau de traduction qui servira tout le bloc : quand tu verras EC2, S3, RDS, IAM, tu sauras déjà leurs équivalents chez GCP/Azure.

---

## 8. Checklist de validation

- [ ] J'explique ce qu'est un cloud provider et le passage « achat → location ».
- [ ] Je définis la VM et la virtualisation avec une analogie.
- [ ] Je distingue IaaS / PaaS / SaaS avec un exemple pour chacun.
- [ ] Je situe AWS (approfondi), Azure, GCP et Alibaba (« à mentionner »).
- [ ] Je traduis un même concept (VM, stockage, base) d'un cloud à l'autre.
- [ ] J'ai installé l'AWS CLI avec `pip --user` et vérifié avec `aws --version`.

---

🧭 **Pont vers la suite** — Tu sais maintenant **où** tu vas (quel cloud) et **sur quel modèle** (IaaS). Mais le réseau n'y est pas encore : comment ton application et ta base vont-elles communiquer **à l'abri d'Internet** ? C'est le rôle de la **Leçon 2 — le réseau cloud (VPC)**.

---

*Prochaine étape :* Leçon 2 — **Réseau cloud (VPC)** dans `02-Reseau-cloud-VPC/`.
