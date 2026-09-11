# Correction 8 — La base `bibliotheque_prod` en production (runbook type)

> Compare **ton runbook** à celui-ci. Il n'y a pas un seul bon runbook — il y a des runbooks **complets, chiffrés et testés**, et des runbooks qui ne le sont pas. La grille de la fin te dit où tu en es.
>
> ⚠️ **Note sur la cohérence du fil rouge** : aux Leçons 1 à 4, tu as travaillé dans la base `bibliotheque` (dev). Le projet crée ici une base dédiée **`bibliotheque_prod`** (la « prod ») : c'est la même logique que `bibliotheque_dev` / `bibliotheque_test` / `bibliotheque_prod` de la Leçon 1 (une base par environnement). Le `CREATE TABLE livres` ci-dessous n'a donc **pas besoin** de `IF NOT EXISTS` — la base est neuve et vide.

## Étape 1 — Le socle (attendu)

```sql
CREATE DATABASE bibliotheque_prod;

CREATE TABLE livres (
  id                GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  titre             VARCHAR(200) NOT NULL,
  auteur            VARCHAR(120) NOT NULL,
  annee_publication INT,
  prix              NUMERIC(8, 2) NOT NULL DEFAULT 0.00,
  cree_le           TIMESTAMPTZ NOT NULL DEFAULT now()
);
-- Les autres tables (membres, emprunts) : voir Leçons 1 et 5 — FOREIGN KEY + NUMERIC + TIMESTAMPTZ

CREATE ROLE app_biblio LOGIN PASSWORD 'fort_et_unique'
  NOSUPERUSER NOCREATEDB NOCREATEROLE;
CREATE ROLE lecteur_biblio LOGIN PASSWORD 'fort_et_unique_2';

GRANT CONNECT ON DATABASE bibliotheque_prod TO app_biblio, lecteur_biblio;
-- puis dans la base : USAGE sur public, SELECT/INSERT/UPDATE/DELETE à app_biblio,
-- SELECT seul + USAGE sur SÉQUENCES à app_biblio, SELECT seul à lecteur_biblio
-- et REVOKE ... FROM PUBLIC (Leçon 2) — le refus par défaut fermé
```

**Les tests de preuve** (dans le runbook § 2) :

```sql
-- en app_biblio : INSERT et SELECT ✅ ; CREATE TABLE pirate ❌ « permission denied for schema public »
-- en lecteur_biblio : SELECT ✅ ; DELETE ❌ « permission denied for table livres »
```

## Étape 2 — L'observation (attendu)

```sql
ALTER SYSTEM SET log_min_duration_statement = '500ms';
SELECT pg_reload_conf();                       -- reload, pas restart (sighup)
SHOW log_min_duration_statement;               -- 500ms ✓

SELECT round(100.0 * blks_hit / nullif(blks_hit + blks_read, 0), 1) AS taux_cache_pct
FROM pg_stat_database WHERE datname = 'bibliotheque_prod';   -- viser > 99 %

EXPLAIN ANALYZE SELECT ... WHERE annee_publication = 1943;   -- Seq Scan → CREATE INDEX → re-mesure
```

**Dans le runbook § 3** — les 3 requêtes de santé + seuils :

| Requête | Seuil d'alerte | Pourquoi |
|---|---|---|
| Taux de cache (`pg_stat_database`) | < 95 % | le plan de travail (RAM) étouffe |
| Connexions (`pg_stat_activity`) vs `max_connections` | > 80 du plafond | les guichets s'épuisent → pool (Leçon 3) |
| Journal des lenteurs (`journalctl -u postgresql`) | > 500 ms | le comptable signale une requête coûteuse → EXPLAIN |

## Étape 3 — Le filet (attendu)

```bash
sudo -u postgres pg_dump -Fc -d bibliotheque_prod -f "backups/bibliotheque_prod-$(date +%F).dump"
sudo -u postgres pg_dumpall --globals-only -f "backups/globals-$(date +%F).sql"
```

Puis la panne vécue et la remise en service (Leçon 4) :

```bash
sudo -u postgres psql -c 'CREATE DATABASE bibliotheque_restaure;'
sudo -u postgres pg_restore -d bibliotheque_restaure "backups/bibliotheque_prod-$(date +%F).dump"
# → comptages ✅ (la preuve) → DROP DATABASE bibliotheque_prod → CREATE → pg_restore
# → ré-appliquer GRANT CONNECT (script versionné, Leçon 2) → TESTER app_biblio / lecteur_biblio ✅
```

**La phrase que le runbook § 5 doit contenir** : *« restaurer à côté d'abord, compter pour vérifier, puis remettre en service — et vérifier avec les rôles clients, pas l'admin. »*

## Étape 4 — L'évolution (attendu)

```bash
sudo -u postgres pg_dump -Fc -d bibliotheque_prod -f "backups/avant-V1-$(date +%F).dump"
# Le RITUEL d'avant-migration (Leçon 5) — le backup d'abord, non négociable
```

```sql
-- db/migration/V1__ajoute_date_retour_prevue.sql
ALTER TABLE emprunts ADD COLUMN date_retour_prevue DATE;      -- NULLABLE d'abord (l'expand)
GRANT SELECT, INSERT, UPDATE ON ALL TABLES IN SCHEMA public TO app_biblio;
-- Puis V2 (le contract, si besoin) : UPDATE de remplissage → SET NOT NULL
```

```bash
flyway -url=jdbc:postgresql://localhost:5432/bibliotheque_prod -user=postgres info
flyway -url=... validate      # checksums ✅
```

## Étape 5 — La panne du serveur (réponse attendue)

Pour **ce** projet (bibliothèque municipale, catalogue consulté le jour, écritures modérées) :

```
RTO : 4 h (une demi-journée sans le catalogue est vivable)
RPO : ≈ 0 (aucune écriture perdue — archivage WAL continu + backup nocturne)
Configuration : backup nocturne + PITR (Leçon 4 + scénario 2 de la Leçon 6)
Justification : RPO ≈ 0 obtenu par le PITR sans payer le synchrone ;
                RTO de 4 h rendu possible en restaurant sur une machine prête ;
                budget minimal (pas de 2e serveur) — la réplication viendrait si le métier exigeait des minutes.
```

**Dans le runbook § 7** : RTO/RPO avec unités, étapes de bascule (détecter → restaurer → vérifier → rediriger), responsable de chaque étape, et **la date du prochain test**.

## Étape 6 — La performance (attendu)

- Redis installé, `SET cache:livres_populaires ... EX 300` (TTL 5 min).
- Stale data **vécu** : `UPDATE` en base pendant que le cache vit → réponse périmée → correction par `DEL` (invalidation à côté de l'écriture).
- Règle d'or **prouvée** : Redis arrêté → l'application **continue** (try/except → miss → base répond).
- **Dans le runbook § 8** : la grille par donnée (quoi / TTL / invalidation) + « jamais de cache pour le critique à la seconde ».

## Étape 7 — Le runbook (la grille d'auto-évaluation)

| Critère | C'est réussi si... |
|---|---|
| Complétude | Les 9 points du gabarit sont remplis (§ 1-9) — aucun « à compléter plus tard » |
| Architecture justifiée | Chaque brique a sa **raison** (la question du bloc 6) — le diagramme se lit seul |
| Rôles (Leçon 2) | `app_biblio` et `lecteur_biblio` créés **sans pouvoirs d'administration**, preuves testées et notées |
| Chiffres (Leçons 3, 6, 7) | RTO/RPO avec unités, TTL par donnée, prix du miss, seuils d'alerte — **tous** écrits |
| Panne vécue (Leçon 4) | `DROP TABLE membres` → silence → restauration à côté → comptages → remise en service → **vérification applicative** |
| Migration (Leçon 5) | Fichier `Vn__...` versionné, ritual de backup fait, `info` + `validate` documentés |
| Planification | Les dates des **prochains tests** (restauration mensuelle, bascule trimestrielle) sont **écrites** |
| Les 2 incidents narrés | « Trop de clients » et « plus rien ne s'écrit » : symptôme → diagnostic → correction, tirés de ce que tu as vécu |
| Versionnement | Le runbook + les scripts sont dans Git (Bloc 4) |
| La réponse de 5 lignes | « Comment éviter une perte de données » : les 4 COUCHES (Leçon 2, WAL, backup testé, réplication) + la phrase « aucune couche ne suffit seule » |

## ✅ Checklist de validation (LE critère du bloc — réécrite)

- [ ] Mon runbook de `bibliotheque_prod` existe en 9 points et **un collègue pourrait le suivre sans moi**.
- [ ] Je peux **mettre une base PostgreSQL en production** : rôles (Leçon 2), observation (Leçon 3), sauvegardes (Leçon 4).
- [ ] Je peux **la sauvegarder et la restaurer** : 2 dumps + restauration à côté + comptages + remise en service + vérification applicative.
- [ ] J'ai **effectué une migration** versionnée avec le rituel de backup d'avant-migration (Leçon 5).
- [ ] Je peux **expliquer comment éviter une perte de données** en 5 lignes : le refus par défaut, le WAL, le backup testé, la réplication — et pourquoi **aucune ne suffit seule**.
- [ ] Mes choix sont **chiffrés** (RTO/RPO, TTL, seuils) et mes tests **planifiés** (mensuel, trimestriel).

## 💡 Conseils

- **Le runbook est un document de travail, pas un devoir** : relis-le dans 1 mois — tout ce qui te semble flou sera flou à 3 h du matin. Corrige-le maintenant.
- **Le test mensuel de restauration est une habitude, pas une corvée** : mets-la dans ton calendrier avec une alarme (`cron` t'envoie même le rapport — Bloc 2).
- **Garde ce projet** : c'est ta **preuve DevOps** — la réponse concrète à « montre-moi ce que tu sais faire avec une base de données ».
- **Le pont vers le Bloc 8** : chaque procédure du runbook deviendra une **ressource Terraform** (la base RDS, les accès, les sauvegardes automatiques). Tu connais déjà le **quoi** — l'IaC t'apprendra le **comment en code**.

---

> 🎉 **Fin de la correction de la Leçon 8 — et du Bloc 7.** Si tout est coché, tu valides la roadmap : *« mettre une base PostgreSQL en production, la sauvegarder, la restaurer, effectuer une migration et expliquer comment éviter une perte de données »* — par un runbook vécu, pas par des notes de cours.
