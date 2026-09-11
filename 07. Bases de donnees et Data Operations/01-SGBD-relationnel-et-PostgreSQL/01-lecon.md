# Leçon 1 — Le SGBD relationnel et PostgreSQL

> **Bloc 7 · Bases de données & Data Operations** — Leçon 1 sur 8
> 🧭 **Pont depuis le Bloc 6 (Cloud Providers)** : au Bloc 6, tu as **loué** une base de données « managée » (RDS) et le cloud s'occupait du capot : installation, sauvegardes, mises à jour. Mais la Leçon 5 t'avait prévenu(e) : *« le cloud gère la base, pas tes requêtes ni ta modélisation — c'est le Bloc 7 qui s'en charge »*. La promesse est tenue ici. D'ailleurs, tu as déjà **installé PostgreSQL** au Bloc 2 (Leçon 6 « Paquets et installation Nginx/PostgreSQL ») sans comprendre ce qui se passait à l'intérieur. Cette leçon **ouvre le capot** : tu vas comprendre ce qu'est une base de données **relationnelle**, le langage **SQL**, et pourquoi PostgreSQL est le choix moderne.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Expliquer** ce qu'est une base de données, un SGBD et une base « relationnelle » (avec l'analogie du classeur).
2. **Écrire** les 4 opérations de base en SQL (créer, lire, modifier, supprimer — le « CRUD ») sur une table PostgreSQL.
3. **Différencier** PostgreSQL et MySQL, et savoir ce que sont MongoDB et OracleDB **sans creuser** (la roadmap le demande explicitement).
4. **Naviguer** dans `psql`, l'outil en ligne de commande de PostgreSQL.
5. **Observer** les ressources d'une base (taille sur disque, connexions actives) — porte d'entrée de la Leçon 3 (configuration).

---

## 2. Explication simple

### 2.1 Le « pourquoi » : à quoi sert une base de données ?

Ton application fil rouge (backend **Spring Boot** + frontend **Angular**) manipule des **données structurées** : des utilisateurs, des commandes, des livres. Pour les stocker, trois choix sont possibles :

```
Choix 1 : dans des fichiers texte   → lent à chercher, se casse facilement, impossible d'écrire à plusieurs en même temps
Choix 2 : dans la mémoire vive (RAM) → tout disparaît à chaque redémarrage
Choix 3 : dans une BASE DE DONNÉES   → rapide, fiable, sécurisé, plusieurs personnes en même temps ✓
```

> 💡 **Analogie** : une base de données, c'est un **classeur professionnel** : des tiroirs (les **tables**), des fiches standardisées dedans (les **lignes**), et un(e) secrétaire rigoureux(se) qui vérifie chaque fiche avant de la ranger (le **SGBD**).

**Définitions clés** (retiens-les : tout le bloc les réutilise) :

- **Donnée structurée** : une information qui a toujours la même forme (un « utilisateur » a toujours un email, un nom, une date d'inscription).
- **SGBD** (Système de Gestion de Base de Données) : le logiciel qui **gère** la base pour toi — il vérifie, protège, sauvegarde et donne accès aux données. PostgreSQL et MySQL sont des SGBD.
- **SQL** (Structured Query Language, « langage de requête structuré ») : le langage que tu utilises pour **parler** au SGBD (lui demander de lire, d'écrire, de modifier, de supprimer).

### 2.2 Le « comment » : la base « relationnelle »

Une base **relationnelle** range les données dans des **tables**, exactement comme une feuille de tableur :

```
Table « utilisateurs » (1 ligne = 1 utilisateur ; 1 colonne = 1 information)
+----+-------+---------------------+-------------+
| id | nom   | email               | inscrite_le |
+----+-------+---------------------+-------------+
|  1 | Amina | amina@example.com   | 2025-03-01  |
|  2 | Jean  | jean@example.com    | 2025-06-12  |
+----+-------+---------------------+-------------+
```

Les règles du jeu :

| Terme | C'est quoi ? | Pourquoi c'est important |
|---|---|---|
| **Colonne typée** | Chaque colonne a un **type** (texte, nombre, date…) | Le SGBD **refuse** de ranger « douze » dans une colonne de nombres → moins d'erreurs |
| **Ligne** | Une fiche complète | L'unité de lecture/écriture |
| **Clé primaire** (primary key) | Le **numéro unique** de chaque ligne (souvent `id`) | Permet de retrouver et modifier **exactement** la bonne ligne |
| **Clé étrangère** (foreign key) | Une colonne qui **pointe** vers la clé primaire d'une autre table | C'est le « lien » entre les tables |

**D'où vient le mot « relationnel » ?** Les tables sont **reliées entre elles** :

```
Table « livres »                      Table « emprunts »
+----+------------------+--------+    +----+----------+-------------+
| id | titre            | auteur |    | id | livre_id | utilisateur |
+----+------------------+--------+    +----+----------+-------------+
|  1 | Le Petit Prince  | 1943   |    |  1 |    1     |      2      |  ← livre_id = 1 pointe vers le livre n° 1
+----+------------------+--------+    +----+----------+-------------+
```

Quand tu lis un emprunt, tu peux **relier** (faire une **jointure**) avec la table des livres pour afficher le titre : « Jean a emprunté *Le Petit Prince* ». C'est la force du relationnel : **des données reliées, avec des règles vérifiées**.

**Une précision utile pour la suite** : la colonne `id` est en général **auto-numérotée** — le SGBD fabrique lui-même le numéro suivant grâce à un compteur interne appelé **séquence**. Tu n'as donc pas besoin de fournir le `id` quand tu ajoutes une ligne.

### 2.3 PostgreSQL vs MySQL (et les autres, « à définir sans creuser »)

Passons du « comment ça marche » au « **lequel choisir** » — la roadmap demande de connaître les SGBD principaux, en distinguant le cœur du bloc et ceux qu'il faut **savoir définir sans creuser**.

| SGBD | Ce que c'est | Quand on le croise | Ton niveau requis |
|---|---|---|---|
| **PostgreSQL** | Le SGBD relationnel **libre** (gratuit) le plus complet : fiable, strict, moderne | Le choix par défaut 2025-2026 ; ta base fil rouge ; RDS du Bloc 6 | ✅ **Cœur du bloc** |
| **MySQL** | L'autre SGBD relationnel libre, très répandu (écosystème web historique, WordPress…) | Beaucoup d'applications existantes ; RDS le fournit aussi | ✅ Savoir expliquer la différence |
| **MongoDB** | Une base **NoSQL** (« pas du SQL ») : elle range des **documents** (texte structuré **JSON** — le format d'échange du Bloc 3), **sans schéma fixe ni jointures** | Le backup/réplication existent aussi en NoSQL, mais avec d'autres mécanismes (« replica set » plutôt que primary/replica classique) | 🔵 **Définir, ne pas creuser** (roadmap) |
| **OracleDB** | SGBD **propriétaire** d'entreprise : payant, présent surtout en grands comptes et systèmes anciens (« **legacy** ») avec licences coûteuses | Banques, assurances, grands groupes | 🟡 **Définir, ne pas creuser** (roadmap) |

**PostgreSQL ou MySQL ?** Les deux font le travail d'une application standard. Trois raisons penchent vers **PostgreSQL** aujourd'hui :

1. **Plus strict** : il refuse les données incohérentes (dates invalides, nombres mal formés) — des garde-fous précieux.
2. **Plus riche** : types avancés (UUID, JSON typé, plages de dates…).
3. **L'écosystème moderne** : ta stack Spring Boot et les outils 2025-2026 l'assument très bien.

> 💡 **Analogie** : MySQL, c'est la voiture citadine éprouvée qui démarre toujours ; PostgreSQL, c'est la voiture moderne avec les capteurs de sécurité en plus. Pour apprendre **bien**, apprends avec les capteurs.

### 2.4 Le « quand » : les ressources d'une base (disque, mémoire, connexions)

Dernier point avant les exemples — il prépare les Leçons 2 et 3. Une base de données est un **processus** sur ta machine (comme les services du Bloc 2, Leçon 4) : elle **consomme** trois ressources :

```
Disque     → les données elles-mêmes (croissent avec le temps)
RAM        → un cache interne : le SGBD garde en mémoire ce qu'il a lu, pour aller plus vite
Connexions → chaque application qui se connecte occupe un « guichet » (nombre limité : max_connections)
```

**Pourquoi c'est le cœur du métier DevOps ?** Une application « lente », c'est très souvent **la base** qui suffoque : disque plein (plus rien n'écrit), cache trop petit (tout est relu à chaque fois), ou **guichets épuisés** (l'application attend une connexion libre). Détecter cela **par observation** plutôt qu'à l'aveugle, c'est la Leçon 3.

> 💡 **Analogie** : la base, c'est un restaurant. Le disque = le garde-manger ; la RAM = le plan de travail (ce qu'on garde à portée de main) ; les connexions = les tables du restaurant — si toutes sont occupées, les clients attendent dehors.

---

## 📖 Vocabulaire / Abréviations

> Définitions d'une ligne pour ne jamais être perdu(e).

- **Base de données** : stockage structuré, fiable et rapide pour les données d'une application.
- **SGBD** (Système de Gestion de Base de Données) : le logiciel qui gère la base (PostgreSQL, MySQL…).
- **SQL** (Structured Query Language) : le langage pour interroger une base relationnelle.
- **Relationnelle** : base en **tables reliées entre elles** par des clés.
- **Table** : un tiroir du classeur (1 ligne = 1 fiche).
- **Colonne typée** : champ avec un type (texte, entier, date…).
- **Ligne** : une fiche complète.
- **Clé primaire** (primary key) : le numéro unique d'une ligne.
- **Clé étrangère** (foreign key) : colonne qui pointe vers la clé primaire d'une autre table.
- **Jointure** : lire plusieurs tables reliées en un seul résultat.
- **Séquence** : compteur interne qui fabrique les numéros automatiques (les `id`).
- **CRUD** (Create, Read, Update, Delete) : les 4 opérations de base (créer, lire, modifier, supprimer).
- **psql** : l'outil en ligne de commande de PostgreSQL.
- **Transaction** : groupe d'opérations qu'on **valide** (`COMMIT`) ou **annule** (`ROLLBACK`) en bloc.
- **JSON** (JavaScript Object Notation) : format de texte structuré très répandu (vu au Bloc 3).
- **NoSQL** : « pas du SQL » — bases qui ne rangent pas en tables relationnelles (MongoDB : des documents).
- **Legacy** : système ancien, hérité du passé, difficile à remplacer.
- **RDS** (Relational Database Service) : le service d'AWS qui loue des bases « managées » (Bloc 6).
- **RAM** (mémoire vive) : mémoire rapide mais **effacée** à chaque extinction.
- **max_connections** : le nombre maximal de connexions simultanées du SGBD.
- **pg_stat_activity** : la table « surveillance » qui liste les connexions actives.

---

## 3. Exemples concrets

> ⚠️ **Réalité pratique** : toutes les commandes suivantes sont **100 % locales et gratuites** (PostgreSQL installé au Bloc 2). On utilise ici l'administrateur local — la Leçon 2 t'apprendra à te connecter **avec un rôle dédié et un mot de passe**.

### 3.1 Se connecter et créer la base

```bash
psql --version                       # vérifie que l'outil existe (s'il manque : sudo apt install -y postgresql)
sudo systemctl status postgresql --no-pager | head -5    # état du service (--no-pager = pas de vue interactive ; head -5 = 5 lignes)
sudo -u postgres psql                # se connecte en tant qu'administrateur local « postgres »
```

> 💡 **Explication de `sudo -u postgres`** : l'option `-u` (de `sudo`) signifie « exécute la commande en tant que l'utilisateur **système** `postgres` ». PostgreSQL a créé ce compte à l'installation. En local, c'est la voie normale — l'authentification « **peer** » exige que ton compte Linux corresponde à l'utilisateur du SGBD. La Leçon 2 t'apprendra les autres méthodes (mot de passe, chiffrement).

```sql
\l                                   -- liste les bases existantes
CREATE DATABASE bibliotheque;        -- crée la base du fil rouge (chaque commande SQL finit par un point-virgule !)
\c bibliotheque                      -- « connect » : entre dans la base
\dt                                  -- « display tables » : liste les tables (vide pour l'instant)
```

> 💡 **Le piège du débutant** : oublier le `;` final. psql croit alors que tu continues ta commande et affiche l'invite `bibliotheque-#` — tape `;` puis Entrée pour la terminer.

### 3.2 Créer la table (le « schéma ») et remplir-la

On enchaîne logiquement : la base existe, il faut définir **la structure** de ses données.

```sql
CREATE TABLE livres (                                -- crée la table « livres »
  id                GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  -- id : clé primaire auto-numérotée par une SÉQUENCE (le compteur interne).
  -- GENERATED ALWAYS : c'est PostgreSQL qui fabrique le numéro (tu ne peux pas le forcer).
  -- C'est la bonne pratique 2025-2026 (l'ancienne forme s'appelait SERIAL).
  titre             VARCHAR(200) NOT NULL,
  -- VARCHAR(200) : texte court de 200 caractères max ; NOT NULL : obligatoire (refusé si vide)
  auteur            VARCHAR(120) NOT NULL,           -- même logique
  annee_publication INT,                             -- INT : nombre entier
  prix              NUMERIC(8, 2) NOT NULL DEFAULT 0.00,
  -- NUMERIC(8,2) : décimal EXACT, 8 chiffres dont 2 après la virgule.
  -- C'est LE type pour l'argent (voir « Pièges à éviter »). DEFAULT : valeur par défaut.
  cree_le           TIMESTAMPTZ NOT NULL DEFAULT now()
  -- TIMESTAMPTZ : date + heure AVEC fuseau horaire ; now() : l'heure actuelle
);

\d livres        -- « describe » : affiche les colonnes, types, défauts de la table
```

Puis les 4 opérations CRUD, chacune avec sa logique :

```sql
-- (C) CRÉER : INSERT INTO table (colonnes) VALUES (valeurs)
INSERT INTO livres (titre, auteur, annee_publication, prix)
VALUES ('Le Petit Prince', 'Antoine de Saint-Exupery', 1943, 7.99)
RETURNING *;
-- RETURNING * : affiche la ligne RANGÉE (id=1 fabriqué par la séquence, cree_le remplie).
-- Utile pour voir la différence entre ce que tu envoies et ce qui est stocké.

-- (L) LIRE : SELECT colonnes FROM table [conditions/filtres]
SELECT titre, prix FROM livres WHERE prix > 5;   -- WHERE filtre les lignes
SELECT titre FROM livres ORDER BY annee_publication ASC;  -- ASC = du plus ancien au plus récent
SELECT titre FROM livres ORDER BY id LIMIT 2;    -- LIMIT 2 = maximum 2 résultats

-- (M) MODIFIER : UPDATE table SET colonne=valeur WHERE condition
UPDATE livres SET prix = 8.99 WHERE id = 1;      -- WHERE cible EXACTEMENT la ligne à modifier

-- (S) SUPPRIMER : DELETE FROM table WHERE condition
DELETE FROM livres WHERE id = 3;                 -- efface UN SEUL livre, ciblé par son id
```

### 3.3 Observer les ressources (préparation de la Leçon 3)

```sql
SELECT pg_size_pretty(pg_database_size('bibliotheque'));
-- pg_database_size : taille de la base en octets ; pg_size_pretty : format lisible (« 7952 kB »)

SHOW max_connections;            -- le nombre maximal de guichets (100 par défaut)
SELECT count(*) FROM pg_stat_activity;   -- le nombre de connexions actives en ce moment
```

### 3.4 Le filet de sécurité : les transactions

```sql
BEGIN;               -- ouvre une transaction : tout ce qui suit peut être annulé
DELETE FROM livres;  -- dangereux : pas de WHERE... mais dans la transaction
ROLLBACK;            -- annule TOUT : rien n'a été supprimé
SELECT * FROM livres;   -- preuve : tes lignes sont toujours là
-- COMMIT ferait l'inverse de ROLLBACK : il VALIDE définitivement
```

> 💡 **Réflexe pro** : toute modification risquée se fait dans une transaction — `BEGIN` → vérifier avec un `SELECT` → `COMMIT` si bon, `ROLLBACK` sinon.

---

## 4. Bonnes pratiques modernes (2025-2026)

Ce qu'il faut faire aujourd'hui quand on crée et gère une base relationnelle :

1. **Clés primaires auto-numérotées avec `GENERATED ALWAYS AS IDENTITY`** (pas l'ancienne `SERIAL`, plus propre et explicite).
2. **`NUMERIC` pour l'argent**, jamais `FLOAT`/`DOUBLE` (types approchés qui provoquent des arrondis — 0.1 + 0.2 ≠ 0.3 en flottant).
3. **`TIMESTAMPTZ` (date + heure avec fuseau)** pour tout horodatage, jamais une date « texte ».
4. **`NOT NULL` par défaut** : ne laisse une colonne vide (nullable) que si c'est **un vrai choix métier**.
5. **Une base par application et par environnement** : `bibliotheque_dev`, `bibliotheque_test`, `bibliotheque_prod` — jamais un mélange.
6. **Tout schéma versionné** : chaque création/modification de table est écrite dans un fichier versionné dans Git (automatisé par les outils de migration — **Leçon 5**).
7. **Observer avant d'ajuster** : taille, connexions, activité (`pg_stat_activity`) **avant** de modifier une configuration (Leçon 3).
8. **En production** : utiliser une base **managée** (RDS, vu au Bloc 6) quand c'est possible — le fournisseur gère sauvegardes et mises à jour pendant que tu gères le contenu.

---

## 5. Pièges à éviter

Les anti-patterns (mauvaises habitudes) classiques — chacun avec sa version correcte juste à côté.

### Piège 1 — Le `SELECT *` (tout sélectionner) partout

```sql
-- ❌ MAUVAIS : demande toutes les colonnes, sans savoir lesquelles
SELECT * FROM livres;

-- ✅ CORRECT : colonnes explicites
SELECT titre, prix FROM livres;
```

**Pourquoi c'est dangereux** : si un jour un développeur ajoute une colonne qui contient **des données sensibles** (mot de passe chiffré, token), ton `SELECT *` se met à **les renvoyer** sans que tu changes ton code. Les colonnes explicites te protègent.

### Piège 2 — `FLOAT` pour l'argent

```sql
-- ❌ MAUVAIS : les flottants (nombres approchés) additionnent des arrondis
CREATE TABLE livres (prix FLOAT);     -- 7.99 peut être stocké comme 7.9900000000000002

-- ✅ CORRECT : décimal exact
CREATE TABLE livres (prix NUMERIC(8, 2));
```

**Pourquoi** : `FLOAT` est un type **approché** (bon pour la physique, pas pour la comptabilité). Les centimes disparaissent, et personne ne trouve pourquoi. Contrepartie honnête à connaître : `NUMERIC` est plus **lent** et plus **gros** que `FLOAT` sur des millions de lignes — mais pour un prix, la justesse vaut toujours plus que la vitesse (on optimise la vitesse ailleurs : les index, Leçon 3).

### Piège 3 — Tout stocker en texte sans type

```sql
-- ❌ MAUVAIS : tout en VARCHAR, « ça marche toujours »
CREATE TABLE livres (titre VARCHAR, annee VARCHAR, prix VARCHAR);

-- ✅ CORRECT : chaque colonne a son type
CREATE TABLE livres (titre VARCHAR(200), annee_publication INT, prix NUMERIC(8, 2));
```

**Pourquoi** : avec tout en texte, le SGBD accepte `annee = 'deux mille'` et `prix = 'beaucoup'` — les erreurs n'apparaissent qu'en pleine nuit, dans le code applicatif. Le typage est un **contrôle gratuit**... et attention à la contrepartie honnête : le même contrôle peut **rejeter un INSERT légitime mal formaté** (ex. un prix arrivé en texte depuis un formulaire web). La solution n'est pas de revenir au texte, mais de **convertir à l'entrée** (côté application, Bloc 3) et de laisser le SGBD **vérifier** : chacun son travail.

### Piège 4 — Utiliser l'administrateur pour l'application

```
❌ MAUVAIS : l'application Spring Boot se connecte avec « postgres » (admin)
   → si elle est piratée, l'attaquant peut TOUT : effacer, créer, détruire.

✅ CORRECT : un rôle DÉDIÉ avec les droits minimum (Leçon 2)
   → si elle est piratée, l'attaquant ne peut que lire/écrire les tables données.
```

**Pourquoi** : c'est le principe du **moindre privilège** (rappel du Bloc 6, IAM) : donner seulement ce qui est nécessaire. C'est l'objet de la **Leçon 2** — un des piliers de tout le bloc.

### Piège 5 — Le `DELETE` (ou `UPDATE`) sans `WHERE`

```sql
-- ❌ MAUVAIS : efface TOUTES les lignes, irréversible
DELETE FROM livres;

-- ✅ CORRECT : ciblé par sa clé primaire, dans une transaction
BEGIN;
DELETE FROM livres WHERE id = 3;
SELECT * FROM livres;    -- vérifie
COMMIT;                  -- valide seulement si tout est bon
```

**Pourquoi** : c'est la faute qui a coûté une entreprise à certains (données perdues sans sauvegarde). La **Leçon 4** (backups) complète ce filet.

### Piège 6 — Choisir MongoDB « parce que c'est à la mode »

```
❌ MAUVAIS : des données RELATIONNELLES (commandes, clients) dans MongoDB
   → pas de jointures, pas de contraintes : tu recodes à la main ce que le SQL faisait tout seul.

✅ CORRECT : le bon outil pour la bonne donnée
   → données reliées entre elles = PostgreSQL ; documents flexibles isolés = MongoDB possible.
```

**Pourquoi** : la roadmap le dit bien — MongoDB est « à définir, ne pas creuser ». Ton stack actuel (Spring Boot + PostgreSQL) est cohérent.

---

## 6. Exercice pratique

> 🔁 **Comment s'articulent les fichiers** : la théorie est terminée, passons à la pratique. L'exercice complet est dans **`02-exercice.md`** (à faire **avant** de lire la correction).

En résumé, tu vas — **en local, gratuitement** :

1. créer la base `bibliotheque` et la table `livres` (avec les types corrects) ;
2. écrire les 4 opérations CRUD en SQL sur cette table ;
3. **observer** les ressources (taille de la base, `max_connections`, connexions actives) ;
4. tester le filet de sécurité `BEGIN`/`ROLLBACK` sur un `DELETE` oublié de `WHERE` (le piège 5, en sécurité).

Livrable : un fichier `notes-exercice-01.md` avec tes commandes et observations.

---

## 7. Correction détaillée de l'exercice

La correction complète (pas à pas, avec les sorties attendues et les explications des choix techniques) est dans **`03-correction.md`**. Compare chaque étape avec ta solution : la correction réécrit aussi la **checklist de validation** finale et donne des conseils.

---

## 8. Checklist de validation

- [ ] Je peux expliquer avec mes mots ce qu'est une base de données, un SGBD, une base relationnelle, une table, une ligne, une clé primaire.
- [ ] Je sais écrire le CRUD complet en SQL (INSERT / SELECT / UPDATE / DELETE), toujours avec un `WHERE` ciblé pour modifier ou supprimer.
- [ ] Je sais choisir les types corrects (`NUMERIC` pour l'argent, `TIMESTAMPTZ` pour les dates, `NOT NULL` pour l'obligatoire).
- [ ] Je sais ce que sont MongoDB et OracleDB **en une phrase chacun** (sans en faire plus, comme le demande la roadmap).
- [ ] Je navigue dans `psql` (`\l`, `\c`, `\dt`, `\d`, `\q`) sans hésiter.
- [ ] Je sais observer les ressources d'une base (taille, `max_connections`, `pg_stat_activity`).
- [ ] Je sais pourquoi une transaction (`BEGIN`/`ROLLBACK`/`COMMIT`) est un filet de sécurité indispensable.

---

> 🧭 **Prochaine étape** : dans cette leçon, tu as tout fait avec **l'administrateur** (`postgres`) — l'équivalent de tout ouvrir avec la clé du patron. En production, c'est interdit. La **Leçon 2** te fait créer des **utilisateurs avec des permissions limitées** (rôles, `GRANT`/`REVOKE`, moindre privilège) et te montre **qui a le droit de se connecter à quoi** (le fichier de règles `pg_hba.conf`). C'est la suite logique : on a vu **quoi** ranger, on va voir **qui** a le droit d'y toucher.
