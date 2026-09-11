# Aide-mémoire — Leçon 1 : psql et SQL de base

> **Bloc 7 · Leçon 1** — Table des commandes à garder à côté. Toutes sont **locales et gratuites** (PostgreSQL du Bloc 2).

---

## 1. psql : les commandes internes (backslash + lettre)

| Commande | À quoi elle sert |
|---|---|
| `psql --version` | Version de l'outil (dans le shell) |
| `sudo -u postgres psql` | Se connecter en tant qu'administrateur local (`-u postgres` = compte système `postgres`) |
| `\l` | Lister les bases |
| `\l+` | Lister les bases **avec leur taille** |
| `\c ma_base` | Se connecter à une base |
| `\dt` | Lister les tables de la base actuelle |
| `\dt+` | Idem, **avec la taille et le nombre de lignes estimé** |
| `\d ma_table` | Détailler une table (colonnes, types, clé primaire) |
| `\du` | Lister les rôles/utilisateurs (détail en **Leçon 2**) |
| `\h CREATE TABLE` | Afficher l'aide SQL d'une commande (remplace `CREATE TABLE` par ce que tu veux) |
| `\?` | Lister toutes les commandes internes de psql |
| `\q` | Quitter psql |
| `\password un_role` | Changer un mot de passe **sans le taper dans l'historique** (détail en Leçon 2) |

## 2. SQL : le CRUD (les 4 opérations de base)

```sql
-- CRÉER une table
CREATE TABLE livres (
  id                GENERATED ALWAYS AS IDENTITY PRIMARY KEY,  -- auto-numéroté + unique
  titre             VARCHAR(200) NOT NULL,                     -- texte court, obligatoire
  annee_publication INT,                                        -- entier
  prix              NUMERIC(8, 2) NOT NULL DEFAULT 0.00,       -- décimal exact (argent)
  cree_le           TIMESTAMPTZ NOT NULL DEFAULT now()         -- date+heure avec fuseau
);

-- CRÉER une ligne
INSERT INTO livres (titre, annee_publication, prix)
VALUES ('Le Petit Prince', 1943, 7.99)     -- pas de id : la séquence le fabrique
RETURNING *;                                -- affiche la ligne réellement rangée

-- LIRE des lignes
SELECT titre, prix FROM livres                      -- colonnes explicites (jamais * en pratique)
WHERE prix > 5                                      -- filtre
ORDER BY annee_publication ASC                      -- tri (ASC = du plus ancien au plus récent)
LIMIT 2;                                            -- maximum 2 résultats

-- MODIFIER une ligne
UPDATE livres SET prix = 8.99 WHERE id = 1;         -- WHERE = la condition qui cible la ligne

-- SUPPRIMER une ligne
DELETE FROM livres WHERE id = 2;                    -- JAMAIS de DELETE/UPDATE sans WHERE

-- EFFACER une table entière (à manier avec précaution)
DROP TABLE livres;                                  -- supprime table + données (irréversible)
```

## 3. Types courants (et leur usage)

| Type | Usage | Exemple |
|---|---|---|
| `VARCHAR(n)` | Texte court limité | `email VARCHAR(200)` |
| `TEXT` | Texte long sans limite | `resume TEXT` |
| `INT` / `BIGINT` | Entier / grand entier | `annee INT` |
| `NUMERIC(8,2)` | Décimal **exact** (argent !) | `prix NUMERIC(8,2)` |
| `BOOLEAN` | Vrai/faux | `actif BOOLEAN` |
| `DATE` / `TIMESTAMPTZ` | Date / date+heure avec fuseau | `cree_le TIMESTAMPTZ` |
| `UUID` | Identifiant aléatoire universel | `token UUID` |

## 4. Observer les ressources

```sql
-- Taille d'une base (pg_database_size = taille en octets ; pg_size_pretty = format lisible)
SELECT pg_size_pretty(pg_database_size('bibliotheque'));

-- Nombre maximal de connexions (le « nombre de guichets »)
SHOW max_connections;

-- Connexions actives en ce moment (pg_stat_activity = la table « surveillance » du SGBD)
SELECT count(*) FROM pg_stat_activity;

-- Qui est connecté, à quoi ?
SELECT usename, application_name, state FROM pg_stat_activity;
```

## 5. Transactions : le filet de sécurité

```sql
BEGIN;               -- ouvre une transaction : tout ce qui suit peut être annulé
DELETE FROM livres;  -- danger : pas de WHERE
ROLLBACK;            -- annule tout, aucune modification appliquée
COMMIT;              -- (au contraire) valide tout ce qui a été fait depuis BEGIN
```

> 💡 **Réflexe pro** : pour toute modification risquée (`UPDATE`/`DELETE` en masse), ouvre un `BEGIN`, vérifie avec un `SELECT`, puis `COMMIT` si tout est bon — ou `ROLLBACK` sinon.
