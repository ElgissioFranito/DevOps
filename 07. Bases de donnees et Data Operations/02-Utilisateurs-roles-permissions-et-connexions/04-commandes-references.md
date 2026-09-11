# Aide-mémoire — Leçon 2 : rôles, permissions et connexions

> **Bloc 7 · Leçon 2** — Table des commandes à garder à côté. Tout est local et gratuit.

---

## 1. Rôles et utilisateurs

```sql
CREATE ROLE mon_role LOGIN PASSWORD 'phrase-de-passe'
  NOSUPERUSER NOCREATEDB NOCREATEROLE;      -- utilisateur applicatif type
CREATE USER mon_user PASSWORD '...';        -- abréviation de CREATE ROLE ... LOGIN
ALTER ROLE app_biblio WITH PASSWORD 'nouvelle';  -- change le mot de passe (ou \password app_biblio dans psql)
ALTER ROLE app_biblio RENAME TO app_biblio_2026; -- renomme
DROP ROLE app_biblio;                       -- supprime (doit d'abord ne plus être propriétaire d'objets)
GRANT mon_groupe TO app_biblio;             -- ajoute le rôle à un groupe
\du                                         -- liste les rôles + attributs
\du+                                        -- idem + la description / membres
```

## 2. Permissions (GRANT / REVOKE)

```sql
-- ACCORDER (grâce = « accorde <quoi> ON <objet> TO <qui> »)
GRANT CONNECT ON DATABASE bibliotheque TO app_biblio;       -- entrer dans la base
GRANT USAGE ON SCHEMA public TO app_biblio;                 -- circuler dans le schéma
GRANT SELECT ON ALL TABLES IN SCHEMA public TO lecteur;     -- lecture (lecture seule)
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app;  -- CRUD
GRANT USAGE ON ALL SEQUENCES IN SCHEMA public TO app;       -- les compteurs auto (nécessaires à INSERT)
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO app;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT USAGE ON SEQUENCES TO app;   -- pour les objets FUTURS
GRANT lecteur TO mon_role;                                  -- hériter des droits d'un groupe

-- RETIRER
REVOKE DELETE ON ALL TABLES IN SCHEMA public FROM app;      -- retirer un droit précis
REVOKE CONNECT ON DATABASE bibliotheque FROM PUBLIC;        -- fermer « tout le monde »
REVOKE CREATE ON SCHEMA public FROM PUBLIC;                 -- idem (créer des objets)

-- VÉRIFIER LES DROITS EFFECTIFS
SELECT * FROM information_schema.role_table_grants WHERE grantee = 'app_biblio';
```

## 3. `pg_hba.conf` : les 5 colonnes d'une règle

```
TYPE   BASE        UTILISATEUR   ADRESSE          MÉTHODE
local  all         postgres      (sans adresse)   peer            # local sans réseau, compte Linux = rôle
host   all         all           127.0.0.1/32     scram-sha-256   # réseau, mot de passe
host   bibliotheque  app_biblio  192.168.1.50/32  scram-sha-256   # règle PRÉCISE (à mettre EN HAUT)
```

```bash
SHOW hba_file;                      # (dans psql) le chemin exact du fichier — ne le devine pas
HBA="$(sudo -u postgres psql -tAc 'SHOW hba_file;')"   # (dans le shell) relit le chemin affiché (-tAc = sortie brute + commande SQL)
sudo nano "$HBA"                    # ouvre le fichier affiché
sudo systemctl reload postgresql    # RELIT la config sans couper les connexions (reload ≠ restart)
```

| Méthode | Définition | Usage |
|---|---|---|
| `peer` | compte Linux = rôle demandé, sans mot de passe | local uniquement |
| `scram-sha-256` | mot de passe exigé, jamais stocké en clair | la méthode moderne (défaut depuis PG 14) |
| `md5` | l'ancienne méthode mot de passe (faible) | à remplacer |
| `trust` | AUCUN mot de passe | jamais, hors test isolé |

## 4. Connexions

```bash
psql -h localhost -p 5432 -U app_biblio -d bibliotheque
# -h l'adresse du serveur ; -p le port ; -U l'utilisateur ; -d la base

psql "host=adresse port=5432 dbname=bibliotheque user=app sslmode=require"
# sslmode=require : chiffrement TLS du trajet OBLIGATOIRE (production) — rappel TLS : Bloc 5, Leçon 4

sudo -u postgres psql       # admin local (peer)
\q                          # quitte psql
```

```sql
SELECT usename, client_addr, ssl FROM pg_stat_ssl JOIN pg_stat_activity USING (pid);
-- pg_stat_ssl : liste les connexions et si elles sont chiffrées (ssl = true/false)
```

## 5. Secrets de l'application (Spring Boot)

```properties
# src/main/resources/application.properties — les NOMS seulement
spring.datasource.url=jdbc:postgresql://localhost:5432/bibliotheque
spring.datasource.username=${DB_USER}
spring.datasource.password=${DB_PASSWORD}
```

```bash
export DB_USER=app_biblio                  # valeurs fournies au démarrage (hors code, hors Git)
export DB_PASSWORD='...'
./mvnw spring-boot:run                     # l'application lit les variables au démarrage
# En production : les variables viennent d'un coffre-fort (AWS Secrets Manager — rappel Bloc 6, IAM)
```
