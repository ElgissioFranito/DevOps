# Leçon 5 — Bases de données managées : RDS

> **Bloc 6 · Cloud Providers** — Leçon 5 sur 8
> 🧭 **Pont depuis la Leçon 4 (S3)** : les **fichiers** (avatars, PDF, backups) ont trouvé leur maison durable dans **S3**. Mais l'application a aussi des **données structurées** — les utilisateurs, les commandes, les compteurs — qui vivent dans une **base de données** (tu connais déjà PostgreSQL depuis le Bloc 2, et la roadmap t'y formera plus en détail au Bloc 7). Dans le cloud, on n'installe pas soi-même PostgreSQL sur une VM sans réfléchir : on commande au fournisseur une **base de données « managée »**, c'est-à-dire **gérée par le cloud** : c'est le service **RDS**. Cette leçon explique pourquoi c'est le choix moderne et sûr.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Expliquer** ce qu'est une base de données « managée » et en quoi elle diffère d'une base self-hébergée sur une VM.
2. **Définir** RDS et ses concepts : moteur (PostgreSQL/MySQL), instance, endpoint, sauvegardes automatiques, réplica.
3. **Comprendre** pourquoi une base de données ne doit **jamais** être accessible depuis Internet (rappel Leçon 2).
4. **Créer et connecter** une base RDS (démarche + commandes, avec alternative locale sans dépenser).
5. **Connaître** le minimum de haute disponibilité : backups automatiques, point-in-time recovery, réplica en lecture.

---

## 2. Explication simple

### 2.1 Le « pourquoi » : pourquoi une base « managée » ?

Installer soi-même sa base, c'est assumer beaucoup de tâches qui reviennent en permanence :

```
Base auto-hébergée (moi-même) :
  installer PostgreSQL          → je le fais
  surveiller le disque          → je le fais
  faire des sauvegardes         → je les oublie souvent...
  appliquer les mises à jour    → je les repousse...
  surveiller la performance     → je découvre les problèmes trop tard
```

Le cloud peut **prendre tout cela en charge** : c'est le principe de la **gestion** (être « managé »). Le fournisseur gère l'installation, la maintenance, les **sauvegardes automatiques**, les **mises à jour de sécurité**, et remplace un disque qui casse.

> 💡 **Analogie** : une base auto-hébergée, c'est **cuisiner toi-même avec tes propres casseroles** (tu gères tout). Une base **managée**, c'est aller dans un **restaurant** : quelqu'un d'autre gère la cuisine, les stocks et la sécurité — tu commandes et tu reçois.

**Le vrai intérêt DevOps** : tu passes ton temps sur ton **application et son architecture**, pas à administrer une base. Et le fournisseur garantit **sauvegarde, restauration et haute disponibilité** bien mieux qu'un débutant seul ne le ferait.

### 2.2 Le « comment » : RDS et ses composants

**RDS** = **Relational Database Service** (service de bases de données relationnelles) : le service d'AWS pour les bases **relationnelles** (PostgreSQL, MySQL, MariaDB). (Une base relationnelle = des données dans des tables liées entre elles, interrogées en SQL — le Bloc 7 t'y formera.) Remarque : RDS fournit l'**instance de base** ; l'administration SQL elle-même s'approfondira au Bloc 7.

Les concepts essentiels :

| Terme | C'est quoi ? (1 ligne) |
|-------|------------------------|
| **Moteur (engine)** | Le type de base : PostgreSQL, MySQL, MariaDB… |
| **Instance RDS** | Une base de données louée, avec sa propre adresse réseau |
| **Endpoint** | L'adresse de connexion (le « nom d'hôte » : `ma-base.xxx.eu-west-3.rds.amazonaws.com`) |
| **Port** | Le numéro de guichet du service (PostgreSQL = **5432**, MySQL = **3306**) — vu au Bloc 5 |
| **Backup automatique** | Sauvegarde faite par AWS sans que tu aies à y penser |
| **Réplica (replica)** | Une **copie en lecture** de la base (pour répartir les lectures / survivre à une panne) |
| **Point-in-time recovery** | Pouvoir restaurer la base à un **instant précis** (avant une erreur) |

> 🔑 **Le point crucial — le réseau** : ta base RDS doit vivre dans un **subnet privé** (Leçon 2) et n'être joignable **que depuis ton application**. **Personne d'Internet ne doit s'y connecter.** C'est la même règle d'or que pour toutes les données sensibles.

### 2.3 Le « comment » (suite) : architecture typique avec RDS

Reprenons le schéma de la Leçon 2 en ajoutant la base managée :

```
Internet
   ↓
Load Balancer (public)
   ↓
Application Spring Boot (privé) → elle SEULE parle à la base
   ↓
RDS PostgreSQL (privé, jamais public) ← backups automatiques par AWS
```

Pourquoi cette architecture ? Si la base était **joignable depuis Internet**, n'importe qui pourrait tenter des connexions (attaques par force brute, exploitation de failles). En la plaçant **en privé**, la seule porte d'entrée reste **l'application** — qui possède les identifiants de connexion (qui se gèrent avec IAM, la Leçon 6).

### 2.4 Le « quand » : haute disponibilité et sauvegardes

Une base qui tombe = application en panne. Le cloud offre des garde-fous :

- **Backups automatiques** (actifs par défaut) : AWS fige la base périodiquement.
- **Point-in-time recovery** : restauration à un instant précis (si tu supprimes des lignes par erreur à 14h03, tu restaures à 14h02).
- **Réplica en lecture** : une copie qui peut prendre une partie des lectures, et qui devient le nouveau serveur si le principal tombe (**failover** — bascule automatique). On y reviendra en détail au Bloc 7.

> ⚠️ **Ne pas confondre** : RDS gère **la base** (moteur, sauvegardes, mises à jour) ; il ne gère pas **tes requêtes ni ta modélisation**. Le design des tables, les index, les migrations SQL — c'est ton travail (Bloc 7 !). La frontière : le **conteneur** (le cloud) vs le **contenu** (ton schéma de données).

---

## 📖 Vocabulaire / Abréviations

> Définitions d'une ligne pour ne jamais être perdu(e).

- **Base de données relationnelle** : des données organisées en tables liées, interrogées en SQL (PostgreSQL, MySQL…).
- **SQL** (Structured Query Language) : le langage pour interroger une base relationnelle (Bloc 7).
- **RDS** (Relational Database Service) : le service d'AWS qui loue des bases relationnelles « managées ».
- **Base « managée »** : base dont le fournisseur s'occupe (installation, sauvegardes, mises à jour, disque, haute dispo).
- **Moteur (engine)** : le type de base (PostgreSQL, MySQL, MariaDB…).
- **Instance RDS** : une unité de base louée, avec son endpoint.
- **Endpoint** : l'adresse réseau de connexion (ex. `ma-base.xxx.eu-west-3.rds.amazonaws.com`).
- **Port** : le guichet du service (5432 = PostgreSQL, 3306 = MySQL) — vu au Bloc 5.
- **Backup automatique** : sauvegarde faite automatiquement par le fournisseur.
- **Point-in-time recovery** : restauration de la base à un **instant précis** dans le passé.
- **Réplica** : une copie de la base (souvent en lecture) pour répartir / secourir.
- **Failover** : bascule automatique vers une base de secours quand la principale tombe.
- **Identifiants (credentials)** : le nom d'utilisateur + mot de passe de connexion à la base (à garder dans un coffre).

---

## 3. Exemples concrets

> ⚠️ **Réalité pratique** : créer une vraie base RDS coûte un peu (même minimale) et demande un compte + clés (Leçon 6). On montre les **commandes AWS réelles** à appliquer plus tard, et une **alternative locale gratuite** (PostgreSQL installé chez toi, Bloc 2) pour pratiquer la logique de connexion maintenant.

### 3.1 Commandes AWS réelles (avec compte configuré)

```bash
# 1. Créer une instance RDS PostgreSQL.
# --engine = moteur (postgres) ; --db-instance-class = gabarit (db.t3.micro = petit) ;
# --allocated-storage = espace disque en Go ; --master-username / --master-user-password = identifiants admin.
aws rds create-db-instance \
  --db-instance-identifier ma-base \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --allocated-storage 20 \
  --master-username admin \
  --master-user-password "MotDePasseTemporaire!"
```

```bash
# 2. Se connecter avec psql (le client PostgreSQL — installé dans ton environnement Linux).
# -h = host/endpoint ; -p = port ; -U = utilisateur ; -d = base.
psql -h MON_ENDPOINT.rds.amazonaws.com -p 5432 -U admin -d postgres
# (on te demandera le mot de passe créé avec --master-user-password)
```

> 📌 **Important** : pour te connecter à une RDS, ton **ordinateur** doit être dans un réseau autorisé par le security group / la politique d'accès du subnet (Leçon 2). En production, l'application se connecte **depuis le subnet privé** ; ton poste ne s'y connecte que via un **bastion** ou un **VPN** (présentés en Leçon 2 ; le VPN WireGuard/OpenVPN se construit au **Bloc 5, Leçon 5**) — jamais en exposant la base sur Internet.

### 3.2 Alternative locale gratuite : PostgreSQL sur ta machine

Reprends ton environnement de test Linux (Bloc 2) et entraîne-toi à la logique « base + utilisateur + connexion » en local :

```bash
# 1. Installer PostgreSQL (déjà vu au Bloc 2) — sur Ubuntu/Debian.
sudo apt update
sudo apt install -y postgresql

# 2. Démarrer le service et vérifier que le port 5432 est bien ouvert.
sudo systemctl start postgresql
ss -tulpn | grep 5432        # le port 5432 = le guichet de PostgreSQL

# 3. Se connecter en local avec psql (-h localhost = sur cette machine).
sudo -u postgres psql -h localhost -p 5432
# (dans psql : \l pour lister les bases, \q pour quitter)
```

> 🔎 **Le +** : les notions `endpoint`/`port`/`utilisateur` pratiquées en local sont **exactement les mêmes** que pour RDS — seule l'adresse change. Ce que tu sais faire en local, tu sauras le faire contre RDS.

### 3.3 Réflexe anti-panne (à garder en tête)

```bash
# Face à "l'application n'arrive pas à joindre la base", vérifie dans l'ordre :
# 1) l'endpoint est-il correct ?  2) le port (5432) ?
# 3) le subnet / security group autorise-t-il l'application ?
# (On vérifie le réseau AVANT le mot de passe : c'est l'erreur la plus fréquente.)
```

---

## 4. Bonnes pratiques modernes (2025-2026)

- **Base toujours en subnet privé** + security group limité à l'application — jamais sur Internet.
- **Backups automatiques activés** + test du **point-in-time recovery** au moins une fois (un backup non testé n'est pas fiable !).
- **Réplica en lecture** pour répartir les lectures et prévoir le failover (approfondi au Bloc 7).
- **Identifiants dans un coffre** (Secret Manager d'AWS, ou Vault, vu au Bloc 5) — **jamais en clair dans le code** ni dans Git.
- **Commencer petit** (ex. `db.t3.micro`), observer les performances (Bloc 12) avant de grossir.
- **Penser aux coûts** : base + stockage = deux lignes de facture distinctes (on y revient en Leçon 8 / FinOps).

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi c'est dangereux/inefficace | ✅ Version correcte |
|----------------|-------------------------------------|---------------------|
| Base RDS dans un subnet public | Accessible / attaquable depuis Internet | Subnet **privé**, accès limité à l'application |
| Mot de passe en dur dans le code (`P@ssw0rd`) | Fuite d'identifiants (Bloc 5 — secrets) | Identifiants dans un **coffre de secrets** (Leçon 6) |
| Ne jamais tester la restauration | Backup corrompu découvert le jour de la panne | Test de **restauration** régulier (DR, Bloc 8) |
| Prendre une grosse classe d'instance « au cas où » | Facture inutilement haute | Commencer petit et dimensionner ensuite (FinOps) |
| Se connecter en `localhost` depuis la VM alors que la base est ailleurs | Confusion et erreurs de connexion | Utiliser le bon **endpoint** et le bon **port** |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`**, la correction dans **`03-correction.md`**.

**Énoncé court** : installe/démarre PostgreSQL en local (ou réutilise celui du Bloc 2), connecte-toi avec `psql`, observe le port 5432, et rédige dans `notes-exercice-05.md` : ce que « managé » change pour toi (3 points), le schéma d'architecture avec RDS (qui parle à qui), et 3 règles de sécurité pour la base.

---

## 7. Correction détaillée de l'exercice

> La correction complète est dans **`03-correction.md`**. On y détaille la connexion `psql`, la justification du « managé » et le schéma réseau de référence.

---

## 8. Checklist de validation

- [ ] J'explique une base « managée » et ce que le cloud gère à ma place.
- [ ] Je définis RDS, moteur, instance, endpoint, port, backup, réplica, failover.
- [ ] Je sais pourquoi une base ne doit **jamais** être publique.
- [ ] Je connecte une base avec `psql` (local maintenant, RDS plus tard).
- [ ] Je pratique backups + point-in-time recovery (testés au moins une fois).
- [ ] Je place les identifiants dans un coffre et jamais dans Git.

---

🧭 **Pont vers la suite** — Ta base (RDS) et ton application ont besoin de **droits** : qui peut créer une base ? lire le bucket ? démarrer une VM ? C'est **IAM**, la gestion des identités et des permissions — et aussi le moment de créer tes **clés d'accès** pour enfin piloter AWS en ligne de commande (Leçon 6).

---

*Prochaine étape :* Leçon 6 — **IAM et sécurité des accès** dans `06-IAM-et-securite-des-acces/`.
