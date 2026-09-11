# Aide-mémoire — Leçon 3 : configuration et ressources

> **Bloc 7 · Leçon 3** — Table des commandes à garder à côté. Tout est local et gratuit.

---

## 1. Trouver la configuration (sans deviner de chemin)

```bash
CONF="$(sudo -u postgres psql -tAc 'SHOW config_file;')"   # chemin du fichier, affiché par le SGBD (-tAc : sortie brute + commande)
sudo nano "$CONF"        # éditer (réglage durable → à versionner dans Git)
```

```sql
SHOW config_file;        -- idem, depuis psql (affiche le chemin)
SHOW hba_file;           -- rappel Leçon 2 : le fichier des règles de connexion
SHOW data_directory;     -- le dossier des DONNÉES du SGBD (à ne JAMAIS toucher à la main)
```

## 2. Changer un paramètre proprement

```sql
-- Voie d'exploitation (rapide, sans éditer de fichier) :
ALTER SYSTEM SET log_min_duration_statement = '500ms';   -- écrit dans postgresql.auto.conf (prime sur le fichier)
SELECT pg_reload_conf();                                  -- reload sans coupure
SHOW log_min_duration_statement;                          -- VÉRIFIER (toujours)

-- Réglage durable : éditer postgresql.conf (et VERSIONNER le changement dans Git)
-- Par session (context 'user') :
SET work_mem = '32MB';        -- n'affecte QUE cette session
RESET work_mem;               -- revenir au réglage général
```

```bash
sudo systemctl reload postgresql     # reload en ligne de commande (sans coupure)
sudo systemctl restart postgresql    # restart (COUPE tout) — seulement pour les params « postmaster », en fenêtre de maintenance
```

## 3. Les paramètres clés

| Paramètre | Rôle | Analogie | Défaut prudent | Context |
|---|---|---|---|---|
| `max_connections` | guichets de connexion | tables du restaurant | 100 | `postmaster` (restart) |
| `shared_buffers` | cache interne en RAM | plan de travail central | 128 Mo | `postmaster` (restart) |
| `work_mem` | RAM par tri/requête | plan de travail de chaque cuisinier | 4 Mo | `user` (par session) |
| `effective_cache_size` | estimation du cache (pour le planificateur) | ce que le chef croit disponible | 4 Go | `postmaster` |
| `log_min_duration_statement` | seuil de journalisation des requêtes lentes | le comptable des lenteurs | -1 (off) | `sighup`/`superuser` (reload) |

```sql
-- Lire la valeur + le type de changement requis (reload ou restart ?)
SELECT name, setting, unit, context FROM pg_settings
WHERE name IN ('max_connections','shared_buffers','work_mem','effective_cache_size','log_min_duration_statement');
```

## 4. Observer la santé

```sql
-- Taux de cache : > 99 % = bon ; < 95 % = le cache étouffe
SELECT round(100.0 * blks_hit / nullif(blks_hit + blks_read, 0), 1) AS taux_cache_pct
FROM pg_stat_database WHERE datname = 'bibliotheque';

-- Connexions actives par utilisateur/application (comparer à max_connections)
SELECT usename, application_name, count(*) FROM pg_stat_activity GROUP BY usename, application_name;

-- Taille des tables (\dt+ fait la même chose en plus court)
SELECT relname, pg_size_pretty(pg_total_relation_size(relid)) FROM pg_catalog.pg_statio_user_tables ORDER BY pg_total_relation_size(relid) DESC;

-- Plan d'exécution (EXPLAIN = estimé ; ANALYZE = exécute et mesure)
EXPLAIN ANALYZE SELECT titre FROM livres WHERE annee_publication = 1943;
-- Seq Scan = lit toutes les lignes ; Index Scan/Bitmap Index Scan = passe par le sommaire
```

```bash
# Le journal des requêtes lentes (via systemd — aucun chemin à deviner)
sudo journalctl -u postgresql --no-pager --since "5 minutes ago" | grep -i duration
# -u : cible le service ; --no-pager : tout d'un coup ; --since : fenêtre de temps ; grep -i : filtre insensible à la casse
```

## 5. La boucle DevOps de la base (à retenir par cœur)

```
PHOTO INITIALE (pg_settings + taux de cache + connexions)
        ↓
UN changement (ALTER SYSTEM SET ... / CREATE INDEX ...)
        ↓
RELOAD si possible (pg_reload_conf() — vérifier le context)
        ↓
RE-MESURE (SHOW + EXPLAIN ANALYZE + journal)
        ↓
NOTE (notes-exercice-XX.md / fichier versionné Git)
```

## 6. Pour aller plus loin (optionnel)

```sql
-- Un vrai test d'index : créer une table de 100 000 lignes, mesurer avant/après
CREATE TABLE mesures AS SELECT g AS id, (g % 10000) AS cle
FROM generate_series(1, 100000) g;                 -- 100 000 lignes avec une colonne « cle »
EXPLAIN ANALYZE SELECT * FROM mesures WHERE cle = 42;          -- Seq Scan + temps T
CREATE INDEX idx_mesures_cle ON mesures (cle);
EXPLAIN ANALYZE SELECT * FROM mesures WHERE cle = 42;          -- Index Scan + temps T' — compare T et T'
```
