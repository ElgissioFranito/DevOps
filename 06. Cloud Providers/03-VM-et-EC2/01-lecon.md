# Leçon 3 — Machines virtuelles : EC2

> **Bloc 6 · Cloud Providers** — Leçon 3 sur 8
> 🧭 **Pont depuis la Leçon 2** : ton application a maintenant un **réseau sûr** (le VPC : subnets public/privé, security groups, NAT…). Mais une architecture, c'est surtout des **machines** qui **calculent** : c'est là que tourne ton application. Le cloud appelle ces machines louées des **VM** (vues en Leçon 1) ; chez AWS, une VM s'appelle **EC2**. Tu vas apprendre à la **créer**, vous **connecter en SSH** (souvenir du Bloc 2 !), la **dimensionner** (combien de CPU/RAM ?) et l'**automatiser** (autoscaling, snapshots).

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Expliquer** ce qu'est EC2 (la VM d'AWS) et comment elle se place dans un VPC.
2. **Définir** les notions essentielles : instance, type d'instance, AMI (image), clé SSH (key pair), état running/stopped/terminated.
3. **Créer et connecter** une instance (démarche et commandes, avec alternative simulée sans dépenser).
4. **Dimensionner** une instance au bon besoin (sizing) et expliquer l'autoscaling.
5. **Comprendre** snapshots / sauvegardes et le coût d'une instance qui tourne.

---

## 2. Explication simple

### 2.1 Le « pourquoi » : pourquoi des machines virtuelles dans le cloud ?

Tu l'as vu en Leçon 1 : le cloud découpe de puissants serveurs physiques en **VM** (des « serveurs virtuels ») louées à la demande. **EC2** (Elastic Compute Cloud, « calcul élastique dans le cloud ») est **le nom de ce service chez AWS**. « Elastic » = élastique : tu peux créer, agrandir, réduire et détruire des machines très facilement, comme un élastique qu'on étire et relâche. C'est l'équivalent de « Compute Engine » chez GCP et « Virtual Machines » chez Azure (vu en Leçon 1).

**Pourquoi en a-t-on besoin ?** Parce que ton **application** (le backend Spring Boot, par exemple) doit tourner **quelque part**, sur un système Linux (Bloc 2), accessible depuis le réseau que tu as construit (Leçon 2). Une VM cloud, c'est exactement ça : **un serveur Linux chez AWS**, dans ton VPC, prêt à recevoir ton code.

### 2.2 Le « comment » : les briques d'une instance EC2

Créer une instance EC2, c'est assembler **quatre briques** :

> 💡 **Analogie** : commander un **ordinateur de location** dans un immeuble de bureaux.
> - L'**image (AMI)** = le système d'exploitation préinstallé (ex. Ubuntu) : le disque dur tout prêt.
> - Le **type d'instance** = la puissance du processeur (CPU) et de la mémoire (RAM) : du petit « scooter » à l'énorme « camion ».
> - La **clé SSH (key pair)** = ta clé d'entrée : sans elle, personne ne peut t'ouvrir la porte.
> - Le **security group** = le portier qui décide qui entre par quel port (Leçon 2).

Les briques en détail :

| Brique | C'est quoi ? | Définition simple |
|--------|--------------|-------------------|
| **AMI** (Amazon Machine Image) | Le modèle du système : ex. Ubuntu 24.04 | Un disque dur préinstallé (OS + config) |
| **Type d'instance** | La puissance louée (ex. `t3.medium` = 2 CPU, 4 Go RAM) | Le « gabarit » de la machine : scooters → camions |
| **Key pair** (paire de clés) | Ta clé privée + la clé publique installée sur la machine | Ta clé d'entrée pour le SSH (Bloc 2) |
| **Security group** | Le pare-feu virtuel de la machine | Le portier qui filtre les ports |
| **État de l'instance** | `running` (en marche) / `stopped` (arrêtée) / `terminated` (supprimée) | Allumée / éteinte / jetée |

> 🔑 **Point clé — les coûts par état** : une instance **`running` coûte de l'argent** (payé à l'heure), une instance **`stopped` coûte beaucoup moins** (seulement le disque), une instance **`terminated` ne coûte plus rien** (elle est détruite). Le sujet de la facture sera approfondi à la Leçon 8 (FinOps), mais il faut déjà le garder en tête.

### 2.3 Le « comment » (suite) : se connecter en SSH

Pour administrer une instance (installer du logiciel, déployer, regarder les logs), on s'y connecte en **SSH** — exactement comme au **Bloc 2** (SSH et administration à distance). La différence : au lieu d'une IP de bureau, tu utilises l'**IP publique ou privée** de ton instance cloud.

Rappel de la logique SSH (Bloc 2) : une **clé privée** (chez toi, secrète, `chmod 600`) et une **clé publique** (sur le serveur, dans `authorized_keys`) forment une **paire**. Le cloud te génère la paire (key pair) à la création de l'instance ; tu gardes le fichier `.pem` de la clé privée, AWS installe la clé publique sur la machine.

### 2.4 Le « quand » : le bon dimensionnement et l'autoscaling

**Sizing / dimensionnement** : choisir le **type d'instance** adapté. Une petite API qui reçoit 50 requêtes/minutes n'a pas besoin du même gabarit qu'une plateforme qui en reçoit 50 000.

> 💡 **Analogie** : un studio pour une personne, une maison pour une famille de 6. Prendre trop grand = **payer pour rien** (FinOps, Leçon 8) ; prendre trop petit = **l'application rame ou tombe**.

**Autoscaling** (auto = automatique, scaling = mise à l'échelle) : le cloud peut **ajouter ou retirer des instances automatiquement** selon la charge réelle (ex. si le CPU dépasse 80 %, AWS en démarre une 2e ; si c'est calme, il en retire une). On le voit en théorie ici ; la mise en œuvre concrète viendra avec le Bloc 8 (IaC) et le Bloc 12 (observabilité).

### 2.5 Le « quand » : snapshots / sauvegardes

Une instance peut **tomber** ou être **supprimée par erreur**. Pour survivre, on fait des **snapshots** : des **photos de l'état du système** (disque + config) à un instant T, qu'on peut restaurer. C'est le **backup de la VM** (les stratégies de sauvegarde seront approfondies au Bloc 7 pour les bases de données).

### 2.6 Où placer l'instance dans le réseau ?

Souviens-toi de la Leçon 2 : tu as des sous-réseaux **publics** et **privés** dans ton VPC.
- Une instance qui doit être **jointe depuis Internet** (ex. ton backend s'il n'y a pas de load balancer) → sous-réseau **public** + security group qui n'ouvre que le port nécessaire (ex. 443/80).
- Une instance **interne** (ex. base de données, services cachés) → sous-réseau **privé**, jamais d'IP publique (la NAT Gateway sert à sortir discrètement).

> 🔑 **Règle d'or répétée du bloc** : moins une machine est exposée, mieux c'est. On ne met en public que ce dont Internet **a besoin**.

---

## 📖 Vocabulaire / Abréviations

> Définitions d'une ligne pour ne jamais être perdu(e).

- **EC2** (Elastic Compute Cloud) : le service de VM d'AWS.
- **Instance** : une VM en cours d'exécution (une unité louée).
- **Type d'instance** : le gabarit de puissance CPU/RAM (ex. `t3.micro` = 1 CPU / 1 Go RAM).
- **AMI** (Amazon Machine Image) : le modèle de système préinstallé (ex. Ubuntu).
- **Key pair / paire de clés** : la clé privée (chez toi) + la clé publique (sur la machine) pour le SSH (Bloc 2).
- **Security group** : pare-feu virtuel par ressource (Leçons 2 et Bloc 5).
- **SSH** : protocole de connexion sécurisée à distance (Bloc 2) — le moyen d'administrer une instance.
- **`chmod 600`** : permission Unix « seul le propriétaire lit/écrit » (Bloc 2) — obligatoire pour une clé privée.
- **Running / stopped / terminated** : en marche (payant) / arrêtée (peu payant) / supprimée (ne coûte plus).
- **Sizing / dimensionnement** : choisir la bonne taille de machine pour le besoin.
- **Autoscaling** : ajouter/retirer des instances automatiquement selon la charge.
- **Snapshot** : photo de l'état d'une VM (disque + config), à restaurer en cas de pépin.
- **Région / zone de disponibilité (AZ)** : emplacement géographique des data centers AWS dans le monde.

---

## 3. Exemples concrets

> ⚠️ **Réalité pratique** : lancer une vraie instance EC2 crée un **coût (même minime)** et demande un compte + des clés (Leçon 6). On présente d'abord les **commandes AWS réelles** (à exécuter avec ton compte), puis une **alternative locale simulée** sans dépenser.

### 3.1 Commandes AWS réelles (avec compte configuré)

```bash
# 1. Trouver une AMI Ubuntu récente (le modèle de système).
# --owners 099720109477 = ID officiel d'Ubuntu chez AWS ; --filters = filtre sur le nom ;
# --query + --output = ne garder que l'ID et le nom (sortie lisible).
aws ec2 describe-images \
  --owners 099720109477 \
  --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-*-amd64-server-*" \
  --query "sort_by(Images, &CreationDate)[-1].{ID:ImageId,Nom:Name}" \
  --output table

# 2. Créer une instance (petit gabarit).
# --image-id = l'AMI trouvée ; --instance-type t3.micro = petit gabarit (1 CPU/1 Go) ;
# --key-name = ta paire de clés ; --subnet-id = ton subnet (Leçon 2) ;
# --associate-public-ip-address = donner une IP publique (seulement si subnet public !).
aws ec2 run-instances \
  --image-id ami-xxxxxxxx \
  --instance-type t3.micro \
  --key-name ma-cle-ssh \
  --subnet-id subnet-xxxxxxxx \
  --associate-public-ip-address

# 3. Vérifier l'état (running ?).
aws ec2 describe-instances --query "Reservations[].Instances[].{ID:InstanceId,Etat:State.Name,IP:PublicIpAddress}"

# 4. Se connecter en SSH avec la clé privée (rappel Bloc 2).
chmod 600 ma-cle-ssh.pem
ssh -i ma-cle-ssh.pem ubuntu@<IP-PUBLIQUE>
```

### 3.2 Alternative locale simulée (sans dépenser)

Pour **apprendre la logique sans payer**, voici un script qui simule la « fiche d'une instance » :

```bash
# simuler-instance.sh — réfléchir à une instance avant de la créer.
echo "🧠 Simulation de création d'instance EC2"
echo "Ami de système      : ubuntu-24.04 (AMI)"
echo "Type d'instance     : t3.micro (1 CPU / 1 Go RAM)"
echo "État                : running 🟢 (coûte de l'argent !)"
echo "Emplacement réseau  : subnet public (IP publique) OU privé (IP privée)"
echo "Clé SSH             : ma-cle-ssh.pem (à conserver en chmod 600)"
echo ""
echo "🧾 Question de dimensionnement :"
echo "   50 requêtes/min → t3.micro suffit ; 50 000 requêtes/min → plusieurs instances + load balancer."
```

Puis exécute-le : `bash simuler-instance.sh` (Bash vu au Bloc 3). Tu peux aussi tester un vrai `ssh` avec une machine **locale** (VM VirtualBox de ton parcours Linux) pour revoir les commandes serveur.

---

## 4. Bonnes pratiques modernes (2025-2026)

- **Commencer petit** (`t3.micro`), puis dimensionner après observation (observabilité = Bloc 12).
- **`chmod 600` sur la clé privée** et **jamais dans Git** (réflexe « secrets » du Bloc 5).
- **Instances éphémères + données ailleurs** : une VM peut mourir ; les données de valeur vont dans un stockage séparé (S3, Leçon 4) ou une base managée (Leçon 5).
- **Autoscaling** pour suivre la charge (à coupler avec une vraie observabilité, Bloc 12).
- **Snapshots réguliers** + un test de restauration au moins une fois.
- **Arrêter/supprimer ce qui ne sert plus** : c'est la première économie possible (préparation FinOps, Leçon 8).

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi c'est dangereux/inefficace | ✅ Version correcte |
|----------------|-------------------------------------|---------------------|
| Laisser tourner des instances `t3.2xlarge` (grosses) « au cas où » | Facture qui grimpe sans besoin | Dimensionner au besoin réel (sizing) + surveiller la charge |
| Oublier `chmod 600` sur la clé privée | SSH refuse (permissions trop ouvertes) ou clé volable | `chmod 600 ma-cle-ssh.pem` immédiatement |
| Committer `ma-cle-ssh.pem` dans Git | Vol de ta clé d'accès au serveur | Ne jamais versionner une clé privée |
| Ouvrir le security group sur le port SSH à tout (`0.0.0.0/0`) | N'importe qui peut tenter des connexions | Restreindre SSH à tes IP (ou passer par un bastion/VPN, Leçon 2) |
| Créer une grosse instance sans besoin | Payer pour du vide (revenu en Leçon 8 / FinOps) | Commencer petit, autoscaling si besoin |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`**, la correction dans **`03-correction.md`**.

**Énoncé court** : rédige, dans `notes-exercice-03.md`, la **« fiche de décision »** d'une instance pour ton backend Spring Boot (50 requêtes/min) : AMI, type d'instance, subnet (public ou privé ?), security group (ports ?), clé SSH, snapshot. Crée le script `simuler-instance.sh`, exécute-le, puis réponds à 2 questions : coût d'une instance running vs stopped ; que faire si la demande passe de 50 à 50 000 requêtes/min ?

---

## 7. Correction détaillée de l'exercice

> La correction complète est dans **`03-correction.md`**. On y compare ta fiche à la réponse type et on justifie chaque choix (subtilité : pour ta demande, le backend est souvent derrière un load balancer → subnet privé !).

---

## 8. Checklist de validation

- [ ] J'explique EC2 (la VM d'AWS) et je le place dans un VPC.
- [ ] Je définis instance, type d'instance, AMI, key pair, security group.
- [ ] Je distingue running / stopped / terminated et leurs coûts.
- [ ] Je crée et connecte une instance (démarche + commandes AWS / alternative simulée).
- [ ] Je dimensionne une instance (sizing) et j'explique l'autoscaling.
- [ ] Je fais un snapshot et j'explique pourquoi c'est indispensable.

---

🧭 **Pont vers la suite** — Ton application tourne maintenant sur une machine, mais **où sont les fichiers** (images, PDF, sauvegardes) et **où stocker les données durables** ? Pas sur la VM (elle peut mourir) : dans un stockage **séparé et très robuste**, le stockage d'objets **S3** — la Leçon 4.

---

*Prochaine étape :* Leçon 4 — **Stockage objet (S3)** dans `04-Stockage-objet-S3/`.
