# Introduction au Bloc 8 — Infrastructure as Code (IaC)

> **À lire en premier**, avant la Leçon 1. Ce fichier te dit :
> - de quoi parle ce bloc et **pourquoi il est central en DevOps**,
> - ce qu'il te faut **préparer** avant de commencer,
> - les **9 leçons** du bloc et le **fil rouge** qui les relie,
> - le **vocabulaire** du bloc (réuni et expliqué ici, en plus de chaque leçon),
> - ce que tu sauras faire à la fin — et la **preuve** attendue.

---

## 🎯 De quoi parle ce bloc ?

Aux blocs précédents, tu as tout construit **à la main** : les serveurs (Bloc 6 — console et CLI), la base PostgreSQL (Bloc 7 — commandes, fichiers, sauvegardes), le tout consigné dans un **runbook** (ton carnet de conduite). Tout fonctionne… mais si le serveur disparaît, il faut tout **refaire à la main**, en suivant ton carnet, avec le risque d'oublier ou de te tromper.

Ce bloc répond à ce problème : **transformer ton infrastructure en code**. Au lieu de :

```
SSH → apt install → configuration manuelle        (l'ancien monde : actions)
Code IaC → Terraform → Infrastructure             (le nouveau monde : description)
```

Deux outils partagent le travail : **Terraform** décrit et crée **l'infrastructure** (réseau, machines, bases — il parle au cloud), **Ansible** configure **l'intérieur des machines** (il parle aux machines, par SSH). Tu apprendras aussi à **concevoir** : scalabilité, haute disponibilité, reprise après sinistre (RTO/RPO).

Objectifs de la roadmap : *« Savoir concevoir une infrastructure qui peut supporter davantage d'utilisateurs, résister à certaines pannes, être restaurée après un incident, évoluer sans tout reconstruire »* — et le critère final : *« Tu peux supprimer ton infrastructure et la reconstruire de manière reproductible »*.

> 💡 **Bloc progressif et sobre en coûts** : les Leçons 1-4 se pratiquent **en local, gratuitement** (Terraform avec des providers `local`/`random`, Ansible sur `localhost`). La Leçon 5 se pratique sur **AWS réel** dans le cadre du **free tier** (palier gratuit), avec des garde-fous stricts : budget + alerte, lecture du plan, `destroy` final. **Rien de coûteux ne sera lancé sans te prévenir** ; un scénario 100 % local est proposé partout si tu n'as pas de compte.

---

## ✅ Prérequis et préparation

- **Les blocs 01-07** : Git (Bloc 4), Linux/Bash (Blocs 2-3), le cloud AWS (Bloc 6 — VPC, EC2, S3, RDS, IAM), PostgreSQL (Bloc 7). Chaque leçon du bloc 8 **rappelle** le concept qu'elle réutilise et indique la leçon d'origine.
- **Terraform ≥ 1.5** : installation en Leçon 1 (dépôt officiel HashiCorp) — rien à préparer avant.
- **L'AWS CLI configurée** (Bloc 6, Leçons 1 et 6) : nécessaires à partir de la Leçon 5. Si tu n'as pas encore de compte AWS gratuit : tu peux suivre tout le bloc en scénario local.
- **Ansible** : installé en Leçon 6 (`pip --user`).
- **Un éditeur** : `nano` suffit ; VS Code est confortable pour les fichiers `.tf` et `.yml`.
- **Un compte GitHub/GitLab pour Git** : ton infrastructure EST du code — elle vit dans un dépôt (Bloc 4).

---

## 🗺️ Les 9 leçons du bloc (et le fil rouge)

Le **fil rouge** : *« transformer ton runbook du Bloc 7 en code exécutable — puis prouver, en détruisant tout, que tu peux tout reconstruire depuis le code »*.

| # | Leçon | Compétence |
|---|-------|------------|
| 1 | IaC : pourquoi et concepts | Pourquoi arrêter le manuel ; déclaratif vs impératif ; idempotence ; drift ; le duo Terraform + Ansible ; installation de Terraform |
| 2 | Terraform : ton premier projet | HCL ; `provider`, `resource`, `variable`, `output` ; le workflow `init`/`plan`/`apply`/`destroy` — en local, gratuit |
| 3 | Le state et son backend | Le « registre » de Terraform ; le drift détecté en pratique ; backend S3 + verrou ; `.gitignore` |
| 4 | Modules et environnements | Un module réutilisable (l'équivalent Terraform des fonctions) ; dev/staging/prod ; `.tfvars` ; secrets hors Git |
| 5 | Terraform chez AWS | L'architecture du Bloc 6 en code (VPC, EC2, S3, RDS) ; `data` (AMI) ; backend S3 activé ; garde-fous free tier |
| 6 | Ansible : bases | Architecture « sans agent » ; inventaire ; ad-hoc ; premier playbook idempotent (`apt`, `service`, `copy`) |
| 7 | Ansible avancé | Variables ; templates Jinja2 ; handlers ; **rôles** ; une source, N machines |
| 8 | Scalabilité, HA et DR | Verticale/horizontale ; SPOF ; load balancer ; RTO/RPO ; le plan de reprise en 6 étapes |
| 9 | **Projet récapitulatif** | Détruire et reconstruire : le critère d'acquis **prouvé et chronométré** (scénario A local ou B AWS) |

**Comment les leçons s'enchaînent** : la 1 pose le *pourquoi* et installe l'outil ; les 2-4 construisent la maîtrise de Terraform **en local** (langage → mémoire → organisation) ; la 5 fait passer le même code **au cloud réel** ; les 6-7 font de même pour Ansible (bases → rôle professionnel) ; la 8 remonte au **pourquoi architectural** ; la 9 rassemble tout en une **preuve**.

---

## 📖 Vocabulaire du bloc (réuni et expliqué)

Chaque leçon a son propre glossaire ; celui-ci les **réunit**, pour relire vite. Les sigles sont définis ici **une fois pour toutes** :

| Terme / sigle | Définition (une ligne) | Leçon |
|---------------|------------------------|-------|
| **IaC** | *Infrastructure as Code* : décrire l'infrastructure dans des fichiers texte qu'un outil exécute | 1 |
| **Déclaratif / impératif** | Décrire le résultat voulu (plan d'architecte) / la liste des gestes (recette) | 1 |
| **Idempotence** | Propriété d'une opération qui, répétée, donne le même résultat | 1, 2, 6, 7 |
| **Drift** | Écart entre l'infrastructure décrite dans le code et la réalité | 1, 3 |
| **Terraform** | L'outil IaC déclaratif qui crée l'infrastructure chez le cloud | 1-5 |
| **Ansible** | L'outil qui configure l'intérieur des machines, via SSH, sans agent | 1, 6-7 |
| **OpenTofu** | Le « jumeau » open source de Terraform, compatible (mêmes fichiers, mêmes commandes) | 1 |
| **HCL** | *HashiCorp Configuration Language* : le langage des fichiers Terraform | 1, 2 |
| **Provider** | Le plugin « traducteur » de Terraform vers une plateforme (local, random, AWS…) | 2, 5 |
| **Resource / data** | Une ressource concrète à créer / une ressource existante à lire (ex. l'AMI) | 2, 5 |
| **Variable / output** | Une valeur réglable de l'extérieur / une information affichée à la fin | 2 |
| **Workflow** | `init` (télécharger les plugins) → `plan` (prévisualiser) → `apply` (exécuter) → `destroy` (démonter) | 2 |
| **State** | Le registre (`terraform.tfstate`) de ce que Terraform a créé, avec les ID réels | 2-5 |
| **Backend** | L'emplacement où vit le state (local par défaut ; S3 à distance) | 3, 5 |
| **Verrouillage (lock)** | Pendant un apply, le state est verrouillé contre les écritures concurrentes | 3, 5 |
| **Module** | Un dossier de code Terraform réutilisable, appelé depuis le projet racine | 4 |
| **Environnement** | Une copie à usage dédié : **dev** (développement), **staging** (pré-production), **prod** (production) | 4 |
| **`.tfvars`** | Un fichier qui fournit des valeurs aux variables, lu par `plan`/`apply` | 4 |
| **AMI** | L'image système préinstallée d'une EC2 (Bloc 6, Leçon 3) | 5 |
| **AZ** | *Availability Zone* : une zone de disponibilité, l'un des data centers isolés d'une région AWS | 5, 8 |
| **Free tier** | Le palier gratuit d'AWS : quotas mensuels offerts (ex. 750 h de machine `t3.micro`) | 5 |
| **Inventaire (Ansible)** | Le fichier listant les machines à configurer, en groupes | 6, 7 |
| **Playbook / play / task** | Le fichier YAML des consignes / un « chapitre » / une consigne unitaire | 6, 7 |
| **Fact** | Une information collectée sur une machine cible (OS, IP, mémoire…) | 6, 7 |
| **Template (`.j2`) / Jinja2** | Un modèle de fichier dont les `{{ ... }}` sont remplis au déploiement | 7 |
| **Handler / notify** | Une action exécutée une seule fois, seulement si notifiée par un vrai changement | 7 |
| **Rôle (role)** | Un dossier structurant une brique de configuration réutilisable | 7 |
| **Scalabilité** | Capacité à supporter une augmentation de charge — **verticale** (agrandir la machine) ou **horizontale** (ajouter des machines) | 8 |
| **HA / SPOF / fault tolerance** | Haute disponibilité (réduire les interruptions) / point de défaillance unique / continuer malgré une panne | 8 |
| **Load balancer / health check** | Le répartiteur de charge / le test de santé qui détourne le trafic d'un serveur mort | 8 |
| **DR / RTO / RPO** | Reprise après sinistre / temps max hors service / quantité max de données perdues | 8, 9 |
| **Runbook DR** | Le plan de reprise écrit, en étapes numérotées, avec les commandes exactes | 8, 9 |
| **Stack** | L'ensemble cohérent des briques d'un projet (infra + config + docs) | 9 |

> 📌 **Mentions « à définir sans creuser »** (conforme à la roadmap) : **CloudFormation** (l'équivalent Terraform, propriétaire AWS), **Pulumi** (l'IaC dans un vrai langage de programmation), **Puppet, Chef, Salt** (anciens outils de gestion de configuration) — Leçon 1 · **Ansible Vault** (le chiffrement des variables à secrets) et **Ansible Galaxy** (le catalogue public de rôles) — Leçon 7 · **Autoscaling** (ajuster automatiquement le nombre de machines selon la charge) — Leçon 8.

---

## ✅ Bloc acquis si

De mémoire et **par la démonstration** (leçon 9) :

- expliquer **pourquoi** l'IaC a remplacé la configuration manuelle, et distinguer **déclaratif/impératif** ;
- exécuter le workflow Terraform (`init`/`plan`/`apply`/`destroy`) et **lire un plan** (`+` créer, `~` modifier, `-` supprimer) ;
- expliquer le **state**, son backend (S3 + verrou), et pourquoi il ne va **jamais** dans Git ;
- organiser le code en **modules** et **environnements** (dev/staging/prod), secrets hors Git ;
- reconstruire **l'architecture du Bloc 6** (VPC, EC2, S3, RDS) en code, avec les garde-fous free tier ;
- écrire un **playbook Ansible** idempotent, puis un **rôle** (variables, templates, handlers) qui configure N machines ;
- **concevoir** : scalabilité verticale/horizontale, HA, SPOF, DR, **RTO/RPO** — et répondre aux deux questions *« que se passe-t-il si le serveur tombe ? »* et *« si la base est détruite ? »* ;
- ✋ **supprimer ton infrastructure et la reconstruire de manière reproductible** — chronométré et documenté.

---

## 🧭 Prochaine étape (et la suite de la roadmap)

Avec ce bloc, ton flux devient : **le code décrit → le cloud crée → Ansible configure → le sinistre se répare**. Il manque une pièce : l'application elle-même. Au Bloc 7, elle était installée sur la machine, à la main ; à la Leçon 7, Ansible l'installe, mais **paquet par paquet, dans le système de la machine**. Le **Bloc 9 — Docker et conteneurisation** change le paradigme : l'application sera **emballée** (avec ses dépendances) dans des **conteneurs** identiques, exécutables partout — et ton dépôt IaC accueillera naturellement Docker (le rôle Ansible installe Docker, Terraform crée les machines qui le portent).

Tu es prêt — commence par la **Leçon 1** : `01-IaC-concepts-et-pourquoi/`.

---

*Démarre maintenant avec la **Leçon 1** dans `01-IaC-concepts-et-pourquoi/` — le pont depuis le Bloc 7 t'y attend.*