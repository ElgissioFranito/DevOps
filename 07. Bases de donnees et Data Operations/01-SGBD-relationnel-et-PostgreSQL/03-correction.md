# Correction — Leçon 1 : Le SGBD relationnel et PostgreSQL

> **Bloc 7 · Leçon 1** — Correction pas à pas de `02-exercice.md`.
>
> 🔁 **Comment lire cette correction** : compare chaque étape avec ce que tu as fait. Si un résultat diffère, relis l'explication de l'étape concernée (chaque étape dit **pourquoi** on fait ce choix technique, pas seulement « comment »). La checklist de validation est réécrite à la fin, avec des conseils.

---

## Étape 0 — Vérifier PostgreSQL (prérequis)

```bash
psql --version                      # affiche la version de l'outil psql (si « command not found », PostgreSQL n'est pas installé)
sudo systemctl status postgresql --no-pager | head -5   # l'état du service ; --no-pager évite la vue interactive ; head -5 garde 5 lignes
```

Attendu : `active (running)`. Si le service est arrêté : `sudo systemctl start postgresql`.

> **Pourquoi `sudo -u postgres` ?** L'installation crée un **utilisateur du SGBD** nommé `postgres` (l'administrateur). En local, Ubuntu autorise la connexion à cet utilisateur **seulement depuis le compte système `postgres`** : c'est l'authentification « **peer** » (le nom de ton compte Linux doit correspondre au nom de l'utilisateur du SGBD). `sudo -u postgres` exécute donc `psql` **depuis le compte système `postgres`** — l'option `-u` de `sudo` signifie « exécute en tant que cet utilisateur ». (Leçon 2 expliquera les autres méthodes d'authentification, comme la saisie d'un mot de passe.)

## Étape 1 — Créer la base et la table

```bash
sudo -u postgres psql               # se connecte au SGBD en tant qu'administrateur local
```

```sql
-- \l : commande interne de psql (« list ») : liste les bases existantes
\l
-- CREATE DATABASE : crée une base ; le nom s'écrit en minuscules (par convention PostgreSQL)
CREATE DATABASE bibliotheque;
-- \c : « connect » : change de base dans la session psql
\c bibliotheque
-- \dt : « display tables » : liste les tables de la base actuelle (vide pour l'instant)
\dt
```

Puis le `CREATE TABLE` attendu :

```sql
CREATE TABLE livres (
  id                GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  titre             VARCHAR(200) NOT NULL,
  auteur            VARCHAR(120) NOT NULL,
  annee_publication INT,
  prix              NUMERIC(8, 2) NOT NULL DEFAULT 0.00,
  cree_le           TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Explication ligne par ligne (chaque choix technique a une raison) :

- **`id GENERATED ALWAYS AS IDENTITY PRIMARY KEY`** : la clé primaire est auto-numérotée par une **séquence** (le compteur interne vu dans la leçon). `GENERATED ALWAYS` signifie que le SGBD **exige** de fabriquer le numéro lui-même (tu ne peux pas forcer un `id`). C'est la bonne pratique 2025-2026 ; l'ancienne `SERIAL` fait la même chose mais avec des réglages implicites moins propres.
- **`VARCHAR(200)`** : texte court limité à 200 caractères. `NOT NULL` = **obligatoire** (refusé si vide).
- **`INT`** : nombre entier (1943, 2026…).
- **`NUMERIC(8, 2)`** : décimal **exact** avec 8 chiffres dont 2 après la virgule — le type qu'on doit toujours utiliser pour l'**argent** (voir « Pièges à éviter » de la leçon). `DEFAULT 0.00` = valeur par défaut si tu ne la fournis pas.
- **`TIMESTAMPTZ`** (« timestamp with time zone ») : date + heure **avec fuseau horaire**. `DEFAULT now()` = remplie automatiquement avec l'heure actuelle.

Vérification :

```sql
\dt           -- attendu : « public | livres | table | postgres » (la table existe)
\d livres     -- « describe » : détaille les colonnes, types, défauts
```

## Étape 2 — Le CRUD (attendu)

### C — Créer

```sql
INSERT INTO livres (titre, auteur, annee_publication, prix)
VALUES ('Le Petit Prince', 'Antoine de Saint-Exupery', 1943, 7.99)
RETURNING *;   -- RETURNING * : affiche la ligne rangée (id auto, cree_le rempli par now())

INSERT INTO livres (titre, auteur, annee_publication, prix)
VALUES ('Clean Code', 'Robert C. Martin', 2008, 25.90);

INSERT INTO livres (titre, auteur, annee_publication)   -- prix omis : le DEFAULT 0.00 s'applique
VALUES ('Le Proces', 'Franz Kafka', 1925);
```

**Attendu avec `RETURNING *`** : 3 lignes avec `id` = 1, 2, 3 et `cree_le` remplie avec la date/heure actuelle. **Pourquoi `RETURNING *` est utile** : il prouve ce que PostgreSQL a **réellement** rangé (id fabriqué par la séquence, valeur par défaut appliquée) — le débutant voit ainsi la différence entre ce qu'il **envoie** et ce qui est **rangé**.

### L — Lire

```sql
-- 1. Les livres de plus de 5 €
SELECT titre, prix FROM livres WHERE prix > 5;
-- Attendu : Le Petit Prince (7.99) et Clean Code (25.90) — Le Proces (0.00) est filtré

-- 2. Triés par année, du plus ancien au plus récent
SELECT titre, annee_publication FROM livres ORDER BY annee_publication ASC;
-- Attendu : Le Proces (1925), Le Petit Prince (1943), Clean Code (2008) — ASC = ascendant

-- 3. Maximum 2 résultats
SELECT titre FROM livres ORDER BY id LIMIT 2;
-- Attendu : Le Petit Prince, Clean Code — LIMIT = plafonne le nombre de lignes
```

### M — Modifier

```sql
UPDATE livres SET prix = 8.99 WHERE id = 1;   -- WHERE cible exactement la ligne (id = 1)
SELECT titre, prix FROM livres WHERE id = 1;  -- vérification (réflexe : vérifier après écrire)
```

### S — Supprimer

```sql
DELETE FROM livres WHERE id = 3;              -- efface UN SEUL livre, par son id
SELECT * FROM livres;                          -- preuve : il en reste 2
```

## Étape 3 — Observer les ressources (attendu)

```sql
SELECT pg_size_pretty(pg_database_size('bibliotheque'));
-- Attendu : quelque chose comme « 8192 kB » ou « 7952 kB » — une base vide pèse quelques kilo-octets (les kilo-octets
-- affichés couvrent le catalogue interne du SGBD, pas tes 2 lignes). L'important : savoir LIRE ce chiffre.

SHOW max_connections;
-- Attendu : 100 (valeur par défaut d'une installation standard). 100 = 100 « guichets » de connexion disponibles.

SELECT count(*) FROM pg_stat_activity;
-- Attendu : un petit nombre (souvent 2-4) : ta session psql + les processus internes du SGBD.
```

**Pourquoi cette étape** : c'est le fondement de la Leçon 3 (configuration). Tu ne configures pas « à l'aveugle » : tu **observes d'abord** (taille, guichets, activité), **puis** tu ajustes. Et un piège de lecture à connaître : sur une machine fraîchement démarrée, `pg_stat_activity` compte les processus **internes** du SGBD (écrivain de fond, vérificateur...) en plus de ta session — un « 3 ou 4 » affiché ne veut PAS dire que 3 personnes sont connectées.

## Étape 4 — Le test de sécurité (attendu)

```sql
BEGIN;               -- ouvre une transaction (filet de sécurité)
DELETE FROM livres;  -- sans WHERE : PostgreSQL accepte et efface TOUT... mais dans la transaction
ROLLBACK;            -- annule tout : aucune suppression appliquée
SELECT * FROM livres;  -- preuve : tes 2 lignes sont toujours là ✓
```

**Ce que tu devrais noter dans `notes-exercice-01.md`** : *« Sans `BEGIN`/`ROLLBACK`, un `DELETE FROM livres;` sans `WHERE` efface **toutes** les lignes, de façon **irréversible**. La transaction est le filet : `BEGIN` ouvre, `COMMIT` valide, `ROLLBACK` annule. Le réflexe pro est de modifier risqué = `BEGIN` → vérifier → `COMMIT`. »*

> 💡 **Pourquoi PostgreSQL autorise un `DELETE` sans `WHERE` ?** C'est voulu : « effacer tout » est parfois légitime (vider une table de test). Le SGBD te fait confiance — c'est précisément pourquoi le **DevOps doit avoir ses propres filets** (transactions, sauvegardes de la Leçon 4, rôles restreints de la Leçon 2).

## ✅ Checklist de validation (réécrite)

À la fin de cet exercice, tu dois pouvoir cocher :

- [ ] Je me connecte au SGBD avec `sudo -u postgres psql` et je peux expliquer l'authentification « peer ».
- [ ] J'ai créé la base `bibliotheque` et la table `livres` (avec `GENERATED ALWAYS AS IDENTITY`, pas `SERIAL`).
- [ ] Je peux expliquer chaque type de la table (`VARCHAR`, `INT`, `NUMERIC`, `TIMESTAMPTZ`) et pourquoi `NUMERIC` pour l'argent.
- [ ] J'ai fait les 4 opérations CRUD en SQL, avec `RETURNING *` pour « créer » et `WHERE` pour « modifier/supprimer ».
- [ ] Je sais filtrer (`WHERE`), trier (`ORDER BY`), plafonner (`LIMIT`).
- [ ] J'ai lu la taille de la base, `max_connections` et `pg_stat_activity` (fondement de la Leçon 3).
- [ ] J'ai testé `BEGIN`/`ROLLBACK` et je peux expliquer pourquoi un `DELETE` sans `WHERE` est dangereux.

## 💡 Conseils

- **Réfléchis à voix haute** : « je cible exactement la ligne avec son `id` » — ce réflexe sauve des données.
- **Vérifie après chaque écriture** (`UPDATE`, `DELETE`) avec un `SELECT` : c'est gratuit et ça évite les dégâts.
- **Ce que tu as fait « à la main » aujourd'hui** (créer table, écrire SQL), les **outils de migration** l'automatiseront en Leçon 5 — mais tu ne peux automatiser que ce que tu sais faire à la main.
- **Prochaine étape logique** : aujourd'hui tu as tout fait avec **l'administrateur** (`postgres`) — dangereux en pratique. La **Leçon 2** t'apprend à créer des **utilisateurs restreints** avec des permissions limitées (le principe du moindre privilège).

---

> 🎉 **Fin de la correction de la Leçon 1.** Si tout est coché, tu es prêt(e) pour la Leçon 2 — elle part de cette question : *« qui a le droit de se connecter à ma base, et avec quels droits ? »*
