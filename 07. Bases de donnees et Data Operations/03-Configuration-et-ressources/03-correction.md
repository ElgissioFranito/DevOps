# Correction — Leçon 3 : Configuration et ressources

> **Bloc 7 · Leçon 3** — Correction pas à pas de `02-exercice.md`.
>
> 🔁 **Comment lire cette correction** : compare chaque étape avec ce que tu as fait ; chaque étape explique **pourquoi** (pas seulement « comment »). Checklist réécrite + conseils à la fin.

---

## Étape 1 — Photo initiale (attendu)

```sql
SELECT name, setting, unit, context FROM pg_settings
WHERE name IN ('max_connections', 'shared_buffers', 'work_mem', 'log_min_duration_statement');
```

Sortie attendue (une installation standard) :

```
           name            | setting | unit |  context
---------------------------+---------+------+------------
 log_min_duration_statement| -1      | ms   | superuser
 max_connections           | 100     |      | postmaster
 shared_buffers            | 16384   | 8kB  | postmaster
 work_mem                  | 4096    | kB   | user
```

**Lecture guidée** (c'est le cœur de l'exercice) :

- `shared_buffers = 16384` avec l'unité `8kB` → 16384 × 8 = **128 Mo** : le réglage par défaut très prudent confirmé.
- `context = postmaster` (sur `shared_buffers`, `max_connections`) → **restart requis** : toute modification coupera les connexions — à planifier.
- `work_mem` en `user` → réglable **par session** avec `SET work_mem = '16MB';` (aucun impact pour les autres).
- `log_min_duration_statement = -1` → **désactivé** : le « comptable des lenteurs » n'est pas encore embauché. (Selon la version, son `context` peut être `sighup` ou `superuser` — dans les deux cas, **un reload suffit**, pas de restart.)

```sql
SELECT round(100.0 * blks_hit / nullif(blks_hit + blks_read, 0), 1) AS taux_cache_pct
FROM pg_stat_database WHERE datname = 'bibliotheque';
-- Attendu : 99.x ou 100.0 sur une petite base qui vient d'être lue — le garde-manger tient dans le plan de travail
```

> 💡 **Pourquoi noter AVANT de changer ?** Sans la photo initiale, tu ne pourras pas prouver demain que ton réglage a servi — ou qu'il a fait du mal. C'est le même réflexe qu'en Git : un commit **avant** chaque changement.

## Étape 2 — Activer l'observateur (attendu)

```sql
ALTER SYSTEM SET log_min_duration_statement = '200ms';
SELECT pg_reload_conf();                      -- reload sans coupure (aucune session interrompue)
SHOW log_min_duration_statement;              -- attendu : 200ms  → le réglage EST appliqué
```

**Pourquoi `ALTER SYSTEM` et pas l'édition du fichier** : c'est le chemin le plus court et sans erreur de syntaxe (le fichier, un mauvais caractère et le service ne redémarre plus). **Mais** : le réglage durable doit être **reporté dans un fichier versionné dans Git** — sinon ta configuration vit seulement dans la mémoire du serveur, ce qui contredit la reproductibilité (Bloc 8).

## Étape 3 — La requête lente et sa preuve (attendu)

```sql
SELECT count(*) FROM generate_series(1, 5000000) g WHERE g % 3 = 0;
-- Attendu : « 1666667 » (les multiples de 3), en 1 à 2 secondes — la boucle de 5 millions prend du temps
```

```bash
sudo journalctl -u postgresql --no-pager --since "5 minutes ago" | grep -i duration
```

Ligne attendue (le SGBD a noté la requête parce qu'elle a dépassé 200 ms) :

```
... duration: 1350.123 ms  statement: SELECT count(*) FROM generate_series(1, 5000000) g WHERE g % 3 = 0;
```

**Pourquoi passer par le journal plutôt que par l'écran** : dans la vraie vie, ce n'est pas **toi** qui exécutes la requête lente — c'est l'application, à 3 heures du matin. Le journal est le seul témoin qui dort jamais. Si ta commande `journalctl` ne montre rien : vérifie que la requête a bien pris **plus** de 200 ms (relis le temps affiché par `psql`) et que tu as bien fait `pg_reload_conf()` avant.

## Étape 4 — Mesurer puis agir : EXPLAIN ANALYZE (attendu)

**Avant l'index** (table minuscule, la lecture de tout est déjà rapide) :

```
                        QUERY PLAN
-----------------------------------------------------------
 Aggregate  (actual time=0.036..0.037 rows=1 loops=1)
   ->  Seq Scan on livres  (actual time=0.014..0.020 rows=1 loops=1)
         Filter: (annee_publication = 1943)
         Rows Removed by Filter: 4      ← il a lu TOUTES les lignes et en a écarté 4
 Planning Time: 0.12 ms
 Execution Time: 0.05 ms
```

**Le point pédagogique** : sur une petite table, le `Seq Scan` est **normal et rapide**. Ne crée pas des index partout « pour faire pro » : un index coûte de l'espace disque et ralentit les écritures (chaque `INSERT` doit aussi mettre à jour le sommaire). **Le critère, c'est la mesure sur des tables volumineuses.**

**Après `CREATE INDEX idx_livres_annee ON livres (annee_publication);`** : le plan peut passer à `Bitmap Index Scan`/`Index Scan`. Sur 5 lignes, le gain sera invisible — c'est **une bonne leçon** : l'outil te dit si l'action valait le coup. (Avec des dizaines de milliers de lignes, la différence devient spectaculaire — c'est l'exercice « pour aller plus loin » de l'aide-mémoire.)

**Pourquoi `EXPLAIN ANALYZE` et pas `EXPLAIN` seul** : `EXPLAIN` seul **estime** le plan sans l'exécuter ; `ANALYZE` **exécute** et affiche les temps réels (`actual time`, `rows`). En production sur une requête d'écriture, préférer `EXPLAIN` seul (sans `ANALYZE`) pour ne pas exécuter deux fois un `UPDATE` risqué.

## Étape 5 — Les questions de réflexion (attendu)

**1. Pourquoi `max_connections = 10000` est une fausse bonne idée ?**
Chaque connexion = **un processus** du SGBD + sa mémoire. À des milliers de connexions, la machine sature de processus et de mémoire **avant** d'avoir servi qui que ce soit. Le vrai problème est presque toujours des connexions **tenues trop longtemps** (une application qui ouvre et garde des guichets) — d'où le **pool** (HikariCP/PgBouncer), qui sert des milliers d'utilisateurs avec quelques dizaines de connexions.

**2. Pourquoi la « config parfaite d'Internet » est dangereuse ?**
Parce qu'elle a été **mesurée sur une autre machine, avec une autre charge, d'autres disques, d'autres données**. Les paramètres de mémoire (`shared_buffers`, `work_mem`) sont de la **comptabilité de RAM** : si leur somme dépasse ta machine réelle, le SGBD s'étrangle ou refuse de démarrer. La seule config parfaite est celle qu'on a **mesurée chez soi** — photo initiale, un changement, re-mesure.

**3. Quelle ressource ne se règle pas dans `postgresql.conf` ?**
**L'espace disque.** On peut régler la mémoire (cache, tris), les guichets (connexions), les journaux — mais le disque restant est une ressource **physique** : soit on purge (données inutiles, anciens journaux, sauvegardes déplacées), soit on **étend** (disque plus grand), et surtout on **surveille** en continu — c'est un sujet d'observabilité (Bloc 12), pas de configuration.

## ✅ Checklist de validation (réécrite)

- [ ] Je peux expliquer pourquoi les défauts sont **prudents** (la selle du vélo) et pourquoi on adapte à SA machine.
- [ ] Je trouve `postgresql.conf` **dynamiquement** (`SHOW config_file;` — jamais un chemin deviné) et je connais les 5 paramètres clés.
- [ ] Je change un paramètre avec `ALTER SYSTEM`, je recharge **sans coupure** (`pg_reload_conf()`) et je **vérifie** avec `SHOW`.
- [ ] Je lis la colonne **`context`** de `pg_settings` et je dis si un changement demande **reload** (sans coupure) ou **restart** (fenêtre de maintenance).
- [ ] Je fais la **photo initiale** avant de régler (paramètres, taux de cache, connexions) et je **re-mesure** après.
- [ ] Je diagnostique les 3 symptômes : **« too many clients »** (guichets → pool), **lenteurs** (EXPLAIN ANALYZE, cache), **disque plein** (pas un paramètre — purge/étendre/surveiller).
- [ ] Je connais les points de départ honnêtes de mémoire (≈ 25 % pour `shared_buffers` sur une machine **dédiée**, moins si l'application tourne dessus).
- [ ] Je peux expliquer le **pool de connexions** (taxi collectif) et pourquoi il bat un `max_connections` géant.

## 💡 Conseils

- **Ta boucle personnelle** : photo initiale → un changement → reload si possible → re-mesure → **note tout**. Cette boucle, tu la réutiliseras pour chaque incident de ta vie DevOps.
- **Le journal des lenteurs est ton premier instrument de production** : active-le dès le premier jour d'une vraie base (seuil 500 ms pour commencer) — tu découvriras des requêtes que personne ne soupçonnait.
- **Attention aux stats fraîches** : les tables `pg_stat_*` comptent depuis le **dernier démarrage** — les lire une fois par an ne veut rien dire ; les lire **régulièrement** (Bloc 12) fait l'observabilité.
- **Ce que cette leçon prépare** : une base réglée produit des métriques qu'on centralisera (Bloc 12) et une config qu'on automatisera (Bloc 8, IaC). Mais **aucune optimisation ne protège d'un disque qui meurt** — d'où la Leçon 4, le cœur du bloc : **backup et restauration**.

---

> 🎉 **Fin de la correction de la Leçon 3.** Si tout est coché, tu sais garder une base **rapide et prévisible** — et tu es prêt(e) pour la leçon la plus importante du bloc : la **sauvegarde** et la **restauration**.
