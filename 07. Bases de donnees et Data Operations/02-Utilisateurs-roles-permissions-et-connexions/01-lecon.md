# Leçon 2 — Utilisateurs, rôles, permissions et connexions

> **Bloc 7 · Bases de données & Data Operations** — Leçon 2 sur 8
> 🧭 **Pont depuis la Leçon 1** : tu sais maintenant créer une base et des tables, et écrire du SQL (le CRUD). Mais dans la Leçon 1, tu as fait **tout avec l'administrateur** (`postgres`) — la clé du patron, en quelque sorte. La roadmap exige ici : *« utilisateurs, rôles, permissions, connexions, configuration »*. Cette leçon répond à trois questions de production : **qui peut se connecter** (les utilisateurs), **qui a le droit de faire quoi** (les permissions), et **comment on entre dans la base** (les connexions et leur sécurisation — avec le TLS que tu connais du Bloc 5).

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Expliquer** pourquoi une application ne doit **jamais** se connecter avec l'utilisateur administrateur (le principe du moindre privilège).
2. **Créer** des rôles et utilisateurs PostgreSQL avec `CREATE ROLE` / `CREATE USER` et des mots de passe.
3. **Accorder et retirer** des permissions avec `GRANT` / `REVOKE` (lecture seule, lecture/écriture…).
4. **Comprendre** le fichier `pg_hba.conf`, qui décide **qui peut se connecter, d'où, à quelle base et comment**.
5. **Sécuriser** les connexions : authentification par mot de passe (`scram-sha-256`) et rappel du **TLS** (Bloc 5).
6. **Stocker** les identifiants de l'application **hors du code** (rappel secrets du Bloc 6).

---

## 2. Explication simple

### 2.1 Le « pourquoi » : le problème de la clé unique

Imagine un immeuble où **toutes les portes** (cave, chaudière, locaux voisins) s'ouvrent avec **une seule clé**, confiée à tout le monde. La première clé perdue compromet **tout l'immeuble**.

```
Avec un seul utilisateur « postgres » (admin) :
  développeur  → postgres → peut tout (effacer la production !)
  application  → postgres → si elle est piratée, TOUT est perdu
  stagiaire    → postgres → accélère une panne qu'on voulait éviter
```

> 💡 **Analogie** : à la place, chaque personne reçoit **une clé de sa propre porte** : la bibliothécaire ouvre les rayonnages mais pas la salle des coffres ; l'application écrit dans sa base mais ne peut pas lire les tables des autres applications... ni **effacer** la bibliothèque entière.

De plus, dans cette leçon, le mot « utilisateur » est utilisé pour deux choses différentes. Retiens la distinction : un **utilisateur du SGBD** n'a RIEN à voir avec un **utilisateur de ton application** (ex. un lecteur qui se connecte au site). Pour éviter la confusion, cette leçon parle de **rôles** (le mot exact de PostgreSQL) quand elle désigne les comptes du SGBD (`postgres`, `app_biblio`...), et d'**utilisateur applicatif** quand la confusion est possible.

C'est le principe du **moindre privilège** — déjà vu au Bloc 5 (contrôle d'accès) et au Bloc 6 (IAM d'AWS) : **donner seulement les droits nécessaires, rien de plus**. Bonne nouvelle : PostgreSQL l'applique **nativement**, car un utilisateur PostgreSQL est un **rôle** (role) — c'est le modèle « contrôle d'accès par rôles » (**RBAC**, Role-Based Access Control, déjà défini au Bloc 5).

### 2.2 Le « comment » (partie 1) : rôles, utilisateurs, permissions

En PostgreSQL, il n'y a qu'**un seul objet** d'identité : le **rôle**. Deux habitudes de vocabulaire :

- un rôle **sans droit de connexion** sert de **groupe** (on y ajoute d'autres rôles) ;
- un rôle **avec droit de connexion** (`LOGIN`) est en pratique un **utilisateur** — c'est exactement ce que crée `CREATE USER`, qui est l'abréviation de `CREATE ROLE ... LOGIN`.

Les **permissions** (aussi appelées privilèges) se donnent avec **`GRANT`** (accorder) et se retirent avec **`REVOKE`** (révoquer). Les principales :

| Permission | Elle autorise à… | Analogie |
|---|---|---|
| `CONNECT` | se connecter à une **base** | entrer dans le bâtiment |
| `USAGE` | utiliser un **schéma** (le « dossier » qui regroupe les tables) | circuler dans un étage |
| `SELECT` | **lire** les lignes d'une table | consulter un dossier |
| `INSERT` | **ajouter** des lignes | déposer un dossier |
| `UPDATE` | **modifier** des lignes | corriger un dossier |
| `DELETE` | **supprimer** des lignes | jeter un dossier |

Et le principe de départ est le **refus par défaut** : un rôle fraîchement créé ne peut **rien** faire tant que tu ne lui donnes pas explicitement des droits. La sécurité vient de ce que tu **décides d'accorder** — et de ce que tu oublies d'accorder, il refuse tout seul.

### 2.3 Le « comment » (partie 2) : les connexions — `pg_hba.conf`

Créer un utilisateur ne suffit pas : il faut décider **comment il peut entrer**. C'est le rôle du fichier **`pg_hba.conf`** (« pg Host-Based Authentication », littéralement « authentification basée sur l'hôte ») : une **liste de règles lues de haut en bas**, où **la première règle qui correspond gagne**.

Chaque règle répond à 5 questions :

```
TYPE        local ou host      → « socket locale » (sans réseau) ou TCP (réseau, même localhost)
BASE        à quelle(s) base(s)
UTILISATEUR à quel(s) rôle(s)
ADRESSE     d'où (une adresse IP ou une plage — la notation CIDR vue au Bloc 5)
MÉTHODE     comment il prouve son identité : peer, scram-sha-256, ...
```

Les deux méthodes à connaître :

- **peer** : uniquement en local, **sans mot de passe** — le système compare ton compte Linux à l'utilisateur du SGBD demandé. C'est ce qui permettait `sudo -u postgres psql` (Bloc 2 et Leçon 1).
- **scram-sha-256** : l'authentification **moderne par mot de passe**. Le serveur ne stocke **jamais le mot de passe en clair** : il stocke une empreinte vérifiable par un calcul cryptographique (SHA-256, une fonction de hachage — le « mixeur » qui transforme un texte en empreinte irréversible, vu au Bloc 5). C'est la méthode par défaut depuis PostgreSQL 14.

> 💡 **Analogie pour `pg_hba.conf`** : c'est le **registre du portier** de l'immeuble. Chaque ligne dit : « si quelqu'un se présente à CETTE porte (type + adresse), pour CE service (base), avec CE nom (rôle), alors demande LUI ce justificatif (méthode) ». Le portier lit le registre **de haut en bas** et s'arrête à la première ligne qui correspond.

Et pour le **trajet réseau** : dès que la base n'est plus sur la même machine que l'application, la connexion doit être **chiffrée avec TLS** (Transport Layer Security — le chiffrement de la communication, vu au Bloc 5, Leçon 4). Sinon, mot de passe et données circulent **en clair** sur le réseau, lisibles par n'importe quel intermédiaire.

> 🔁 **Pont vers le Bloc 6 (RDS)** : sur une base **managée** (RDS), tu ne touches pas à `pg_hba.conf` — le fournisseur le gère. Chez AWS, l'équivalent « qui peut joindre la base depuis où » se règle avec les **security groups** (les pare-feux cloud du Bloc 6, Leçon 2). Rappel utile : le principe est le même, seul le fichier change.

### 2.4 Le « quand » : où stocker le mot de passe de l'application ?

Dès qu'une application se connecte à la base, elle a besoin d'identifiants (nom d'utilisateur + mot de passe — en anglais **credentials**). La règle est ferme : **jamais dans le code, jamais dans Git**. La solution standard 2025-2026 :

```
Coffre-fort de secrets (rappel Bloc 6 : AWS Secrets Manager)
   ↓ fournit au démarrage
Variables d'environnement (des valeurs définies HORS du code quand l'application démarre)
   ↓ lues par
application.properties : password=${DB_PASSWORD}   ← le code ne contient QUE le NOM de la variable
```

> 💡 **Analogie** : le code de l'application, c'est un plan de bâtiment qu'on **partage** (dans Git). On n'y écrit jamais le numéro du coffre — seulement « voir coffre ».

---

## 📖 Vocabulaire / Abréviations

> Définitions d'une ligne pour ne jamais être perdu(e).

- **Rôle (role)** : l'unique objet d'identité de PostgreSQL ; peut être un utilisateur (avec `LOGIN`) ou un groupe.
- **Utilisateur** : un rôle avec le droit de connexion (`LOGIN`).
- **Moindre privilège** : donner seulement les droits nécessaires, rien de plus.
- **RBAC** (Role-Based Access Control) : contrôle d'accès par rôles (Bloc 5 ; ici natif PostgreSQL).
- **GRANT / REVOKE** : accorder / retirer une permission.
- **Permission (privilège)** : un droit précis (`SELECT`, `INSERT`, `UPDATE`, `DELETE`, `CONNECT`, `USAGE`…).
- **Schéma** : le « dossier » qui regroupe les tables (par défaut : `public`).
- **Séquence** : le compteur interne des colonnes auto-numérotées (vu en Leçon 1).
- **`pg_hba.conf`** : le fichier de règles qui décide qui peut se connecter, d'où, à quelle base, avec quelle méthode.
- **peer** : méthode d'authentification locale sans mot de passe (compte Linux = utilisateur du SGBD).
- **scram-sha-256** : authentification moderne par mot de passe, jamais stocké en clair (SHA-256 = fonction de hachage cryptographique, Bloc 5).
- **TLS** (Transport Layer Security) : le chiffrement de la communication (Bloc 5, Leçon 4).
- **Credentials** : les identifiants (utilisateur + mot de passe) de connexion.
- **Secret** : toute donnée sensible à protéger (mot de passe, clé, token).
- **Variable d'environnement** : valeur fournie au démarrage d'un programme, **hors du code**.
- **Coffre-fort de secrets** (ex. AWS Secrets Manager) : l'endroit central et chiffré où vivent les secrets (Bloc 6).
- **Socket locale** : canal de communication **sans réseau** entre deux processus de la même machine.
- **localhost / 127.0.0.1** : « cette machine elle-même » — vu au Bloc 5.
- **CIDR** : notation d'une plage d'adresses IP (ex. `127.0.0.1/32` = exactement cette adresse) — vu au Bloc 5.
- **PUBLIC** : le pseudo-rôle « tout le monde » dans PostgreSQL.
- **SUPERUSER** : rôle tout-puissant (l'équivalent « root » du SGBD) — à ne presque jamais accorder.

---

## 3. Exemples concrets

> 🔁 On enchaîne logiquement : la théorie a défini le vocabulaire ; ces exemples sont **les commandes exactes** à copier-coller, commentées ligne par ligne. Tout est local et gratuit.

### 3.1 Créer les rôles (en tant qu'administrateur)

```bash
sudo -u postgres psql       # se connecte en tant qu'administrateur local (méthode « peer »)
```

```sql
CREATE ROLE app_biblio LOGIN PASSWORD 'Biblio-App-2026-moulin!coffre'
  NOSUPERUSER NOCREATEDB NOCREATEROLE NOINHERIT;
-- CREATE ROLE : crée un rôle ; LOGIN : il peut se connecter (c'est un utilisateur)
-- PASSWORD '...' : SA phrase de passe — en apostrophes SIMPLES (les " doubles sont réservées aux noms d'objets)
-- NOSUPERUSER : pas de pouvoirs tout-puissants ; NOCREATEDB : ne peut pas créer de bases ;
-- NOCREATEROLE : ne peut pas créer de rôles. On liste les refus POUR LE RELIRE plus tard.

CREATE ROLE lecteur_biblio LOGIN PASSWORD 'Biblio-Lecteur-2026-papier!plume';
-- même principe pour l'analyste

\du        -- liste les rôles : « Attributs » doit rester VIDE de pouvoirs d'administration
```

> 💡 **Où stocker ces phrases de passe ?** Pour l'exercice : dans un gestionnaire de mots de passe (Bitwarden, KeePassXC — des « coffres personnels »). Pour l'application : voir 3.6. Et **pour changer un mot de passe sans le coller dans l'historique du shell** : `\password app_biblio` dans psql (il te le demande en caché).

### 3.2 Accorder les permissions

```sql
-- PARTIE COMMUNE : entrer dans la base (depuis l'admin, base « postgres »)
GRANT CONNECT ON DATABASE bibliotheque TO app_biblio;
GRANT CONNECT ON DATABASE bibliotheque TO lecteur_biblio;
-- GRANT <quoi> ON <sur quel objet> TO <qui> : « accorde la connexion à la base bibliotheque au rôle app_biblio »

\c bibliotheque        -- l'admin entre dans la base pour accorder les droits internes

-- (1) UTILISER LE SCHÉMA (le « dossier » qui contient les tables)
GRANT USAGE ON SCHEMA public TO app_biblio;
GRANT USAGE ON SCHEMA public TO lecteur_biblio;

-- (2) L'APPLICATION : lire + écrire sur les tables EXISTANTES
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_biblio;

-- (3) L'APPLICATION : la même chose sur les tables créées PLUS TARD
ALTER DEFAULT PRIVILEGES IN SCHEMA public
  GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO app_biblio;
-- Pourquoi : les GRANT ne touchent que les tables EXISTANTES. Cette commande règle les tables FUTURES.

-- (4) L'APPLICATION : utiliser les SÉQUENCES (les compteurs des colonnes auto-numérotées, vu en Leçon 1)
GRANT USAGE ON ALL SEQUENCES IN SCHEMA public TO app_biblio;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT USAGE ON SEQUENCES TO app_biblio;
-- Sans cela, le premier INSERT échouerait : écrire une ligne demande de fabriquer son numéro.

-- (5) L'ANALYSTE : lecture seule, sur les tables existantes ET futures
GRANT SELECT ON ALL TABLES IN SCHEMA public TO lecteur_biblio;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO lecteur_biblio;

-- (6) LE NETTOYAGE : le pseudo-rôle PUBLIC (« tout le monde ») n'a pas besoin d'entrer ici
REVOKE CONNECT ON DATABASE bibliotheque FROM PUBLIC;
REVOKE CREATE ON SCHEMA public FROM PUBLIC;
-- Pourquoi : par défaut, « tout le monde » peut se connecter aux bases et créer des objets.
-- On ferme la porte large avant d'avoir distribué les clés précises (moindre privilège).

\du    -- vérifie : les attributs restent « simples » ; les droits se vérifient plutôt par le TEST (3.3)
```

### 3.3 Tester par la connexion (le test qui prouve tout)

```bash
psql -h localhost -p 5432 -U app_biblio -d bibliotheque
# -h : l'HÔTE, c'est-à-dire l'adresse du serveur (localhost = cette machine elle-même)
# -p : le PORT, le numéro de guichet du service (5432 = PostgreSQL, vu au Bloc 5)
# -U : l'utilisateur (le rôle) ; -d : la base ; le mot de passe est demandé (scram-sha-256)
```

Dans cette session **`app_biblio`**, les 3 tests attendus :

```sql
INSERT INTO livres (titre, auteur, annee_publication) VALUES ('1984', 'George Orwell', 1949);
-- ✅ RÉUSSIT : elle a le droit d'écrire (INSERT accordé)

SELECT titre FROM livres;                -- ✅ RÉUSSIT : lecture accordée

CREATE TABLE pirate (x INT);             -- ❌ ÉCHOUE : « permission denied for schema public »
-- Preuve : elle ne peut PAS créer de tables (aucun GRANT CREATE donné)
```

Dans une session **`lecteur_biblio`** :

```sql
SELECT titre FROM livres;                -- ✅ RÉUSSIT : lecture accordée
DELETE FROM livres;                      -- ❌ ÉCHOUE : « permission denied for table livres »
-- C'est la preuve que ta protection marche : le refus est le BUT recherché ici
```

> 💡 **Le réflexe DevOps** : une permission se **prouve par un test**, pas par une lecture du fichier de config. « Ça devrait marcher » ne vaut rien ; « j'ai vu l'erreur permission denied » est une preuve.

### 3.4 Lire et comprendre `pg_hba.conf`

```bash
sudo -u postgres psql          # la lecture du fichier se fait depuis l'admin
```

```sql
SHOW hba_file;                 -- affiche le CHEMIN EXACT du fichier (il change selon la version :
                               -- ex. /etc/postgresql/16/main/pg_hba.conf pour PostgreSQL 16).
                               -- NE DEVINE PAS le chemin : lis-le.
\q                             -- quitte psql
```

```bash
HBA="$(sudo -u postgres psql -tAc 'SHOW hba_file;')"
# HBA : une variable shell qui reçoit le chemin du fichier affiché par le SGBD LUI-MÊME (pas de chemin à deviner).
# Options de psql : -t = sortie brute (sans cadre de tableau) ; -A = sans alignement ; -c = exécute la commande SQL.
sudo nano "$HBA"    # ouvre le fichier affiché (nano : Ctrl+O sauve, Ctrl+X quitte — vu au Bloc 2)
```

Les lignes à repérer (une installation standard d'Ubuntu) :

```
local   all             postgres                peer        # pourquoi « sudo -u postgres psql » marchait
local   all             all                     peer        # connexions locales SANS réseau
host    all             all             127.0.0.1/32    scram-sha-256
        # host = via le réseau (TCP) ; all = toutes les bases et rôles ; 127.0.0.1/32 = depuis CETTE machine
        # scram-sha-256 = « demande un mot de passe » — pourquoi ta connexion -h localhost a fonctionné
```

**Comment ajouter une règle précise** (à mettre **AVANT** la règle générique `host all all`, car la première qui correspond gagne) :

```
# La base bibliotheque, pour app_biblio, uniquement depuis la machine applicative :
host    bibliotheque    app_biblio      192.168.1.50/32    scram-sha-256
```

```bash
sudo systemctl reload postgresql    # recharger = relire la configuration SANS couper les connexions actives
# (restart, lui, couperait tout — à réserver aux changements qui l'exigent)
```

### 3.5 Le chiffrement du trajet : TLS et `sslmode`

Pour une connexion **distante** (l'application sur une machine, la base sur une autre) :

```bash
psql "host=ma-base.mon-cloud.amazonaws.com port=5432 dbname=bibliotheque user=app_biblio sslmode=require"
# le format « chaîne de connexion » regroupe tout en un argument
# sslmode=require : « exige le chiffrement TLS du trajet, sinon refuse de se connecter »
# (sur une base managée du Bloc 6, le certificat du serveur est géré par le fournisseur)
```

> 🔁 **Rappel Bloc 5 (Leçon 4)** : TLS chiffre la communication entre deux machines grâce à des certificats. En local (`localhost`), le trajet ne quitte pas ta machine — TLS n'est pas exigé. **En production, `sslmode=require` est le réflexe.**

### 3.6 Les identifiants de l'application : variables d'environnement

Dans le projet Spring Boot, `src/main/resources/application.properties` :

```properties
# Le code ne contient QUE les NOMS de variables — jamais les valeurs
spring.datasource.url=jdbc:postgresql://localhost:5432/bibliotheque
spring.datasource.username=${DB_USER}          # ${...} = « lis la variable d'environnement »
spring.datasource.password=${DB_PASSWORD}
```

Au démarrage, le shell fournit les valeurs (elles ne vivent **ni dans le code, ni dans Git**) :

```bash
export DB_USER=app_biblio                       # export = définit une variable d'environnement pour la session
export DB_PASSWORD='Biblio-App-2026-moulin!coffre'
./mvnw spring-boot:run                          # lance l'application (elle lit les variables au démarrage)
```

> 🔁 **Rappel Bloc 6 (IAM, Leçon 6)** : en production, ces variables sont remplies par un **coffre-fort de secrets** (ex. AWS Secrets Manager) — jamais collées à la main.

---

## 4. Bonnes pratiques modernes (2025-2026)

1. **Un rôle par application et par service** : `app_biblio` pour Spring Boot, `lecteur_biblio` pour l'analyste — jamais un compte partagé « entre nous ».
2. **`NOSUPERUSER NOCREATEDB NOCREATEROLE`** sur chaque rôle applicatif : les pouvoirs d'administration restent à l'admin.
3. **Refus par défaut + `GRANT` précis** : accorder les permissions, ne jamais compter sur « tout le monde » (`PUBLIC` — ferme la porte large avec `REVOKE ... FROM PUBLIC`).
4. **Authentification `scram-sha-256` partout** (méthode moderne, jamais l'ancienne `md5` ni surtout `trust` qui n'exige **aucun** mot de passe).
5. **`pg_hba.conf` : règles précises en haut** (base + rôle + adresse exacte), règles génériques en bas — et `systemctl reload` (jamais `restart` quand des utilisateurs sont connectés).
6. **TLS exigé dès que le trajet quitte la machine** (`sslmode=require`), notamment vers une base managée du Bloc 6.
7. **Secrets hors du code** : variables d'environnement remplies par un coffre-fort (Bloc 6) ; rotation des mots de passe périodique (`\password` pour le faire sans historique).
8. **Un backup des permissions** : tout ce qui est écrit à la main (`CREATE ROLE`, `GRANT`) doit être **versionné dans Git** comme du code (ce sera automatique avec les migrations de la Leçon 5 et l'IaC du Bloc 8).

---

## 5. Pièges à éviter

### Piège 1 — Le mot de passe en dur dans le code

```properties
# ❌ MAUVAIS : versionné dans Git = remis à chaque « git push » au monde entier
spring.datasource.password=Biblio-App-2026-moulin!coffre

# ✅ CORRECT : le nom de la variable, la valeur vient du démarrage/coffre
spring.datasource.password=${DB_PASSWORD}
```

**Pourquoi** : Git **conserve l'historique** — même retiré plus tard, le mot de passe reste dans les commits passés. Et un dépôt « privé » le devient rarement.

### Piège 2 — L'application avec le compte admin

```sql
-- ❌ MAUVAIS : l'application se connecte avec « postgres » (SUPERUSER)
-- → piratée, elle peut effacer TOUTES les bases, créer des rôles, détruire le serveur.

-- ✅ CORRECT : rôle dédié au moindre privilège
CREATE ROLE app_biblio LOGIN PASSWORD '...' NOSUPERUSER NOCREATEDB NOCREATEROLE;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_biblio;
```

**Pourquoi** : la surface d'attaque (ce qu'un attaquant peut toucher) se réduit à **ce que le rôle peut faire**.

### Piège 3 — La méthode `trust` dans `pg_hba.conf`

```
❌ MAUVAIS : host  all  all  0.0.0.0/0  trust
   → « n'importe qui, de n'importe où, SANS mot de passe » : porte ouverte sur Internet.

✅ CORRECT : host  bibliotheque  app_biblio  192.168.1.50/32  scram-sha-256
   → une base, un rôle, une adresse précise, un mot de passe exigé.
```

**Pourquoi** : `0.0.0.0/0` signifie « tout Internet » (le CIDR du Bloc 5) ; `trust` supprime la preuve d'identité. Combinés, n'importe quel robot d'attaque entre en quelques minutes.

### Piège 4 — `GRANT ALL` « pour avancer vite »

```sql
-- ❌ MAUVAIS : on ne sait pas ce qu'on a donné
GRANT ALL PRIVILEGES ON DATABASE bibliotheque TO app_biblio;

-- ✅ CORRECT : on liste ce qui est nécessaire
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_biblio;
```

**Pourquoi** : « pour avancer vite » donne des droits inconnus qui survivent des années. Le jour de l'incident, personne ne sait ce que le rôle pouvait faire.

### Piège 5 — `systemctl restart` pour un changement de permissions

```bash
# ❌ MAUVAIS : coupe TOUTES les connexions actives (panne pendant le reload)
sudo systemctl restart postgresql

# ✅ CORRECT : relit la configuration sans couper personne
sudo systemctl reload postgresql
```

**Pourquoi** : `reload` suffit pour `pg_hba.conf` et la plupart des réglages ; `restart` est réservé aux changements qui l'exigent (Leçon 3). Chaque coupure inutile = une micro-panne évitable.

---

## 6. Exercice pratique

> 🔁 **Comment s'articulent les fichiers** : la théorie est terminée, passons à la pratique. L'exercice complet est dans **`02-exercice.md`** (à faire **avant** de lire la correction).

En résumé, tu vas — **en local, gratuitement** :

1. créer deux rôles : `app_biblio` (l'application : lire **et** écrire) et `lecteur_biblio` (l'analyste : **lecture seule**) ;
2. accorder **exactement** les droits nécessaires (`CONNECT`, `USAGE`, `SELECT`, `INSERT`, `UPDATE`, `DELETE`, séquences) ;
3. **prouver par le test** que chaque rôle peut faire ce qu'il doit — et **rien** de plus (les erreurs « permission denied » attendues) ;
4. lire `pg_hba.conf` et comprendre pourquoi `sudo -u postgres psql` (peer) et `psql -h localhost` (scram) fonctionnaient ;
5. justifier où vit le mot de passe de l'application (rappel secrets/coffre du Bloc 6).

Livrable : `notes-exercice-02.md`.

---

## 7. Correction détaillée de l'exercice

La correction complète (pas à pas, sorties attendues, explications des choix — notamment `ALTER DEFAULT PRIVILEGES` et le `REVOKE ... FROM PUBLIC`) est dans **`03-correction.md`**, qui réécrit la checklist finale et donne des conseils.

---

## 8. Checklist de validation

- [ ] Je peux expliquer le moindre privilège avec l'analogie des clés, et pourquoi l'application ne se connecte **jamais** avec `postgres`.
- [ ] Je sais créer un rôle utilisateur avec mot de passe et **sans pouvoirs d'administration** (`NOSUPERUSER NOCREATEDB NOCREATEROLE`).
- [ ] Je sais accorder des permissions précises (`GRANT`) et les retirer (`REVOKE`), y compris pour les **tables futures** (`ALTER DEFAULT PRIVILEGES`).
- [ ] Je sais **prouver** par un test de connexion qu'un rôle peut faire ce qu'il doit — et pas plus (les « permission denied » attendus).
- [ ] Je comprends `pg_hba.conf` : ses 5 colonnes, l'ordre des règles (la première qui correspond gagne), et la différence `peer` / `scram-sha-256`.
- [ ] Je sais la différence `systemctl reload` / `restart` et quand utiliser l'un ou l'autre.
- [ ] Je sais où vit le mot de passe de l'application (variable d'environnement + coffre, jamais le code ni Git).

---

> 🧭 **Prochaine étape** : ta base est maintenant **bien gardée** — qui peut entrer et qui peut toucher à quoi est défini. Mais une base bien gardée peut encore **mal tourner** : lente, saturée, en panne. La **Leçon 3** (configuration et ressources) t'apprend à **régler** le SGBD (`postgresql.conf` : mémoire, nombre de connexions, journaux) et à **observer** ce qu'il consomme — la suite directe des ressources vues en Leçon 1.
