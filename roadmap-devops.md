# Roadmap DevOps — Ce que tu dois savoir à la fin de chaque bloc

Cette roadmap est **très large** : elle ne signifie pas que tu dois devenir expert de chaque technologie. L'objectif DevOps est plutôt de comprendre **comment tout s'assemble**, puis d'être réellement opérationnel sur quelques outils principaux.

Je te donne pour chaque bloc :

-   🎯 **Objectif final**
-   📚 **Ce que tu dois connaître**
-   🧠 **Jargon expliqué**
-   🛠️ **Ce que tu dois savoir faire concrètement**
-   ✅ **Critère pour considérer le bloc acquis**

----------

# 1. Bases du SDLC

**SDLC = Software Development Life Cycle**, c'est-à-dire le cycle de vie d'un logiciel, de l'idée jusqu'à sa maintenance.

### 🎯 À la fin, tu dois comprendre

Comment une application passe de :

```
Besoin
  ↓
Développement
  ↓
Tests
  ↓
Build
  ↓
Déploiement
  ↓
Production
  ↓
Monitoring
  ↓
Maintenance
```

### 📚 Tu dois connaître

#### Cycle de vie du développement

Comprendre les différentes étapes :

-   analyse du besoin
-   conception
-   développement
-   tests
-   intégration
-   déploiement
-   maintenance

#### Planification et backlog

**Backlog** = liste des tâches/fonctionnalités à réaliser.

Exemple :

```
BACKLOG
├── Ajouter authentification
├── Ajouter gestion utilisateurs
├── Ajouter export Excel
└── Corriger bug login
```

Tu dois comprendre :

-   User Story
-   tâche
-   bug
-   sprint
-   priorité
-   estimation

#### Tests et déploiement

Comprendre la différence entre :

-   test unitaire
-   test d'intégration
-   test end-to-end
-   environnement de développement
-   staging
-   production

### 🧠 Jargon

**Build** : transformation du code source en quelque chose d'exécutable/déployable.

**Artifact** : résultat produit par le build.

Exemple :

```
Code Java
   ↓
Maven build
   ↓
application.jar
```

**Production** : environnement utilisé par les vrais utilisateurs.

**Staging** : environnement proche de la production utilisé avant le déploiement réel.

### 🛠️ Tu dois savoir faire

Prendre une petite application Spring Boot (variante : NestJS) et expliquer :

```
Git
 ↓
Build
 ↓
Tests
 ↓
Artifact
 ↓
Deployment
 ↓
Production
```

### ✅ Bloc acquis si

Tu peux expliquer **sans hésiter comment ton code passe de ton ordinateur jusqu'au serveur de production**.

----------

# 2. Système d'exploitation — Linux

C'est un bloc **fondamental**.

### 🎯 Objectif

Être capable d'administrer un serveur Linux sans dépendre constamment d'une interface graphique.

### 📚 Tu dois connaître

### Commandes Shell

Maîtriser notamment :

```
pwd
ls
cd
cp
mv
rm
mkdir
touch
cat
less
head
tail
grep
find
sort
awk
sed
cut
xargs
```

Mais surtout comprendre les concepts :

```
stdin
stdout
stderr
pipe |
redirection >
redirection >>
```

Exemple :

```
cat app.log | grep ERROR
```

### SSH

**SSH = Secure Shell**

Permet de se connecter à distance :

```
ssh user@server
```

Tu dois comprendre :

-   clé privée
-   clé publique
-   `authorized_keys`
-   authentification
-   port SSH
-   `scp`
-   `rsync`

### Permissions

Linux fonctionne notamment avec :

```
r = read
w = write
x = execute
```

Exemple :

```
-rwxr-xr--
```

Tu dois comprendre :

```
chmod
chown
chgrp
sudo
```

### Gestion des paquets

Selon la distribution :

```
apt
dnf
yum
```

### Éditeurs

Au minimum :

```
nano
vim
```

Tu n'as pas besoin de devenir expert Vim immédiatement.

### Distributions

Comprendre :

-   Debian
-   Ubuntu
-   CentOS/RHEL

Et surtout comprendre qu'une **distribution** est un système Linux assemblé avec un ensemble de logiciels, outils et gestionnaires de paquets.

> 🔵 **À mentionner, définir, ne pas creuser — Windows Server** : l'alternative à Linux pour administrer un serveur (interface graphique, PowerShell comme équivalent de Bash). Pertinent surtout en environnement .NET/Windows Server ; secondaire pour ton profil actuel.

### 🧠 Jargon

**Processus** = programme en cours d'exécution.

```
Node.js
   ↓
processus
```

**Daemon/service** = programme qui fonctionne généralement en arrière-plan.

Exemple :

```
nginx
postgresql
docker
```

**PID** = identifiant d'un processus.

### 🛠️ Tu dois savoir faire

Sur un serveur Ubuntu vierge :

-   créer un utilisateur
-   configurer SSH
-   installer Nginx
-   installer PostgreSQL
-   gérer un service
-   consulter les logs
-   vérifier les ports
-   modifier les permissions
-   rechercher un fichier
-   tuer un processus
-   vérifier CPU/RAM/disque

### ✅ Bloc acquis si

Tu peux recevoir :

> « Voici un serveur Ubuntu vide. Déploie cette application et diagnostique pourquoi elle ne répond pas. »

…et savoir où chercher.

----------

# 3. Scripting et programmation

### 🎯 Objectif

Arrêter de faire manuellement les tâches répétitives.

Le DevOps doit être capable de dire :

> « Pourquoi je fais ça 50 fois à la main ? Je vais l'automatiser. »

### 📚 Langages

Tu n'as pas besoin d'être expert de tous.

> ℹ️ **Point important** : **Linux et Python ne font pas partie de tes acquis présumés** — ce sont des **thèmes à apprendre** dans cette roadmap, pas des prérequis déjà maîtrisés. Linux est le bloc 2 ci-dessous ; Python est présenté ici comme un langage à découvrir, pas comme une compétence supposée.

Priorité (à acquérir) — ton langage principal de scripting :

```
Bash
```

À découvrir ensuite (tu n'as jamais fait de Python) :

```
Python
```

Puis éventuellement :

```
Go
```

> 🔵 **À mentionner, définir, ne pas creuser — Rust** : langage bas niveau très performant et sûr (pas de garbage collector, sécurité mémoire garantie à la compilation). De plus en plus utilisé pour réécrire des outils DevOps critiques (ex. une partie de l'écosystème npm/Deno). Pas une priorité d'apprentissage, mais bon à savoir lire si tu tombes sur du code d'outil.
>
> 🟡 **À mentionner, définir, ne pas creuser — Ruby** : langage de scripting historiquement lié à des outils DevOps plus anciens (Chef utilise Ruby comme DSL, Vagrant aussi). Peu utile si tu n'utilises pas ces outils.

### Bash

Savoir faire :

```
if
for
while
variables
fonctions
arguments
exit codes
```

Exemple conceptuel :

```
./deploy.sh production
```

### Python

> ℹ️ **Point important** : tu n'as **jamais fait de Python**. Cette section est là pour te guider **au moment où tu commenceras** à apprendre ce langage — pas avant. Elle reste utile car Python est très courant en DevOps (Ansible, scripts d'automatisation), mais ce n'est **pas** un prérequis de départ.

Comprendre :

-   fichiers
-   JSON
-   YAML
-   HTTP
-   API
-   subprocess
-   scripts
-   gestion d'erreurs

### Formats de données — YAML / JSON

Ce sont **les deux formats que tu croiseras absolument partout** en DevOps : manifests Kubernetes, pipelines GitHub Actions/GitLab CI, `docker-compose.yml`, réponses d'API REST, fichiers de configuration Ansible/Terraform.

#### 🎯 Objectif final

Lire et écrire du YAML/JSON sans erreur de syntaxe, et convertir mentalement de l'un à l'autre.

#### 📚 Tu dois connaître

**JSON** — structuré avec des accolades et crochets :

```json
{
  "name": "api",
  "replicas": 3,
  "ports": [80, 443]
}
```

**YAML** — la même chose, mais avec de l'indentation (espaces, jamais de tabulation) :

```yaml
name: api
replicas: 3
ports:
  - 80
  - 443
```

Points d'attention fréquents :

-   l'indentation YAML est **significative** (une erreur d'espace casse le fichier)
-   YAML supporte les commentaires (`#`), pas JSON
-   JSON est plus strict et plus facile à valider/parser automatiquement ; YAML est plus lisible pour un humain

#### 🧠 Jargon

**Sérialisation** : transformer une structure de données (objet, dictionnaire) en texte (JSON/YAML) pour la stocker ou l'envoyer sur le réseau.

#### 🛠️ Tu dois savoir faire

Lire un manifest Kubernetes ou un fichier `docker-compose.yml` et identifier ses erreurs d'indentation ou de syntaxe à l'œil, sans validateur.

#### ✅ Critère pour considérer le bloc acquis

Tu peux écrire un `docker-compose.yml` ou un pipeline `.github/workflows/*.yml` sans copier-coller un exemple ligne à ligne.

> 🔵 **À mentionner, définir, ne pas creuser — XML** : format de données plus ancien que JSON/YAML, basé sur des balises (`<tag>valeur</tag>`). Dans ton stack, tu le croiseras surtout dans `pom.xml` (Maven) et la configuration Spring/legacy Spring Boot (et SOAP). Peu prioritaire dans l'écosystème Next.js/NestJS (orienté JSON, en variante secondaire), mais **pertinent pour toi côté Java**. À savoir lire, pas à maîtriser.

### Automatisation

Exemples :

```
Backup
 ↓
Compression
 ↓
Upload
 ↓
Notification
```

### API / SDK / REST

**API = Application Programming Interface**

Une application peut demander quelque chose à une autre application.

Exemple :

```
Script Python
     ↓ HTTP
GitHub API
     ↓
Liste des repositories
```

**SDK = Software Development Kit**

Ensemble d'outils permettant d'utiliser plus facilement une API.

**REST = REpresentational State Transfer.**

C'est le style d'architecture le plus courant pour les API web (c'est ce que tu utilises déjà dans tes contrôleurs **Spring Boot** — en variante **NestJS** — sans forcément l'avoir nommé). Principes clés :

-   chaque **ressource** a une URL (`/users`, `/users/42`)
-   les actions passent par les **verbes HTTP** :

```
GET    /users       → lister
GET    /users/42    → lire un utilisateur
POST   /users       → créer
PUT    /users/42    → remplacer
PATCH  /users/42    → modifier partiellement
DELETE /users/42    → supprimer
```

-   **stateless** : chaque requête contient toute l'information nécessaire (le serveur ne garde pas de session en mémoire entre deux appels)
-   les **codes de statut HTTP** indiquent le résultat (`200 OK`, `201 Created`, `400 Bad Request`, `401 Unauthorized`, `404 Not Found`, `500 Internal Server Error`)

**Critère pour considérer ce point acquis** : tu peux dessiner une API REST simple (ressources + verbes + codes de statut) pour un cas donné, sans hésiter sur le verbe ou le code à utiliser.

### Analyse des journaux

**Log = journal d'événements produit par une application ou un système.**

Exemple :

```
2026-09-02 10:20 INFO Server started
2026-09-02 10:21 ERROR Database connection failed
```

Tu dois savoir rechercher :

```
ERROR
WARNING
IP
timestamp
request ID
```

### 🛠️ Tu dois savoir faire

Créer un script qui :

```
vérifie un serveur
   ↓
vérifie l'espace disque
   ↓
vérifie un service
   ↓
analyse les logs
   ↓
envoie un rapport
```

### ✅ Bloc acquis si

Tu identifies spontanément les tâches répétitives et sais les automatiser avec **Bash** (et **Python** une fois que tu l'auras appris — ce n'est pas un acquis à ce stade).

----------

# 4. Git — contrôle de version

Tu dois être **très à l'aise** avec Git.

### 🎯 Objectif

Pouvoir travailler proprement sur du code seul ou en équipe.

### 📚 Commandes

Maîtriser :

```
git clone
git status
git add
git commit
git push
git pull
git fetch
git branch
git switch
git merge
git rebase
git log
git diff
git stash
git reset
git revert
```

### Branching

Comprendre :

```
main
 │
 ├── feature/login
 ├── feature/export
 └── fix/auth
```

### Feature branch

Une branche destinée à développer une fonctionnalité.

### Pull/Merge Request

**Pull Request (PR)** / **Merge Request (MR)** :

> « J'ai terminé mon travail, vérifiez mon code avant de l'intégrer. »

### Conflit Git

Deux personnes ont modifié la même partie du code.

Tu dois savoir :

```
identifier le conflit
       ↓
comprendre les changements
       ↓
résoudre
       ↓
tester
       ↓
commit
```

### Plateformes

Comprendre le rôle de :

-   GitHub
-   GitLab

Ce sont des plateformes autour de Git, avec notamment :

-   repositories
-   issues
-   pull requests
-   CI/CD
-   permissions
-   releases

> 🔵 **À mentionner, définir, ne pas creuser — SVN (Subversion)** : système de contrôle de version **centralisé** (un seul serveur central, contrairement à Git où chaque clone a l'historique complet). Encore présent sur certains vieux projets.
>
> 🟡 **À mentionner, définir, ne pas creuser — CVS** : l'ancêtre de SVN, quasiment obsolète aujourd'hui. À connaître seulement par culture générale si tu croises le terme dans un projet très ancien.

### 🛠️ Tu dois savoir faire

Travailler avec :

```
feature branch
    ↓
commit
    ↓
push
    ↓
Pull Request
    ↓
review
    ↓
merge
```

Et récupérer un projet après une mauvaise manipulation Git.

### ✅ Bloc acquis si

Une équipe te donne un repository et tu peux travailler dessus proprement sans « casser » l'historique ou avoir peur des conflits.

----------

# 5. Réseautage et sécurité

C'est particulièrement important pour le DevOps.

### 🎯 Objectif

Comprendre **comment les machines communiquent** et pourquoi une application est ou n'est pas accessible.

### 📚 Protocoles

Comprendre :

```
TCP
UDP
HTTP
HTTPS
DNS
SSH
```

### TCP

**TCP** assure une communication fiable entre deux machines.

Exemple :

```
Client ───── TCP ─────> Serveur
```

### HTTP

Protocole utilisé notamment par les applications web.

```
GET /users
POST /login
```

### HTTPS

HTTP + chiffrement TLS.

### DNS

Transforme :

```
api.example.com
```

en adresse IP :

```
192.168.x.x
```

### ICMP — diagnostic réseau

**ICMP = Internet Control Message Protocol.** Ce n'est pas un protocole de transport de données comme TCP/UDP, mais un protocole de **diagnostic et de signalisation réseau**. C'est lui qui est utilisé par les deux commandes les plus basiques du dépannage réseau :

```
ping api.example.com     → la machine répond-elle ?
traceroute api.example.com  → par quels routeurs passe la requête ?
```

**Critère d'acquisition** : face à « je n'arrive pas à joindre le serveur », `ping` et `traceroute`/`tracepath` sont les tout premiers réflexes, avant même de regarder les logs applicatifs.

### Adressage IP

Comprendre :

```
IP (IPv4 principalement)
subnet
gateway
CIDR
private IP
public IP
```

Exemple :

```
192.168.1.10/24
```

> 🔵 **À mentionner, définir, ne pas creuser — IPv6** : la version plus récente d'IPv4, avec un espace d'adressage bien plus grand (adresses en hexadécimal, ex. `2001:db8::1`). La plupart des infrastructures actuelles fonctionnent encore majoritairement en IPv4 ; IPv6 est bon à reconnaître, pas urgent à maîtriser.
>
> 🔵 **À mentionner, définir, ne pas creuser — IPSec** : suite de protocoles permettant de chiffrer/authentifier le trafic IP, utilisée notamment pour monter des VPN site-à-site. À connaître de nom si tu croises une configuration VPN d'entreprise ; ces tunnels sont montrés en pratique dans la leçon « VPN et tunnels sécurisés » ci-dessous (WireGuard/OpenVPN).

### Pare-feu

**Firewall** = système qui décide quels flux réseau sont autorisés ou bloqués.

Exemple :

```
Internet
   ↓
Firewall
   ↓
Port 443 → autorisé
Port 22  → autorisé
Port 5432 → bloqué
```
---

### VPN et tunnels sécurisés

**VPN (Virtual Private Network)** = tunnel chiffré entre ta machine et un réseau distant : tu circules dedans comme dans un réseau privé local, alors que le transport passe par Internet.

```
Ta machine ════ tunnel chiffré (VPN) ════> Serveur / réseau distant
```

#### 🎯 Objectif final

Savoir monter et utiliser un VPN **au quotidien** (ordinateur, téléphone, serveurs) avec WireGuard, et savoir choisir entre WireGuard, OpenVPN et un simple tunnel SSH.

#### 📚 Ce que tu dois connaître

- VPN, tunnel, interface virtuelle (`wg0`, `tun0`)
- **WireGuard** : clés (`wg genkey`/`wg pubkey`), `AllowedIPs`, `wg-quick`, split tunnel vs full tunnel, `PersistentKeepalive`
- **OpenVPN** : PKI à base de certificats (lien avec la section TLS)
- **Tunnels SSH** : `ssh -L` (local), `-R` (distant), `-D` (proxy SOCKS) — pour un besoin ponctuel
- Usages quotidiens : Wi-Fi public, administration à distance sans exposer SSH, homelab, accès à un réseau privé (y compris un VPC cloud, voir Bloc 6)

#### 🧠 Jargon expliqué

**Tunnel** = canal chiffré qui « emballe » ton trafic pour le faire traverser Internet sans être lisible.

**Split tunnel** = seul une partie du trafic (le réseau privé) passe par le VPN ; le reste sort normalement.

**Full tunnel** = tout le trafic passe par le VPN (ex. Wi-Fi public non fiable).

**Fuite DNS** = les requêtes de noms de domaine sortent en clair malgré le VPN.

#### 🛠️ Ce que tu dois savoir faire concrètement

Générer des clés, écrire une config serveur et une config client (`wg0.conf`), ouvrir le port VPN au pare-feu, démarrer le tunnel, le vérifier (`wg show`, `ping`) et le rendre persistant (`systemctl enable wg-quick@wg0`).

#### ✅ Critère pour considérer le point acquis

Tu peux monter un tunnel WireGuard (serveur + client) de zéro, expliquer tes `AllowedIPs`, et dire quand préférer SSH (ponctuel) ou OpenVPN (compatibilité).

### Reverse Proxy et Load Balancing

#### 🎯 Objectif final

Comprendre comment exposer correctement des applications et répartir le trafic entre plusieurs serveurs.

#### 📚 Ce que tu dois connaître

##### Reverse Proxy

-   Nginx
-   Traefik
-   HAProxy
-   Apache (avec le module `mod_proxy`)

Architecture :

```
Internet
   ↓
Nginx
   ↓
Backend
```

**Apache HTTP Server** peut lui aussi faire reverse proxy, mais il est surtout connu comme serveur web/serveur d'applications historique (souvent utilisé avec PHP). Nginx est aujourd'hui plus répandu comme reverse proxy pur (plus léger, meilleures performances en forte charge), mais Apache reste très présent sur d'anciens projets ou dans des environnements mutualisés.

> 🔵 **À mentionner, définir, ne pas creuser — Tomcat** : serveur d'application Java qui exécute des applications web. **Spring Boot embarque Tomcat par défaut** (mode application autonome `.jar`), c'est donc directement lié à ton stack. On ne voit un Tomcat « séparé » que sur du legacy Java/Spring non conteneurisé (fichiers `.war`). Sans lien avec l'écosystème Next.js/NestJS (variante secondaire).

##### Load Balancer

```
             ┌── Server 1
Internet ─── LB ── Server 2
             └── Server 3
```

Comprendre :

-   distribution du trafic
-   health check
-   failover
-   session
-   L4
-   L7

#### 🧠 Jargon expliqué

**Reverse proxy** = serveur intermédiaire placé devant les applications.

**Load Balancer (LB)** = répartiteur de charge entre plusieurs serveurs.

**Health check** = vérification automatique qu'un serveur fonctionne.

**Failover** = basculement vers une autre instance en cas de panne.

**L4** = équilibrage basé principalement sur TCP/UDP.

**L7** = équilibrage basé sur le niveau applicatif, par exemple HTTP.

#### 🛠️ Ce que tu dois savoir faire concrètement

Configurer :

```
Internet
   ↓
Nginx
   ↓
Angular (frontend statique)
   ↓
Spring Boot (backend)
   ↓
PostgreSQL
```

> ℹ️ **Angular est un frontend statique** : les fichiers compilés sont **servis par Nginx** (contrairement à Next.js/Node qui exécutent un serveur). Nginx fait donc reverse proxy vers ton backend Spring Boot (`/api`), et peut aussi servir directement le frontend Angular. Variante éventuelle : NestJS/Next.js.

Puis :

```
Nginx
  ↓
 ┌────────────┐
 ↓            ↓
API 1        API 2
```

#### ✅ Critère pour considérer le bloc acquis

Tu sais expliquer pourquoi on place Nginx/Traefik/HAProxy devant une application et tu sais configurer un reverse proxy simple.

---

### Sécurité

Comprendre :

-   authentification
-   autorisation
-   chiffrement
-   TLS
-   certificats
-   secrets
-   moindre privilège

### openssl — manipuler TLS/certificats en pratique

**openssl** est l'outil en ligne de commande de référence pour tout ce qui touche au chiffrement et aux certificats.

#### 🎯 Objectif final

Ne plus subir un certificat TLS comme une boîte noire : savoir en générer un, l'inspecter, comprendre pourquoi une connexion HTTPS échoue.

#### 🛠️ Ce que tu dois savoir faire concrètement

```
openssl genrsa -out key.pem 2048          # générer une clé privée
openssl req -new -key key.pem -out csr    # créer une demande de certificat
openssl x509 -in cert.pem -text -noout    # inspecter un certificat existant
openssl s_client -connect example.com:443 # tester une connexion TLS
```

#### ✅ Critère pour considérer le bloc acquis

Face à « certificat expiré » ou « connexion TLS refusée », tu sais utiliser `openssl` pour diagnostiquer avant de chercher ailleurs.

### RBAC / ABAC — modèles de contrôle d'accès

Ce sont les deux façons standard de répondre à la question **« qui a le droit de faire quoi ? »**, que tu retrouveras aussi bien dans Kubernetes que dans IAM AWS/GCP.

#### 🎯 Objectif final

Comprendre la différence entre les deux modèles et savoir lequel utiliser selon le contexte.

#### 📚 Ce que tu dois connaître

**RBAC = Role-Based Access Control.** On assigne des **rôles** (ex. `admin`, `lecteur`, `développeur`) et chaque rôle a un ensemble de permissions fixes.

```
Utilisateur → Rôle "admin" → peut lire/écrire/supprimer
Utilisateur → Rôle "lecteur" → peut seulement lire
```

**ABAC = Attribute-Based Access Control.** Les permissions dépendent d'**attributs** (qui, quoi, quand, où) évalués dynamiquement, plutôt que d'un rôle fixe.

```
Règle : autoriser l'accès SI
  département = "RH"
  ET heure ∈ [8h-18h]
  ET ressource.confidentialité = "interne"
```

#### 🧠 Jargon expliqué

**RBAC** = modèle simple, statique, facile à auditer — utilisé par défaut dans Kubernetes (`Role`, `RoleBinding`).
**ABAC** = modèle plus fin et flexible, mais plus complexe à mettre en place et à auditer.

#### ✅ Critère pour considérer le bloc acquis

Tu peux expliquer pourquoi Kubernetes utilise RBAC par défaut, et dans quel cas un système aurait plutôt besoin d'ABAC (règles conditionnelles fines).

> 🔵 **À mentionner, définir, ne pas creuser — LDAP** : protocole d'annuaire utilisé pour centraliser l'authentification (utilisateurs, groupes) dans une organisation — souvent couplé à Active Directory. Pertinent si le CUA ou une future entreprise utilise un annuaire centralisé pour les comptes.
>
> 🔵 **À mentionner, définir, ne pas creuser — SMTP** : protocole d'envoi d'e-mails. En DevOps, on le croise surtout pour configurer l'envoi de notifications/alertes (ex. un pipeline CI/CD ou un système de monitoring qui envoie un e-mail en cas d'échec).

### 🧠 Shift-Left Security

**Shift-left** = faire les contrôles de sécurité **le plus tôt possible**, notamment pendant le développement.

Au lieu de :

```
Code → Production → problème de sécurité
```

on cherche :

```
Code
 ↓
Scan sécurité
 ↓
Tests
 ↓
Build
 ↓
Production
```

### DevSecOps

**DevSecOps = Development + Security + Operations.**

L'objectif est d'intégrer la sécurité dans tout le cycle de développement au lieu d'attendre la production.

> ℹ️ Cette section mentionne Docker et les pipelines CI/CD avant que tu aies vu les blocs correspondants (9. Docker et 11. CI/CD). C'est normal : retiens seulement les *principes* (scanner tôt, ne jamais commiter de secret) pour l'instant, et reviens relire cette section une fois ces deux blocs acquis pour la mettre en pratique concrètement.

#### 🎯 Objectif final

Être capable d'intégrer des contrôles de sécurité dans :

```
Code
 ↓
Git
 ↓
CI
 ↓
Security Scan
 ↓
Build
 ↓
Docker
 ↓
Deploy
 ↓
Production
```

#### 📚 Ce que tu dois connaître

##### Gestion des secrets

Ne jamais mettre dans Git :

```
DATABASE_PASSWORD
JWT_SECRET
AWS_ACCESS_KEY
API_KEY
```

Connaître :

-   variables d'environnement
-   secrets CI/CD
-   Kubernetes Secrets
-   AWS Secrets Manager
-   Vault

##### Vulnérabilités

Comprendre :

-   CVE
-   vulnérabilité
-   patch
-   dépendance vulnérable
-   container vulnérable

##### SAST

**SAST = Static Application Security Testing**

Analyse le **code source sans l'exécuter** pour détecter des problèmes de sécurité.

```
Code
 ↓
SAST
 ↓
Vulnérabilités détectées
```

##### DAST

**DAST = Dynamic Application Security Testing**

Teste une application **en fonctionnement**.

```
Application démarrée
        ↓
       DAST
        ↓
Tests de sécurité
```

##### Dependency Scanning

Analyse les dépendances du projet.

Exemple :

```
Spring Boot
 ↓
pom.xml
 ↓
dépendance vulnérable (ex. Log4j)
 ↓
alerte
```

##### Container Scanning

Analyse une image Docker à la recherche de vulnérabilités.

#### 🧠 Jargon expliqué

**CVE** = identifiant public d'une vulnérabilité connue.
**SAST** = analyse du code.
**DAST** = analyse de l'application en fonctionnement.
**Dependency scanning** = recherche de vulnérabilités dans les bibliothèques utilisées.
**Secret scanning** = recherche accidentelle de mots de passe/API keys dans Git.

#### 🛠️ Ce que tu dois savoir faire concrètement

Créer un pipeline :

```
git push
 ↓
Tests
 ↓
SAST
 ↓
Dependency Scan
 ↓
Docker Build
 ↓
Container Scan
 ↓
Deploy
```

Et empêcher le déploiement si une vulnérabilité critique est détectée.

#### ✅ Critère pour considérer le bloc acquis

Tu sais identifier **où et comment intégrer la sécurité dans un pipeline CI/CD**, gérer les secrets correctement et comprendre un rapport de vulnérabilité.

### 🛠️ Tu dois savoir faire

Diagnostiquer :

> « Mon frontend arrive à accéder au backend localement mais pas depuis Internet. »

Tu dois penser à :

```
DNS
 ↓
IP
 ↓
route
 ↓
firewall
 ↓
port
 ↓
service
 ↓
application
```

### ✅ Bloc acquis si

Un problème réseau ne te paraît plus « magique ».

Tu peux expliquer **où circule une requête HTTP et à quel niveau elle peut être bloquée**.

----------

# 6. Cloud Providers

La roadmap mentionne :

```
AWS
Azure
GCP
Certifications Cloud
```

### 🎯 Objectif

Comprendre les concepts cloud, pas apprendre trois clouds simultanément en profondeur.

Je te conseille :

> **AWS en profondeur + notions Azure/GCP.**

> 🔵 **À mentionner, définir, ne pas creuser — GCP (Google Cloud Platform)** : les concepts génériques ci-dessous (VPC, IAM…) s'y transposent directement — VM = Compute Engine, S3 = Cloud Storage, RDS = Cloud SQL. Bon à savoir traduire d'un cloud à l'autre plutôt qu'à maîtriser en détail.
>
> 🟡 **À mentionner, définir, ne pas creuser — Azure** : cloud provider Microsoft, pertinent surtout en environnement d'entreprise déjà équipé Microsoft (Active Directory, .NET). Mêmes concepts que AWS/GCP sous d'autres noms.
>
> 🟡 **À mentionner, définir, ne pas creuser — Alibaba Cloud** : cloud provider chinois, pertinent surtout pour des projets ciblant le marché asiatique. Peu probable dans ton contexte.

### 📚 Concepts fondamentaux

### VM

**Virtual Machine** = serveur virtuel.

Exemple :

```
Serveur physique
├── VM Ubuntu
├── VM Windows
└── VM Ubuntu
```

### VPC

**Virtual Private Cloud** = réseau privé virtuel dans le cloud.

Tu y trouves notamment :

-   subnets
-   routes
-   firewall/security groups
-   gateways


### Cloud Networking

#### 🎯 Objectif final

Savoir construire le réseau d'une application dans le cloud et comprendre comment les différentes ressources communiquent.

#### 📚 Ce que tu dois connaître

-   VPC
-   subnet
-   route table
-   Internet Gateway
-   NAT Gateway
-   Security Group
-   DNS
-   Load Balancer
-   VPN
-   IP publique
-   IP privée

Architecture typique :

```
Internet
   ↓
DNS
   ↓
Load Balancer
   ↓
Public Subnet
   ↓
Private Subnet
   ↓
Application
   ↓
Database
```

#### 🧠 Jargon expliqué

**VPC** = réseau privé isolé dans le cloud.
**Subnet** = subdivision d'un réseau.
**Route Table** = règles indiquant où envoyer les paquets.
**NAT Gateway** = permet notamment à des ressources privées de sortir vers Internet sans être directement exposées.
**VPN** = connexion réseau sécurisée entre deux réseaux.
**Security Group** = règles de trafic appliquées aux ressources cloud.

#### 🛠️ Ce que tu dois savoir faire concrètement

Construire un réseau où :

```
Internet
   ↓
Load Balancer
   ↓
Application privée
   ↓
Database privée
```

La base de données **ne doit pas être directement accessible depuis Internet**.

#### ✅ Critère pour considérer le bloc acquis

Tu peux dessiner et expliquer le réseau d'une application cloud, notamment **qui est public, qui est privé et pourquoi**.

### EC2

Chez AWS, une machine virtuelle.

### S3

Stockage d'objets.

Exemple :

```
application
   ↓
S3
   ↓
images / PDF / backups
```

### RDS

Base de données managée.

Au lieu de gérer toi-même :

```
PostgreSQL
backup
patch
disque
```

le cloud prend une partie de l'administration en charge.

### IAM

**Identity and Access Management**

Gestion :

```
Qui ?
Peut faire quoi ?
Sur quelle ressource ?
```

### Lambda

**Serverless function**.

Tu fournis une fonction :

```
event
 ↓
Lambda
 ↓
résultat
```

sans gérer directement le serveur.

---

### FinOps

#### 🎯 Objectif final

Savoir utiliser le cloud sans créer une facture inutilement élevée.

#### 📚 Ce que tu dois connaître

-   coût des VM
-   coût du stockage
-   coût du réseau
-   ressources inutilisées
-   autoscaling
-   budgets
-   alertes de coût
-   pricing
-   rightsizing

#### 🧠 Jargon expliqué

**FinOps** = discipline consistant à gérer et optimiser les coûts du cloud.

**Rightsizing** = choisir une taille de ressource adaptée au besoin.

Exemple :

```
Application utilise :
10% CPU

VM :
32 CPU
```

La machine est probablement surdimensionnée.

**Autoscaling** = augmenter/réduire automatiquement les ressources selon la charge.

**Budget** = limite financière surveillée.

#### 🛠️ Ce que tu dois savoir faire concrètement

Identifier :

```
VM inutilisée
      ↓
coût inutile
      ↓
suppression
```

Et mettre en place :

```
Budget
   ↓
Monitoring des coûts
   ↓
Alert
   ↓
Action
```

#### ✅ Critère pour considérer le bloc acquis

Tu peux expliquer **combien coûte approximativement une architecture cloud**, identifier les ressources coûteuses/inutilisées et proposer des optimisations.

---

### 🛠️ Tu dois savoir faire

Déployer une petite application :

```
Internet
   ↓
Load Balancer
   ↓
Application
   ↓
Database
```

avec stockage et permissions correctement configurés.

### ✅ Bloc acquis si

Tu peux concevoir une petite architecture cloud et expliquer **pourquoi chaque service existe**.

----------


# 7. Bases de données & Data Operations

### 🎯 Objectif final

Être capable d'administrer une base de données utilisée par une application en production et de protéger ses données contre les pannes.

### 📚 Ce que tu dois connaître

#### Administration

-   PostgreSQL
-   MySQL
-   utilisateurs
-   rôles
-   permissions
-   connexions
-   configuration
-   ressources

> 🔵 **À mentionner, définir, ne pas creuser — MongoDB** : base NoSQL orientée documents (pas de schéma fixe, pas de jointures SQL). Les concepts de backup/réplication ci-dessous existent aussi en NoSQL mais avec des mécanismes différents (replica set plutôt que primary/replica classique). Bon à savoir que « tout n'est pas du SQL », sans en faire une priorité vu ton stack actuel (PostgreSQL/MySQL).
>
> 🟡 **À mentionner, définir, ne pas creuser — OracleDB** : SGBD propriétaire d'entreprise, alternative payante à PostgreSQL/MySQL, présent surtout en grands comptes et legacy avec licences coûteuses. Peu probable dans ton contexte.

#### Backup

Comprendre :

-   backup complet
-   backup incrémental
-   restauration
-   point-in-time recovery
-   stratégie de backup

#### Migration

Comprendre :

-   migration de schéma
-   migration de données
-   versionnement
-   rollback
-   outils de migration

Exemples :

```
Flyway
Liquibase
(Prisma Migrate — variante écosystème Node/Next.js)
```

#### Réplication

Comprendre :

```
Primary
   ↓
Replica
```

La base principale transmet ses changements à une ou plusieurs répliques.

#### Haute disponibilité

Comprendre comment éviter :

```
Database
    ↓
    💥
    ↓
Application indisponible
```

grâce notamment à :

-   réplication
-   failover
-   replicas
-   managed databases

### 🧠 Jargon expliqué

**Backup** = copie permettant de restaurer les données.
**Restore** = restauration d'un backup.
**Replication** = copie des données vers une autre base.
**Primary** = base principale qui reçoit généralement les écritures.
**Replica** = copie de la base.
**Failover** = basculement automatique vers une autre instance lorsqu'une instance tombe.
**High Availability (HA)** = architecture conçue pour rester disponible malgré certaines pannes.
**Migration** = modification contrôlée de la structure ou des données de la base.

### 🛠️ Ce que tu dois savoir faire concrètement

Tu dois pouvoir :

```
Créer DB
   ↓
Configurer utilisateurs
   ↓
Faire migration
   ↓
Faire backup
   ↓
Supprimer DB
   ↓
Restaurer backup
```

Et comprendre :

```
Application
     ↓
Primary DB
     ↓
Replica
```

### ✅ Critère pour considérer le bloc acquis

Tu peux mettre une base PostgreSQL en production, **la sauvegarder, la restaurer, effectuer une migration et expliquer comment éviter une perte de données**.

---

### Cache applicatif — Redis / Memcache

Un bloc manquant de la roadmap initiale : avant d'ajouter des serveurs ou de scaler la base de données, la mise en cache est souvent le levier de performance le plus rentable.

#### 🎯 Objectif final

Comprendre pourquoi et quand mettre des données en cache, et savoir configurer un cache simple devant une base de données.

#### 📚 Ce que tu dois connaître

Sans cache :

```
Requête
   ↓
Application
   ↓
Base de données (lente, sollicitée à chaque requête)
```

Avec cache :

```
Requête
   ↓
Application
   ↓
Cache (Redis) → réponse rapide si déjà présent
   ↓ (si absent : "cache miss")
Base de données
   ↓
Cache mis à jour
```

Comprendre :

-   cache hit / cache miss
-   TTL (Time To Live) — durée de vie d'une donnée en cache
-   invalidation de cache
-   éviction (que faire quand le cache est plein)
-   cache de session (stocker les sessions utilisateurs plutôt qu'en base)

##### Redis

Le cache en mémoire le plus utilisé aujourd'hui. Stocke des paires clé-valeur, très rapide, supporte aussi des structures plus riches (listes, sets, files d'attente).

Exemple :

```
SET user:42 "{...}" EX 3600   # stocke pendant 1h
GET user:42                    # lecture quasi instantanée
```

##### Memcache (Memcached)

Plus ancien et plus simple que Redis (clé-valeur pur, pas de structures avancées, pas de persistance sur disque). Encore utilisé quand on n'a besoin que d'un cache basique.

#### 🧠 Jargon expliqué

**Cache hit** = la donnée demandée est trouvée dans le cache.
**Cache miss** = la donnée n'est pas dans le cache, il faut aller la chercher à la source (base de données).
**TTL** = durée après laquelle une entrée de cache expire automatiquement.
**Invalidation de cache** = supprimer/mettre à jour une entrée de cache devenue obsolète — **c'est la partie la plus difficile** ("il n'y a que deux choses difficiles en informatique : le cache invalidation et nommer les variables").

#### 🛠️ Ce que tu dois savoir faire concrètement

Mettre en cache le résultat d'une requête coûteuse dans une application **Spring Boot** (via **Spring Cache** : `@EnableCaching` et `@Cacheable`) avec Redis, avec un TTL cohérent, et savoir expliquer ce qui se passe si le cache tombe (l'application doit continuer à fonctionner, juste plus lentement — jamais de panne totale à cause du cache). Variante équivalente possible côté NestJS.

#### ✅ Critère pour considérer le bloc acquis

Tu sais identifier quelles données méritent d'être mises en cache, choisir un TTL raisonnable, et expliquer le risque de données périmées ("stale data") si l'invalidation est mal gérée.

> 🔵 **À mentionner, définir, ne pas creuser — Infinispan** : cache distribué de l'écosystème Java/JBoss, alternative à Redis. Niche, pertinent seulement en environnement Red Hat/JBoss.

---

# 8. Infrastructure as Code — IaC

C'est un bloc **très important pour devenir DevOps moderne**.

### 🎯 Objectif

Ne plus configurer les infrastructures manuellement.

Au lieu de :

```
SSH
 ↓
apt install
 ↓
configuration manuelle
```

tu écris :

```
Code IaC
 ↓
Terraform
 ↓
Infrastructure
```

### Architecture système

### 🎯 Objectif final

Savoir concevoir une infrastructure qui peut :

-   supporter davantage d'utilisateurs
-   résister à certaines pannes
-   être restaurée après un incident
-   évoluer sans tout reconstruire

### 📚 Ce que tu dois connaître

#### Scalabilité

**Scalabilité = capacité à supporter une augmentation de charge.**

Deux approches :

**Verticale :**

```
2 CPU
 ↓
8 CPU
```

**Horizontale :**

```
1 serveur
 ↓
3 serveurs
```

#### Haute disponibilité

Éviter :

```
Server unique
    ↓
    💥
    ↓
Tout est arrêté
```

Préférer :

```
       Load Balancer
       /           \
   Server 1      Server 2
```

#### Fault Tolerance

Capacité à continuer à fonctionner malgré certaines pannes.

#### Disaster Recovery

**DR = Disaster Recovery**

Plan permettant de récupérer le système après un incident majeur.

Exemples :

-   perte d'un serveur
-   suppression de données
-   panne régionale
-   corruption de données

#### Backup

Une infrastructure sérieuse doit avoir une stratégie de sauvegarde.

### 🧠 Jargon expliqué

**Scalabilité** = capacité à augmenter la capacité du système.
**Vertical scaling** = augmenter la puissance d'une machine.
**Horizontal scaling** = ajouter des machines.
**High Availability** = réduire les interruptions de service.
**Fault Tolerance** = continuer malgré une panne.
**Disaster Recovery** = reconstruire/récupérer le système après un incident majeur.
**RTO** = temps maximal acceptable pour restaurer le service.
**RPO** = quantité maximale de données qu'on accepte de perdre.

### 🛠️ Ce que tu dois savoir faire concrètement

Concevoir :

```
                 Load Balancer
                /             \
               ↓               ↓
           Server 1        Server 2
               \               /
                ↓             ↓
                 Database
                    ↓
                  Backup
```

Et répondre à :

> « Que se passe-t-il si Server 1 tombe ? »

> « Que se passe-t-il si la base est détruite ? »

### ✅ Critère pour considérer le bloc acquis

Tu sais concevoir une architecture **scalable, disponible et récupérable après une panne**.

### Terraform

**Terraform** permet de décrire une infrastructure sous forme de code.

Exemple conceptuel :

```
Je veux :

1 serveur
1 réseau
1 base PostgreSQL
1 firewall
```

Terraform crée cette infrastructure.

### Concepts Terraform

Comprendre :

```
Provider
Resource
Variable
Output
State
Module
Plan
Apply
Destroy
```

### State

**Terraform state** = état connu de l'infrastructure gérée par Terraform.

C'est une notion **très importante**.

> 🟡 **À mentionner, définir, ne pas creuser — AWS CloudFormation** : équivalent de Terraform mais natif et propriétaire à AWS (ne fonctionne pas sur d'autres clouds), écrit en JSON/YAML. Terraform reste préférable si tu veux rester multi-cloud.
>
> 🟡 **À mentionner, définir, ne pas creuser — Pulumi** : alternative à Terraform qui permet d'écrire l'infrastructure dans un vrai langage de programmation (Python, TypeScript) plutôt qu'en HCL. Intéressant si tu préfères coder ton infra plutôt qu'écrire une syntaxe déclarative dédiée, mais Terraform reste le standard du marché.

### Ansible

Ansible est principalement utilisé pour la **configuration et l'automatisation des machines**.

Exemple :

```
Terraform
   ↓
crée VM
   ↓
Ansible
   ↓
installe/configure Nginx
```

> 🔵 **À mentionner, définir, ne pas creuser — Puppet** : outil de gestion de configuration concurrent d'Ansible, plus ancien, avec une architecture agent/master (Ansible fonctionne sans agent, via SSH). Encore présent dans certaines grandes infrastructures legacy.
>
> 🟡 **À mentionner, définir, ne pas creuser — Chef, Salt** : autres outils de gestion de configuration, moins utilisés aujourd'hui qu'Ansible. Chef utilise Ruby comme DSL (contrairement à Ansible qui utilise du YAML), ce qui le rend plus puissant mais avec une courbe d'apprentissage plus raide.

### Différence simplifiée

```
Terraform → "Je veux cette infrastructure."

Ansible   → "Je veux que cette machine soit configurée ainsi."
```

### Environnements

Comprendre :

```
dev
staging
production
```

et comment éviter de mélanger leurs configurations/secrets.

### 🛠️ Tu dois savoir faire

Recréer une infrastructure entière à partir du code.

Par exemple :

```
Terraform
 ↓
VM
 ↓
réseau
 ↓
firewall
 ↓
Ansible
 ↓
Docker
 ↓
application
```

### ✅ Bloc acquis si

Tu peux supprimer ton infrastructure et la **reconstruire de manière reproductible**.

----------

# 9. Docker et conteneurisation

### 🎯 Objectif

Comprendre comment emballer et exécuter les applications de manière reproductible.

## Docker

Un **container** est un environnement isolé permettant d'exécuter une application avec ses dépendances.

> 🔵 **À mentionner, définir, ne pas creuser — Podman** : alternative à Docker, sans daemon central et pouvant tourner sans privilèges root ("rootless"), ce qui la rend plus sûre par défaut. Compatible avec la même syntaxe CLI que Docker (`podman build`, `podman run`). Répandu dans les environnements RHEL/CentOS.

Exemple :

```
Application Spring Boot
+ JRE
+ jar + dépendances
+ configuration nécessaire
        ↓
     Docker image
        ↓
     Container
```

### Image

**Image Docker** = modèle utilisé pour créer des containers.

```
Image
 ↓
Container
```

Une image peut créer plusieurs containers.

### Dockerfile

Fichier décrivant comment construire une image.

### Build

```
docker build
```

### Run

```
docker run
```

### Registry

**Docker Registry** = endroit où les images sont stockées.

Exemple :

```
Docker Hub
```

```
Developer
   ↓
docker build
   ↓
image
   ↓
docker push
   ↓
registry
   ↓
server
   ↓
docker pull
```

### Repository d'artefacts — Nexus / Artifactory / GitLab Package Registry

Un Docker Registry ne stocke que des **images Docker**. Or une équipe produit aussi d'autres types d'**artefacts** qu'il faut stocker et versionner quelque part : packages npm privés, dépendances Maven/Java, fichiers `.jar`/`.war`, binaires compilés. C'est le rôle d'un **repository d'artefacts**, un concept plus large que le simple registre d'images.

#### 🎯 Objectif final

Comprendre pourquoi une équipe a besoin d'un endroit centralisé pour stocker packages ET images, distinct du code source (Git) et distinct de la production.

#### 📚 Ce que tu dois connaître

```
Code source (Git)
      ↓
   Build
      ↓
Artefact (jar, npm package, image Docker...)
      ↓
Repository d'artefacts (Nexus / Artifactory / GitLab Package Registry)
      ↓
Déploiement / installation
```

-   **Sonatype Nexus** — repository d'artefacts open-source très répandu, supporte npm, Maven, Docker, etc. en un seul outil.
-   **JFrog Artifactory** — équivalent commercial de Nexus, très utilisé en entreprise.
-   **GitLab Package & Container Registry** — repository d'artefacts intégré directement à GitLab (pas besoin d'un outil séparé si tu utilises déjà GitLab CI).

#### 🧠 Jargon expliqué

**Artefact** = tout résultat de build réutilisable (image, package, binaire) — notion déjà vue au bloc 1, ici on lui donne un **lieu de stockage** dédié.
**Repository d'artefacts** = registre générique (au-delà des seules images Docker) où sont publiés et versionnés les artefacts produits par le pipeline CI/CD.

#### 🛠️ Ce que tu dois savoir faire concrètement

Expliquer où va chaque brique après un build : le code reste dans Git, l'artefact compilé part dans un repository (Nexus/Artifactory/GitLab Package Registry), l'image Docker part dans un registre (Docker Hub/Harbor/GitLab Container Registry).

#### ✅ Critère pour considérer le bloc acquis

Tu sais expliquer la différence entre un dépôt Git (code source), un repository d'artefacts (packages/binaires) et un registre d'images (Docker), et pourquoi on ne mélange jamais ces trois usages.

> 🔵 **À mentionner, définir, ne pas creuser — Harbor** : registre de conteneurs open-source alternatif à Docker Hub, avec scan de vulnérabilités intégré. Pertinent si tu veux héberger ton propre registre d'images plutôt que dépendre de Docker Hub.

### Microservices

Architecture où l'application est divisée en plusieurs services.

```
Frontend
   ↓
API Gateway
   ↓
 ┌───────────┬───────────┐
 ↓           ↓           ↓
Users      Orders      Payments
```

Mais attention :

**Docker ≠ microservices.**

On peut parfaitement utiliser Docker avec un monolithe.

### Isolation des dépendances

Chaque container peut avoir :

```
Node 22
PostgreSQL
Python 3.12
Java 21
```

sans polluer directement le système hôte.

### Docker Compose

**Docker Compose** permet de décrire et lancer **plusieurs containers ensemble** avec un seul fichier, plutôt que d'enchaîner des `docker run` à la main pour chaque service.

#### 🎯 Objectif final

Décrire une petite stack multi-containers (frontend + backend + base de données) et la lancer/arrêter en une seule commande.

#### 📚 Ce que tu dois connaître

Exemple conceptuel de `docker-compose.yml` :

```yaml
services:
  api:
    build: ./api
    ports:
      - "3000:3000"
    depends_on:
      - db
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

Comprendre :

-   `services` — chaque container de la stack
-   `depends_on` — ordre de démarrage entre containers
-   `volumes` — persistance des données au-delà du cycle de vie du container
-   réseau interne automatique entre les services (ils peuvent s'appeler par leur nom, ex. `db` plutôt qu'une IP)

#### 🧠 Jargon expliqué

**Service** (au sens Docker Compose) = un container défini dans le fichier, avec son image/build, ses ports, ses variables d'environnement.
**Volume** = espace de stockage persistant, indépendant du cycle de vie du container (les données survivent à un `docker compose down`).

#### 🛠️ Tu dois savoir faire

Dockeriser :

-   Spring Boot (backend)
-   Angular (frontend servi par Nginx)
-   PostgreSQL
-   (variante : NestJS, Next.js)

et les faire communiquer avec Docker Compose.

#### ✅ Critère pour considérer ce point acquis

Tu peux écrire un `docker-compose.yml` à partir de zéro pour une stack Angular + Spring Boot + PostgreSQL (variante : Next.js/NestJS), sans copier un exemple existant, et expliquer où vont les données si tu fais `docker compose down` avec ou sans `-v`.

### ✅ Bloc acquis si

Tu peux prendre une application que tu n'as jamais exécutée auparavant et la lancer grâce à :

```
docker compose up
```

----------

# 10. Orchestration — Kubernetes

Ici on monte fortement en niveau.

### 🎯 Objectif

Gérer **beaucoup de containers** automatiquement.

Docker seul :

```
1 serveur
   ↓
quelques containers
```

Kubernetes :

```
Cluster
├── Node 1
│   ├── Pod
│   └── Pod
├── Node 2
│   ├── Pod
│   └── Pod
└── Node 3
    ├── Pod
    └── Pod
```

### Kubernetes

Kubernetes est un **orchestrateur de containers**.

Il peut notamment :

-   démarrer des containers
-   les redémarrer
-   répartir la charge
-   faire du scaling
-   effectuer des déploiements progressifs

### Pod

Un **Pod** est la plus petite unité déployable de Kubernetes.

Souvent :

```
Pod
 └── 1 container
```

### Cluster

Ensemble de machines gérées par Kubernetes.

### Deployment

Décrit comment une application doit être exécutée.

Exemple :

```
Je veux 3 instances de mon API.
```

Kubernetes maintient cet état.

### Service

Permet aux applications de communiquer avec les Pods.

Pourquoi ?

Parce que les Pods peuvent être remplacés et leurs IP changent.

### CKA / CKAD / CKS

Ce sont des certifications Kubernetes :

-   **CKA** → administration Kubernetes
-   **CKAD** → développement d'applications Kubernetes
-   **CKS** → sécurité Kubernetes

Tu n'as pas besoin de commencer par les certifications.

> 🔵 **À mentionner, définir, ne pas creuser — Kubernetes managés (Google Kubernetes Engine, Azure Kubernetes Services, Elastic Kubernetes Service) et distributions (OpenShift, Tanzu)** : ce sont des façons d'héberger/gérer un cluster Kubernetes sans tout administrer soi-même (le cloud provider gère le control plane), ou des distributions Kubernetes packagées avec des outils supplémentaires (OpenShift = Red Hat, Tanzu = VMware). Les concepts Kubernetes de ce bloc restent identiques partout — ce sont juste des façons différentes d'obtenir un cluster.
>
> 🔵 **À mentionner, définir, ne pas creuser — Nomad** : orchestrateur de HashiCorp, alternative plus simple à Kubernetes (gère aussi bien des containers que des tâches non conteneurisées), mais avec un écosystème et une communauté bien plus restreints. Pertinent seulement si une organisation l'a déjà adopté.

### 🛠️ Tu dois savoir faire

Déployer :

```
Angular (frontend)
   ↓
Service
   ↓
Spring Boot (backend)
   ↓
Service
   ↓
PostgreSQL
```

dans Kubernetes.

Comprendre :

```
Pod
Deployment
Service
ConfigMap
Secret
Ingress
Namespace
Volume
```

### ✅ Bloc acquis si

Tu comprends pourquoi Kubernetes existe et tu peux déployer/diagnostiquer une application simple dans un cluster.

----------

# 11. Politiques CI/CD

C'est probablement **le cœur pratique du DevOps**.

### 🎯 Objectif

Automatiser :

```
Code
 ↓
Test
 ↓
Build
 ↓
Security checks
 ↓
Deploy
```

à chaque changement.

----------

## CI

**Continuous Integration = intégration continue.**

À chaque changement :

```
git push
   ↓
tests
   ↓
lint
   ↓
build
```

On vérifie que le code reste sain.

----------

## CD

**Continuous Delivery / Continuous Deployment**

Automatisation de la livraison/déploiement.

Exemple :

```
git push
 ↓
CI
 ↓
build Docker
 ↓
push image
 ↓
deploy production
```

### Pipeline

Un **pipeline** est une succession automatique d'étapes.

```
Checkout
   ↓
Install
   ↓
Test
   ↓
Build
   ↓
Docker Build
   ↓
Security Scan
   ↓
Deploy
```

### Jenkins

Serveur d'automatisation CI/CD.

> 🔵 **À mentionner, définir, ne pas creuser** : Jenkins est **auto-hébergé** (contrairement à GitHub Actions/GitLab CI qui sont intégrés à la plateforme), très flexible via ses plugins, mais plus lourd à maintenir (mises à jour, sécurité, plugins à gérer soi-même). Encore très répandu en entreprise — bon à savoir lire un pipeline Jenkins (`Jenkinsfile`) même sans l'utiliser au quotidien.

### GitHub Actions

CI/CD directement intégré à GitHub.

### GitLab CI

CI/CD intégré à GitLab.

### 🧠 Concepts importants

**Runner** = machine qui exécute les tâches du pipeline.

**Artifact** = résultat produit par le pipeline.

**Secret** = information sensible :

```
DATABASE_PASSWORD
AWS_ACCESS_KEY
JWT_SECRET
```

Il ne faut **jamais** les mettre directement dans Git.

### 🛠️ Tu dois savoir faire

Pour un projet Spring Boot :

```
git push
   ↓
GitHub Actions
   ↓
mvn test
   ↓
mvn package
   ↓
jar
   ↓
Docker image
   ↓
Docker Hub
   ↓
serveur
   ↓
deployment
```

> ℹ️ **Variante Next.js/NestJS** (écosystème npm) : le pipeline est identique, mais on remplace `mvn test`/`mvn package` (et l'artefact `.jar`) par `npm install` → `npm test` → `npm run build` (artefact de build Node).

### ✅ Bloc acquis si

Un développeur fait :

```
git push
```

et ton système s'occupe automatiquement du reste.

----------

# 12. Surveillance et observabilité

Une application en production doit être observable.

### 🎯 Objectif

Pouvoir répondre à :

> « Est-ce que mon application fonctionne ? Si non, pourquoi ? »

### Monitoring

**Monitoring** = surveillance de l'état du système.

Exemples :

```
CPU : 80%
RAM : 90%
Disk : 75%
HTTP errors : 12%
```

### Observabilité

Plus large que le monitoring.

Les trois piliers classiques :

```
Logs
Metrics
Traces
```

### Metrics

Valeurs numériques :

```
CPU usage
RAM
requests/sec
latency
error rate
```

### Logs

Événements :

```
ERROR Database connection failed
```

### Traces

Permettent de suivre une requête à travers plusieurs services.

```
Frontend
   ↓
API Gateway
   ↓
User Service
   ↓
Database
```

On peut voir où la requête a pris 2 secondes.

### Prometheus

Système de collecte de **metrics**.

### Grafana

Interface permettant de construire des dashboards et visualiser notamment les métriques.

### Centreon / Zabbix — monitoring traditionnel

Avant Prometheus/Grafana (nés dans le monde cloud-native), le monitoring d'infrastructure se faisait déjà avec des outils comme **Centreon** et **Zabbix** : surveillance d'hôtes/services via des agents installés sur chaque machine, alertes par seuil (CPU, RAM, disque, ping), souvent utilisés en environnement on-premise plus traditionnel.

**Différence avec Prometheus/Grafana** : Centreon/Zabbix sont plus orientés "supervision d'infrastructure classique" (parc de serveurs physiques/VM), alors que Prometheus/Grafana sont nés pour du cloud-native (containers, Kubernetes, métriques applicatives). Les deux mondes coexistent souvent dans une même organisation, notamment dans les environnements publics/administratifs qui gèrent un parc de serveurs historique en parallèle d'une infrastructure plus moderne.

**Critère à retenir** : tu sais reconnaître qu'un système de supervision par agents avec seuils d'alerte (Centreon/Zabbix) répond au même besoin qu'un stack Prometheus/Grafana, avec une approche plus ancienne mais toujours largement utilisée.

### ELK / Loki / autres

Solutions permettant notamment de centraliser les logs.

> 🔴 **À connaître au moins de nom — Splunk** : plateforme de centralisation de logs/observabilité, très puissante mais payante, répandue en grande entreprise. Fonctionne sur le même principe qu'ELK/Loki (centraliser puis chercher dans les logs), avec un langage de requête propriétaire (SPL).
>
> 🔵 **À mentionner, définir, ne pas creuser — DataDog** : plateforme SaaS tout-en-un (logs + metrics + traces + dashboards), alternative payante à la combinaison Prometheus/Grafana/ELK. Très répandu en entreprise privée, mais coûteux — peu probable dans un contexte d'administration publique où les stacks open-source (Prometheus/Grafana) sont généralement préférées.

### Maintenance proactive

Ne pas attendre :

> « Le serveur est mort. »

Mais détecter :

```
Disk 80%
 ↓
alerte
 ↓
action
```

### 🛠️ Tu dois savoir faire

Créer un dashboard montrant :

```
CPU
RAM
Disk
Requests
Errors
Latency
```

et retrouver la cause d'une erreur à partir des logs.

### ✅ Bloc acquis si

On te dit :

> « L'API est lente depuis 10 minutes. »

Tu sais **où regarder et comment trouver la cause**.

----------

# 13. GitOps

GitOps est une évolution logique du CI/CD.

### 🎯 Objectif

Faire de Git la **source de vérité de l'infrastructure et des déploiements**.

### Dépôt Git comme source de vérité

Par exemple :

```
Git
 │
 ├── application
 ├── Kubernetes manifests
 ├── configuration
 └── infrastructure
```

Si tu veux changer la production :

```
Modification Git
      ↓
Pipeline / outil GitOps
      ↓
Production
```

### Argo CD

Outil GitOps très utilisé avec Kubernetes.

Il surveille Git et vérifie que le cluster correspond à ce qui est déclaré dans Git.

### FluxCD

Autre outil GitOps.

### 🧠 Concept important : Desired State

**Desired State = état souhaité.**

Tu dis :

```
replicas: 3
```

Cela signifie :

> Je veux 3 instances.

Kubernetes essaie continuellement de maintenir cet état.

### Drift

**Drift = différence entre l'état déclaré et l'état réel.**

Exemple :

```
Git :
3 replicas

Cluster :
2 replicas
```

Il y a un drift.

GitOps permet de détecter/corriger ce genre de divergence.

### 🛠️ Tu dois savoir faire

Modifier :

```
Git
 ↓
Pull Request
 ↓
review
 ↓
merge
 ↓
Argo CD
 ↓
Kubernetes
```

### ✅ Bloc acquis si

Tu peux expliquer clairement :

> « Je ne me connecte pas manuellement au cluster pour modifier la production ; je modifie l'état désiré dans Git. »

----------

# 14. Service Mesh

C'est le bloc **avancé**. Ne te précipite surtout pas dessus.

### 🎯 Problème

Avec beaucoup de microservices :

```
Service A
 ↓
Service B
 ↓
Service C
 ↓
Service D
```

Chaque service doit gérer :

-   TLS
-   retries
-   timeouts
-   authentification
-   observabilité
-   contrôle du trafic

Cela devient compliqué.

### Service Mesh

Un service mesh fournit une couche spécialisée pour gérer la communication entre services.

Conceptuellement :

```
Application A
     ↓
Proxy
     ↓
Proxy
     ↓
Application B
```

Les applications délèguent certaines fonctions réseau aux proxies.

### Istio

Un des service meshes les plus connus dans l'écosystème Kubernetes.

### Consul

HashiCorp Consul fournit notamment des fonctions liées à la découverte de services et à la connectivité sécurisée.

### Linkerd

Service mesh Kubernetes orienté simplicité et légèreté.

### 🧠 Jargon

**Sidecar** = processus/container associé à l'application pour lui fournir des fonctions supplémentaires.

Exemple :

```
Pod
├── Application
└── Proxy sidecar
```

**mTLS** = mutual TLS.

Contrairement au HTTPS classique où le serveur est authentifié, les deux côtés peuvent s'authentifier mutuellement.

**Service discovery** = mécanisme permettant de trouver dynamiquement où se trouve un service.

### 🛠️ Tu dois savoir faire

Comprendre et mettre en place des concepts comme :

```
Service A
   ↓
Service Mesh
   ↓
Service B
```

avec :

-   TLS
-   retries
-   timeout
-   traffic management
-   observabilité

### ✅ Bloc acquis si

Tu comprends **pourquoi un service mesh est nécessaire dans certaines architectures**, et surtout pourquoi il est inutile de l'introduire dans une petite application.

----------

# Vue globale : ce que tu devrais être capable de faire

Le plus important est de ne pas apprendre ces blocs comme **14 matières indépendantes**.

Ils s'emboîtent :

```
1. SDLC
      ↓
2. Linux
      ↓
3. Scripting
      ↓
4. Git
      ↓
5. Réseau & sécurité
      ↓
6. Cloud
      ↓
7. Bases de données & Data Operations
      ↓
8. Infrastructure as Code
      ↓
9. Docker
      ↓
10. Kubernetes
      ↓
11. CI/CD
      ↓
12. Observabilité
      ↓
13. GitOps
      ↓
14. Service Mesh
```

Et à terme, ton environnement pourrait ressembler à :

```
                 DEVELOPER
                    │
                  Git
                    │
                    ▼
              GitHub / GitLab
                    │
                    ▼
                CI/CD
                    │
          ┌─────────┴─────────┐
          │                   │
       Tests              Docker Build
                              │
                              ▼
                         Registry
                              │
                              ▼
                         Kubernetes
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
           Frontend         Backend        Database
              │               │
              └───────┬───────┘
                      ▼
                Service Mesh
                      │
                      ▼
                 Monitoring
                      │
              ┌───────┴───────┐
              ▼               ▼
            Metrics          Logs
              │               │
              └───────┬───────┘
                      ▼
                   Grafana
```

## ⚠️ Mais il y a un point important pour toi

Avec ton profil actuel de développeur web, **tu n'as pas besoin de donner le même poids aux 14 blocs**.

Je les classerais ainsi :

| Priorité | Blocs | Niveau à viser |
| :--- | :--- | :--- |
| 🔴 Très haute | Linux | Solide |
| 🔴 Très haute | Git | Solide |
| 🔴 Très haute | Réseau | Solide |
| 🔴 Très haute | Docker | Solide |
| 🔴 Très haute | CI/CD | Solide |
| 🔴 Très haute | Scripting | Solide |
| 🟠 Haute | Cloud | Solide |
| 🟠 Haute | Bases de données | Solide |
| 🟠 Haute | IaC | Solide |
| 🟠 Haute | Kubernetes | Solide |
| 🟡 Moyenne | SDLC | Bon |
| 🟡 Moyenne | Observabilité | Bon |
| 🟡 Moyenne | GitOps | Bon |
| 🟢 Avancée | Service Mesh | Notions ➔ pratique ensuite |

*Note : « Microservices » n'apparaît pas comme ligne séparée — c'est une notion abordée dans le bloc Docker (bloc 9), pas un bloc à part entière avec son propre objectif/critère d'acquisition. La concevoir comme architecture avancée reste un sujet à part, à explorer une fois Docker/Kubernetes/CI-CD solides.*

*Note : deux sous-thèmes ont été ajoutés dans des blocs existants (pas de nouveau numéro, pour ne pas casser la progression) : le **Cache applicatif (Redis/Memcache)** dans le bloc 7 (Bases de données), et le **Repository d'artefacts (Nexus/Artifactory/GitLab Package Registry)** dans le bloc 9 (Docker) — priorité 🟠 Haute pour les deux, au même niveau que le reste de leur bloc parent.*

**Service Mesh ne doit clairement pas être ton objectif de départ.** Si tu maîtrises Linux + réseau + Git + Docker + CI/CD + Terraform + Kubernetes, tu as déjà construit une base DevOps très sérieuse.

Et surtout, pour ton profil de développeur **Java/Spring + Angular** (avec **Next.js/NestJS** en pratique secondaire) sur **Linux** (à apprendre, pas un acquis), cette roadmap peut être transformée en un parcours beaucoup plus concret : **un seul projet fil rouge** qui commence avec une application Spring Boot + frontend Angular et finit avec Docker → CI/CD → AWS → Terraform → Kubernetes → monitoring → GitOps. C'est beaucoup plus efficace que d'étudier les 14 blocs séparément.