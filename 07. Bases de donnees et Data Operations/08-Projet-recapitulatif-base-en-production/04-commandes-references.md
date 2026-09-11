# Commandes & références 8 — Le gabarit du runbook (à remplir)

> Copie ce fichier en `runbook-bibliotheque.md` et remplis **chaque** point par ce que tu as **vécu** dans le projet. Un point vide = un devoir, pas un livrable.

## § 1. Architecture (le diagramme + pourquoi chaque brique)

```
                    Internet
                       ↓
              Load Balancer (Bloc 5/6)
                       ↓
        Application Spring Boot (le rôle app_biblio)
          ↓                        ↓
    cache Redis (les réponses        → PostgreSQL PRIMARY (guichet 5432)
    fréquentes — post-it)                 ├── backup pg_dump chaque nuit (cron)
                                          ├── archivage WAL (le carnet → PITR)
                                          ├── migrations Flyway versionnées dans Git
                                          └── RÉPLICA (le sous-chef — lectures/bascule)
```

Écris sous le diagramme : **pourquoi chaque brique** (une ligne par brique — la question de validation).

## § 2. Qui accède à quoi (Leçon 2)

```sql
-- Rôles (prouvés par \du : AUCUN pouvoir d'administration)
CREATE ROLE app_biblio LOGIN PASSWORD 'fort_et_unique' NOSUPERUSER NOCREATEDB NOCREATEROLE;
CREATE ROLE lecteur_biblio LOGIN PASSWORD 'fort_et_unique_2' NOSUPERUSER NOCREATEDB NOCREATEROLE;

-- Droits (prouvés par les tests) :
-- app_biblio : CONNECT + USAGE + SELECT/INSERT/UPDATE/DELETE (+ USAGE sur SÉQUENCES)
-- lecteur_biblio : CONNECT + USAGE + SELECT seul
-- REVOKE ... FROM PUBLIC (la porte large fermée)
```

Tests de preuve (à noter) : `INSERT` ✅ / `CREATE TABLE pirate` ❌ en app · `SELECT` ✅ / `DELETE` ❌ en lecteur.

## § 3. L'observation (Leçon 3)

```sql
ALTER SYSTEM SET log_min_duration_statement = '500ms';   -- le comptable des lenteurs
SELECT pg_reload_conf();
SELECT round(100.0 * blks_hit / nullif(blks_hit + blks_read, 0), 1) AS taux_cache_pct
FROM pg_stat_database WHERE datname = 'bibliotheque_prod';   -- seuil : < 95 % = alerte
SELECT usename, application_name, count(*) FROM pg_stat_activity GROUP BY 1, 2;  -- seuil : 80 % du plafond
EXPLAIN ANALYZE SELECT ... ;   -- avant/après index (les index créés + leur gain mesuré)
```

```bash
sudo journalctl -u postgresql --no-pager --since "5 minutes ago" | grep -i duration
```

## § 4. La sauvegarde (Leçon 4)

```bash
sudo -u postgres pg_dump -Fc -d bibliotheque_prod -f "backups/bibliotheque_prod-$(date +%F).dump"
sudo -u postgres pg_dumpall --globals-only -f "backups/globals-$(date +%F).sql"
```

Fréquence : **chaque nuit à 02h30** (cron du compte postgres). La règle 3-2-1 : 3 copies (originale + 2 dumps), 2 supports (disque + S3 — Bloc 6), 1 hors site (S3 chiffré). RPO visé : ___ (avec unité).

## § 5. La restauration — la procédure PAS À PAS (testée)

```bash
# 1. À CÔTÉ d'abord (jamais d'abord en production)
sudo -u postgres psql -c 'CREATE DATABASE bibliotheque_restaure;'
sudo -u postgres pg_restore -d bibliotheque_restaure "backups/bibliotheque_prod-$(date +%F).dump"
# 2. VÉRIFIER par les comptages
sudo -u postgres psql -d bibliotheque_restaure -c 'SELECT count(*) FROM membres;'
# 3. REMETTRE en service
sudo -u postgres psql -c 'DROP DATABASE bibliotheque_prod;'
sudo -u postgres psql -c 'CREATE DATABASE bibliotheque_prod;'
sudo -u postgres pg_restore -d bibliotheque_prod "backups/bibliotheque_prod-$(date +%F).dump"
# 4. Ré-appliquer GRANT CONNECT (script versionné, Leçon 2) — les ACL des tables reviennent AVEC le dump
# 5. VÉRIFICATION APPLICATIVE : tester avec app_biblio ET lecteur_biblio (pas l'admin !)
```

**Date du prochain test de restauration mensuel** : ___ (écrite, pas espérée).

## § 6. Les migrations (Leçon 5)

```bash
# Le RITUEL (non négociable) :
sudo -u postgres pg_dump -Fc -d bibliotheque_prod -f "backups/avant-Vn-$(date +%F).dump"
flyway -url=jdbc:postgresql://localhost:5432/bibliotheque_prod -user=postgres validate
flyway -url=... -user=... migrate
flyway -url=... -user=... info
```

Règles : numéros uniques (`Vn__description_minuscules.sql`), **jamais éditer** une migration appliquée (checksum), **expand → remplir → contract** pour le NOT NULL, forward fix (`V(n+1)`), GRANT embarqués.

## § 7. Le plan de reprise (Leçon 6)

```
RTO : ___ (avec unité — ce service) | RPO : ___ (avec unité — ce service)
Configuration : ___ (backup+PITR / async / sync+auto — justifiée en 3 lignes)
Étapes : détecter (___) → élire/promouvoir (___) → rediriger (___) → vérifier (humain) → re-cloner (humain)
Date du prochain test de bascule : ___ (trimestriel)
Rappel anti-erreur-humaine : la réplication copie AUSSI les erreurs — le PITR (§ 4-5) reste le filet.
```

## § 8. Le cache (Leçon 7)

```
CACHÉ : la liste des populaires → TTL 5 min + invalidation au UPDATE
        les agrégats            → TTL 1 h
        les sessions            → TTL = la durée de session
JAMAIS CACHÉ : le statut lu après validation (critique à la seconde)
Règle d'or : try/except — Redis tombé = comme un miss (l'application CONTINUE)
Surveillance : redis-cli INFO stats → keyspace_hits / keyspace_misses (le taux de hit)
```

## § 9. Incidents narrés (vécus, pas imaginés)

**Incident 1 — « trop de clients »** : symptôme (`FATAL: sorry, too many clients`) → diagnostic (`pg_stat_activity` vs `max_connections`) → correction (pool 20-50 connexions, pas `max_connections = 10000`).

**Incident 2 — « plus rien ne s'écrit »** : symptôme (disque plein, Leçon 3) → diagnostic (`df -h`) → correction (purge/étendre, surveiller — pas un paramètre).

## La réponse de 5 lignes (à réciter)

> Les couches : (1) refus par défaut — seuls les rôles nécessaires écrivent (Leçon 2) ; (2) WAL archivé — survivre au crash, PITR à la seconde (Leçon 4) ; (3) backup testé — 2 dumps, restauration réelle, 3-2-1 (Leçon 4) ; (4) réplication — bascule en minutes (Leçon 6). Aucune ne suffit seule : c'est leur **empilement** qui supprime la perte.

## Liens croisés du bloc

| Besoin | Où |
|---|---|
| Rôles et droits | `02-Utilisateurs-roles-permissions-et-connexions/04-commandes-references.md` |
| Observation (taux de cache, journal) | `03-Configuration-et-ressources/04-commandes-references.md` |
| Dumps, PITR, cron | `04-Backup-et-restauration/04-commandes-references.md` |
| Flyway CLI, expand/contract | `05-Migrations-de-schema-et-donnees/04-commandes-references.md` |
| Diagnostic de réplication, RTO/RPO | `06-Replication-et-haute-disponibilite/04-commandes-references.md` |
| redis-cli, Spring Cache, grille TTL | `07-Cache-applicatif-Redis/04-commandes-references.md` |