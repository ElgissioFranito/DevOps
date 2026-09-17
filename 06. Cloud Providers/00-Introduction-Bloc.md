# Introduction au Bloc 6 — Cloud Providers

> **À lire en premier**, avant la Leçon 1. Ce fichier te dit :
> - de quoi parle ce bloc et **pourquoi il est central en DevOps**,
> - ce qu'il te faut **préparer** avant de commencer,
> - les **8 leçons** du bloc et le **fil rouge** qui les relie,
> - le vocabulaire que tu vas croiser, et ce que tu sauras faire à la fin.

---

## 🎯 De quoi parle ce bloc ?

Dans les blocs précédents, ton code est versionné (Git), tu sais automatiser (Bash/Python), et tu comprends comment les machines communiquent et se sécurisent (réseau, TLS, pare-feu — Bloc 5). Mais en 2025-2026, un DevOps ne gère presque plus de serveurs physiques : il **loue** des serveurs virtuels, du stockage et des bases de données **à la demande** chez des **fournisseurs cloud** (AWS, Azure, GCP…).

Ce bloc répond à ces questions :
- **Comprendre** : qu'est-ce qu'un cloud provider, et les modèles IaaS / PaaS / SaaS ?
- **Construire** : réseau privé (VPC), machines virtuelles (EC2), stockage objet (S3), bases managées (RDS).
- **Sécuriser** : qui peut faire quoi (IAM), au moindre privilège.
- **Choisir et optimiser** : serverless (Lambda) vs machines, et maîtrise de la facture (FinOps).

Objectif de la roadmap : *« concevoir une petite architecture cloud et expliquer pourquoi chaque service existe »*. C'est un **socle bloquant** : l'IaC/Terraform (Bloc 8), Docker (Bloc 9) et Kubernetes (Bloc 10) supposent que tu sais ce qu'est une VM, un réseau cloud, un stockage objet et un rôle IAM.

> 💡 **Bloc progressif et sobre en coûts** : les Leçons 1-2 se pratiquent **en local sans compte** (installation d'un outil, schémas). Les Leçons 3-5 montrent les commandes réelles mais tout est pratiqué via des **scripts de simulation gratuits**. Seule la Leçon 6 demande (optionnellement) un **compte AWS gratuit** pour créer des clés — sinon on simule. **Rien de coûteux ne sera lancé sans te prévenir.**

---

## ✅ Prérequis et préparation

- **Les Blocs 01-05** : Git, Linux/Bash (Bloc 2-3), et surtout le **Bloc 5** (IP, subnet, pare-feu, DNS, load balancer, RBAC, secrets). Chaque leçon du Bloc 6 rappelle le concept du Bloc 5 qu'elle réutilise.
- **Python 3 + pip** : pour installer l'AWS CLI (Leçon 1). Vérifie avec `python3 --version` et `python3 -m pip --version`.
- **Un éditeur** : `nano` (suffit) ou VS Code, pour les notes et scripts (`notes-exercice-XX.md`).
- **(Optionnel) Un compte AWS gratuit** : utile à partir de la Leçon 6 (IAM) pour créer de vraies clés. Pas obligatoire : chaque exercice propose une **version simulée sans compte**.

---

## 🗺️ Les 8 leçons du bloc (et le fil rouge)

Le **fil rouge** : *« construire pas à pas l'architecture cloud d'une application (frontend Angular + backend Spring Boot + base PostgreSQL) : du réseau aux coûts, en comprenant chaque brique »*.

| # | Leçon | Compétence |
|---|-------|------------|
| 1 | Concepts cloud et panorama des providers | Cloud provider, virtualisation, VM, IaaS/PaaS/SaaS, AWS/Azure/GCP/Alibaba, AWS CLI |
| 2 | Réseau cloud : VPC | VPC, subnets public/privé, route tables, IGW, NAT, Security Groups, VPN/bastion |
| 3 | Machines virtuelles : EC2 | Instance, AMI, type, clé SSH, security group, snapshot, autoscaling |
| 4 | Stockage objet : S3 | Bucket, objet/clé, privé par défaut, versioning, lifecycle |
| 5 | Bases managées : RDS | Instance managée, Multi-AZ, backups, endpoint, sécurité d'accès |
| 6 | IAM et sécurité des accès | Utilisateur/groupe/rôle/politique, ARN, moindre privilège, clés, `aws configure` |
| 7 | Serverless : Lambda | Fonction, événement/trigger, facturation à l'exécution, Lambda vs EC2 |
| 8 | FinOps et optimisation des coûts | 4 postes de coût, gaspillages, rightsizing, budgets/alertes, Cost Explorer |

Chaque dossier contient 4 fichiers : `01-lecon.md`, `02-exercice.md`, `03-correction.md`, `04-commandes-references.md`.

> 🔁 **Comment s'articulent les fichiers** : lis d'abord `01-lecon.md` (la théorie + exercice court + checklist), puis fais `02-exercice.md` en autonomie, et compare avec `03-correction.md` (qui réécrit la checklist + conseils). La `04-commandes-references.md` est l'aide-mémoire à garder à côté.


---

## 🧠 Vocabulaire que tu vas croiser

| Terme | C'est quoi ? (1 phrase) | Tu l'apprendras |
|-------|--------------------------|-----------------|
| **Cloud provider** | Une entreprise qui loue puissance de calcul + stockage à la demande (AWS, Azure, GCP, Alibaba) | Leçon 1 |
| **Virtualisation** | Technique qui découpe un serveur physique en plusieurs serveurs virtuels isolés | Leçon 1 |
| **VM (Virtual Machine)** | Un « serveur virtuel » : un OS complet tournant sur un serveur physique partagé | Leçon 1 |
| **IaaS / PaaS / SaaS** | 3 niveaux de location : l'infrastructure / l'infra + cadre applicatif / le logiciel complet | Leçon 1 |
| **AWS CLI** | L'outil en ligne de commande pour piloter AWS (`aws ...`) | Leçon 1 |
| **VPC** | Ton « immeuble privé » dans le cloud : ton réseau isolé | Leçon 2 |
| **Subnet (public / privé)** | Une tranche du VPC ; publique (avec route vers Internet) ou privée (sans) | Leçon 2 |
| **Route table** | Le plan de circulation : quelle route pour quelle destination | Leçon 2 |
| **IGW (Internet Gateway)** | La porte d'entrée/sortie du VPC vers Internet | Leçon 2 |
| **NAT (Gateway)** | Le relais qui permet aux machines privées de *sortir* sur Internet sans être *accessibles* | Leçon 2 |
| **Security Group** | Le pare-feu virtuel attaché à chaque ressource (qui peut entrer ?) | Leçon 2 |
| **VPN** | Tunnel chiffré reliant deux réseaux séparés par Internet comme s'ils étaient adjacents (montage pratique : Bloc 5, Leçons 5a-5b) | Leçon 2 |
| **Bastion** | Petite machine publique servant de porte d'entrée vers les machines privées (alternative au VPN) | Leçon 2 |
| **EC2** | Le service de VM d'AWS (« Elastic Compute Cloud ») | Leçon 3 |
| **Instance** | Une VM en cours d'exécution | Leçon 3 |
| **AMI** | Le modèle de système préinstallé pour créer une instance (ex. Ubuntu) | Leçon 3 |
| **Snapshot** | Une photo instantanée d'un disque, pour sauvegarder/restaurer | Leçon 3 |
| **Autoscaling** | L'ajustement automatique du nombre de machines selon la charge | Leçon 3 |
| **S3** | Le stockage d'objets d'AWS (« Simple Storage Service ») | Leçon 4 |
| **Bucket / objet / clé** | Le conteneur (nom unique mondial) / un fichier + métadonnées / son nom complet | Leçon 4 |
| **Versioning / lifecycle** | Garder les anciennes versions d'un objet / effacer ou archiver automatiquement après N jours | Leçon 4 |
| **CDN** | Réseau de serveurs proches des visiteurs qui servent tes fichiers plus vite | Leçon 4 |
| **RDS** | Les bases de données managées d'AWS (« Relational Database Service ») | Leçon 5 |
| **Endpoint** | L'adresse DNS stable pour joindre ta base (jamais une IP en dur) | Leçon 5 |
| **Multi-AZ** | Une copie synchrone de la base dans une autre zone, pour survivre à une panne | Leçon 5 |
| **IAM** | Le service AWS qui gère qui peut faire quoi (« Identity and Access Management ») | Leçon 6 |
| **ARN** | L'adresse formelle d'une ressource AWS (`arn:aws:...`) | Leçon 6 |
| **Moindre privilège** | Ne donner à chacun que les droits strictement nécessaires | Leçon 6 |
| **Serverless** | Exécuter du code sans gérer de serveur (le cloud gère tout) | Leçon 7 |
| **Lambda / fonction / trigger** | Le service serverless d'AWS / le bout de code exécuté / l'événement qui le réveille | Leçon 7 |
| **Cold start** | Le léger délai du premier appel d'une fonction inactive depuis longtemps | Leçon 7 |
| **FinOps** | La discipline pour maîtriser la facture cloud (informer → optimiser → opérer) | Leçon 8 |
| **Rightsizing** | Adapter la taille d'une machine à son besoin réel | Leçon 8 |
| **Cost Explorer / Budget** | L'outil pour voir les coûts par service / l'enveloppe avec alertes | Leçon 8 |

---

## ✅ Bloc acquis si

Tu peux, **de mémoire** :

- expliquer ce qu'est un cloud provider, la location à la demande, et distinguer IaaS / PaaS / SaaS avec un exemple ;
- dessiner ton **VPC** (subnets public/privé, IGW, NAT, security groups) et expliquer qui est public/privé et pourquoi ;
- décrire le **cycle de vie d'une EC2** (AMI → type → clé → security group → snapshot → stop/terminate) et t'y connecter en SSH ;
- expliquer le stockage **S3** (bucket, objet, privé par défaut, versioning, lifecycle) et choisir le bon stockage ;
- expliquer une base **RDS** (managée, Multi-AZ, backups, endpoint) et pourquoi elle reste privée ;
- répondre aux 3 questions **IAM** (qui / quoi / quelle ressource), écrire une politique au moindre privilège, et protéger tes clés ;
- choisir entre **Lambda et EC2** (court/intermittent vs long/permanent) et schématiser `événement → Lambda → résultat` ;
- lister les **4 postes de coût**, repérer les gaspillages, et mettre un budget avec alertes.

Si tu coches tout, la suite logique de la roadmap t'attend : **Bloc 7 — Bases de données & Data Operations** (où tu approfondiras PostgreSQL), puis **Bloc 8 — IaC/Terraform** (où tu automatiseras en code toute l'architecture construite ici à la main).

---

*Démarre maintenant avec la **Leçon 1** (les concepts cloud et le panorama des providers) dans `01-Concepts-cloud-et-providers/`.*
