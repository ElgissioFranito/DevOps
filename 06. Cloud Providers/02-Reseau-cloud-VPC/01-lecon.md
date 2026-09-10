# Leçon 2 — Réseau cloud : VPC et cloud networking

> **Bloc 6 · Cloud Providers** — Leçon 2 sur 8
> 🧭 **Pont depuis la Leçon 1** : tu sais qu'un cloud loue des briques virtuelles (VM, stockage…) chez un provider (AWS en profondeur). Mais **où** vivent ces briques, et **comment communiquent-elles** en sécurité ? C'est le réseau. Tu as déjà les **clés** grâce au Bloc 5 : IP, subnet, pare-feu, load balancer, DNS… Le cloud les **réutilise telles quelles**, à grande échelle, dans ce qu'on appelle le **VPC**. Cette leçon est l'une des plus importantes du bloc : sans elle, aucune architecture cloud solide n'est possible.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Définir** le VPC (Virtual Private Cloud) et son rôle d'« immeuble privé » dans le cloud.
2. **Expliquer** le rôle de chaque composant réseau : subnet, route table, internet gateway, NAT gateway, security group, DNS, load balancer, VPN.
3. **Distinguer** sous-réseau public et privé, IP publique et IP privée, en sachant **pourquoi** isoler.
4. **Dessiner** l'architecture réseau d'une application cloud (Load Balancer → App privée → DB privée).
5. **Lire et interpréter** les commandes AWS de base sur le VPC, avec une alternative locale simulable.

---

## 2. Explication simple

### 2.1 Le « pourquoi » : isoler et contrôler le réseau

Dans le Bloc 5, tu as vu qu'une application a un **réseau** : des IP, des sous-réseaux (subnets), un pare-feu, un DNS. Dans le cloud, tu n'as pas un « câble » physique : le cloud met à ta disposition un **réseau privé virtuel**, isolé du reste, dans lequel tu décides **quoi est public** et **quoi est privé**.

Pourquoi cette isolation est-elle vitale ? Parce que la **base de données ne doit jamais être accessible depuis Internet** — seule l'application doit l'atteindre. C'est le même principe de « moindre exposition » que le pare-feu du Bloc 5, appliqué au cloud.

### 2.2 Le « comment » : le VPC et ses composants

> 💡 **Analogie** : un **VPC**, c'est ton **immeuble privé** dans la grande ville qu'est le cloud. À l'intérieur, tu décides quels appartements donnent sur la rue (sous-réseau **public**) et lesquels sont dans la cour intérieure (sous-réseau **privé**). Un **Security Group**, c'est le **portier** de chaque appartement qui décide qui entre.

Le VPC (Virtual Private Cloud, « réseau privé virtuel ») regroupe tous les composants réseau d'un projet :

```
VPC (ton immeuble privé)
├── Public Subnet   ← exposé à Internet (load balancer, machines publiques)
├── Private Subnet  ← caché (application, base de données)
├── Route Table     ← les "panneaux" qui indiquent où envoyer le trafic
├── Internet Gateway (IGW)  ← la porte vers Internet
├── NAT Gateway     ← la sortie "discrète" du sous-réseau privé
├── Security Groups ← les portiers qui filtrent le trafic
└── VPN             ← un tunnel sécurisé vers un autre réseau (ex. bureau)
```

#### Panorama des briques

| Composant | C'est quoi ? | Analogie |
|-----------|--------------|----------|
| **VPC** | Réseau privé virtuel global d'un projet cloud | L'immeuble privé |
| **Subnet** | Subdivision du VPC en zones (public/privé) — vu au Bloc 5 | Les étages/quartiers de l'immeuble |
| **Route Table** | Table de règles disant où envoyer les paquets (Bloc 5) | Les panneaux de signalisation |
| **Internet Gateway (IGW)** | Le passage qui relie le VPC à Internet | La porte d'entrée principale |
| **NAT Gateway** | Permet au privé de sortir vers Internet **sans être vu** | Une sortie de service discrète |
| **Security Group** | Pare-feu virtuel appliqué aux ressources (Bloc 5) | Le portier |
| **DNS** | L'annuaire nom → IP (Bloc 5) | L'annuaire de l'immeuble |
| **Load Balancer** | Répartit le trafic entre plusieurs serveurs (Bloc 5) | La réception qui répartit les clients |
| **VPN** | Tunnel chiffré entre deux réseaux | Un passage souterrain privé |

> 🔎 **VPN — de quoi parle-t-on vraiment ?** Un **VPN** (Virtual Private Network, « réseau privé virtuel ») relie **deux réseaux séparés par Internet** comme s'ils n'en formaient **qu'un seul réseau privé**, en **chiffrant** tout ce qui circule entre eux. Sans VPN, ton ordinateur à la maison et le serveur de ton entreprise sont comme **deux immeubles dans deux villes différentes** : tu ne peux pas entrer dans l'immeuble de l'entreprise si le portier ne te connaît pas. Avec un VPN, tu empruntes un **tunnel secret** entre les deux immeubles : en entrant dans le tunnel, tu sors **à l'intérieur** du réseau de l'entreprise, comme si tu y étais physiquement branché(e), et personne sur Internet ne voit ce que tu transportes (tout est chiffré). **Pourquoi c'est utile au cloud ?** Pour **administrer** (en SSH, Bloc 2) tes machines privées **sans** avoir à les exposer sur Internet, ou pour relier le **bureau de l'entreprise** au **VPC** cloud : les deux réseaux se voient comme s'ils étaient adjacents, en sécurité. (Un **bastion**, lui, est une petite machine publique **faite uniquement pour servir de porte d'entrée** : tu te connectes d'abord au bastion, puis de là aux machines privées — c'est une alternative au VPN, avec la même idée : ne jamais exposer directement les machines internes.)

> 🔑 **Le principe central à retenir** : le sous-réseau **public** contient ce qu'Internet doit joindre (le **load balancer**) ; le sous-réseau **privé** contient ce qu'on **cache** (application, base de données). La base en particulier n'a **aucune** route vers Internet : seul l'application peut la joindre.

### 2.3 Le « comment » (suite) : sous-réseau public vs privé

Pour comprendre **pourquoi** tel composant est public et tel autre privé, regarde le trajet d'une requête vers le site :

```
Internet
   ↓ (DNS : le nom exemple.com devient l'IP du load balancer)
Internet Gateway (IGW) → la porte du VPC s'ouvre
   ↓
Load Balancer (sous-réseau PUBLIC) → répartit entre plusieurs serveurs
   ↓
Application (sous-réseau PRIVÉ) ← sécurité : pas accessible depuis Internet
   ↓
Base de données (sous-réseau PRIVÉ) ← la plus cachée
```

- Le **load balancer** est **public** : c'est lui qu'Internet voit et joint.
- L'**application** est **privée** : seuls le load balancer et les autres services internes peuvent la joindre.
- La **base de données** est **privée**, voire dans un sous-réseau encore plus isolé : **personne d'Internet ne doit l'atteindre**.

> 💡 Pourquoi cette cascade ? Si un pirate atteignait la base directement, il volerait **toutes les données**. En la cachant derrière plusieurs couches (load balancer → application), tu réduis les zones d'attaque : c'est la **défense en profondeur** et le **moindre exposition** du Bloc 5.

**IP publique vs IP privée** : une **IP publique** est joignable depuis tout Internet ; une **IP privée** (vue au Bloc 5, ex. `10.0.0.x`, `192.168.x.x`) n'est valable que **dans** le réseau (le VPC). Le load balancer a une IP publique (via l'IGW) ; l'application et la base n'ont que des IP privées.

> 🔎 **NAT Gateway — la sortie discrète** : parfois l'application privée doit **quand même** aller sur Internet (télécharger des mises à jour) sans être joignable. La **NAT Gateway** lui permet de **sortir** vers Internet tout en restant invisible de l'extérieur. C'est comme un portier qui sort pour toi et ramène le résultat : personne ne sait que c'est toi.

### 2.4 Le « quand » : quand construit-on un VPC ?

À chaque fois qu'on veut porter une application en production dans le cloud de façon sérieuse et sécurisée. Ce n'est pas un luxe : la roadmap exige de « dessiner et expliquer le réseau d'une application cloud, qui est public, qui est privé et pourquoi ». On le construira d'abord **sur le papier** (ici), puis en **IaC** (Bloc 8, Terraform), qui automatisera cette vision en code.

---

## 📖 Vocabulaire / Abréviations

> Définitions d'une ligne pour ne jamais être perdu(e).

- **VPC** (Virtual Private Cloud) : le réseau privé virtuel global d'un projet cloud.
- **Subnet** : subdivision du VPC (public ou privé) — repris du Bloc 5.
- **Route Table** : table de règles qui indiquent où envoyer les paquets (Bloc 5).
- **Internet Gateway (IGW)** : la porte qui relie le VPC à Internet.
- **NAT Gateway** : sortie discrète vers Internet pour les ressources privées (elles sortent sans être joignables).
- **Security Group** : pare-feu virtuel appliqué aux ressources cloud (Bloc 5).
- **DNS** : l'annuaire qui transforme un nom (exemple.com) en IP (Bloc 5).
- **Load Balancer** : répartiteur de charge entre plusieurs serveurs (Bloc 5).
- **VPN** (Virtual Private Network) : tunnel chiffré qui relie deux réseaux séparés par Internet comme s'ils étaient un seul réseau privé (ex. relier ton bureau au VPC).
- **Bastion** : petite machine publique servant uniquement de porte d'entrée pour rejoindre les machines privées (alternative au VPN).
- **IP publique** : adresse joignable depuis tout Internet.
- **IP privée** : adresse valable uniquement dans le réseau privé (VPC).
- **Défense en profondeur** : cumuler plusieurs couches de sécurité avant la donnée sensible.

---

## 3. Exemples concrets

> ⚠️ **Réalité pratique** : créer un VPC réel demande un **compte AWS** et des **clés** (créées à la Leçon 6). On présente ici les **commandes AWS réelles** (à adapter quand tu auras tes clés) et une **alternative locale simulable**. Commandes commentées ligne par ligne.

### 3.1 Commandes AWS réelles (une fois les clés disponibles)

```bash
# Crée un VPC avec le bloc d'adresses 10.0.0.0/16 (65 536 IP disponibles).
# AWS renvoie un VPC-ID (ex. vpc-0abc123).
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# Crée un sous-réseau dans ce VPC, plage 10.0.1.0/24 (256 adresses).
# --vpc-id = identifiant du VPC ; --cidr-block = la plage de ce subnet.
aws ec2 create-subnet --vpc-id vpc-0abc123 --cidr-block 10.0.1.0/24

# Liste les VPC pour vérifier que le tien existe.
aws ec2 describe-vpcs

# Liste les sous-réseaux.
aws ec2 describe-subnets
```

> 💡 **`--cidr-block 10.0.0.0/16`** : le `/16` est un masque CIDR (Bloc 5). Plus le nombre après `/` est petit, plus le réseau est grand : `/16` donne beaucoup d'adresses, `/24` en donne 256 (typique d'un subnet).

### 3.2 Alternative locale simulable (sans compte AWS)

Pour **visualiser** un VPC sans dépenser, un petit script Bash qui affiche l'architecture publique/privée :

```bash
# Ce script n'appelle pas AWS : il aide à mémoriser qui est public et qui est privé.
echo "🏢 Mon VPC 10.0.0.0/16"
echo "  ├── [PUBLIC] load balancer  10.0.1.10  → joignable depuis Internet"
echo "  ├── [PUBLIC] internet gateway (IGW)     → la porte vers Internet"
echo "  └── [PRIVÉ ] application    10.0.2.10  → cachée, IP privée"
echo "      └── [PRIVÉ ] base de données 10.0.3.10 → totalement cachée"

# Règle d'or : on n'expose jamais la base sur Internet.
echo "Règle d'or : jamais d'IP publique sur la base de données."
```

### 3.3 Réflexe face à une exposition

```bash
# Face à "le backend est accessible depuis Internet", vérifie :
# 1) le subnet (est-il public ?) ; 2) le security group (autorise-t-il le trafic depuis 0.0.0.0/0 ?).
# Une base correcte = subnet privé + security group restreint à l'application.
```

---

## 4. Bonnes pratiques modernes (2025-2026)

- **Toujours séparer public et privé** : load balancer public ; application et base privées.
- **Répartir sur plusieurs zones de disponibilité (AZ)** pour la redondance (détaillé au Bloc 8).
- **Moindre exposé pour la base** : aucun accès Internet, security group restreint à l'application.
- **Security Group en « défaut-deny »** : n'autoriser que le strict nécessaire (comme le pare-feu du Bloc 5).
- **Tout réseau en code (IaC)** : en production, décrire le VPC dans Terraform (Bloc 8), pour la reproductibilité.
- **Nommer les ressources** (Tags) pour les retrouver et suivre les coûts (Leçon 8).

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi c'est dangereux/inefficace | ✅ Version correcte |
|----------------|-------------------------------------|---------------------|
| Base dans un sous-réseau public | Accessible et attaquable depuis Internet | Sous-réseau **privé**, aucun accès Internet |
| Security Group de la base ouvert à `0.0.0.0/0` | N'importe qui peut se connecter | Restreindre à l'IP / security group de l'application |
| Multiplier les VPC sans réfléchir | Complexité et coût inutiles | **Un** VPC avec subnets public/privé bien séparés |
| Oublier le NAT Gateway pour un service privé qui sort | Le service privé ne peut plus faire ses mises à jour | Ajouter un NAT Gateway (sortie discrète) |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`**, la correction dans **`03-correction.md`**.

**Énoncé court** : dessine (sur papier ou dans `notes-exercice-02.md`) l'architecture réseau d'une application cloud (frontend Angular + backend Spring Boot + base PostgreSQL) en indiquant **qui est public, qui est privé, et pourquoi**, avec tous les composants du VPC (IGW, load balancer, subnets, security groups, NAT). Écris aussi le petit script Bash d'affichage du VPC et réponds à : « que se passe-t-il si la base est dans un subnet public ? ».

---

## 7. Correction détaillée de l'exercice

> La correction complète est dans **`03-correction.md`**. On compare ta réponse au schéma de référence, on justifie chaque choix public/privé, et on valide la règle d'or (base toujours privée).

---

## 8. Checklist de validation

- [ ] Je définis le VPC et son rôle d'« immeuble privé » dans le cloud.
- [ ] J'explique subnet, route table, IGW, NAT Gateway, Security Group, DNS, Load Balancer, VPN.
- [ ] Je distingue sous-réseau public et privé, IP publique et privée.
- [ ] Je dessine l'architecture réseau (LB → App → DB) et explique qui est public/privé et pourquoi.
- [ ] Je sais pourquoi la base de données ne doit jamais être accessible depuis Internet.
- [ ] Je lis les commandes AWS de base (VPC/subnet) et la règle d'or de sécurité.

---

🧭 **Pont vers la suite** — Ton architecture a maintenant un **réseau sûr** (le VPC). Mais où tourne réellement ton **application** sur ce réseau ? Il faut une machine : c'est la **VM / EC2**, la Leçon 3. Tu vas la créer, t'y connecter en SSH (souvenir du Bloc 2 !), et comprendre le dimensionnement et l'autoscaling.

---

*Prochaine étape :* Leçon 3 — **Machines virtuelles (EC2)** dans `03-VM-et-EC2/`.
