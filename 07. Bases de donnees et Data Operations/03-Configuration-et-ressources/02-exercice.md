# Exercice — Leçon 3 : Configuration et ressources

> **Bloc 7 · Leçon 3** — Exercice en autonomie, **100 % en local** (PostgreSQL du Bloc 2). Aucun coût.
>
> 🔁 **Comment s'articulent les fichiers** : lis d'abord `01-lecon.md` (théorie + vocabulaire), fais cet exercice **sans** regarder la solution, puis compare avec `03-correction.md`. Aide-mémoire : `04-commandes-references.md`.

---

## Contexte

L'analyste (Leçon 2) se plaint : *« la recherche des livres par année devient lente »*. Personne ne sait si c'est la base, la machine ou l'application. Ton travail : **observer**, **activer le bon journal**, **mesurer**, et **documenter** — sans rien casser.

> ⚠️ **Prérequis** : la base `bibliotheque` existe avec sa table `livres` et les rôles de la Leçon 2.

---

## Énoncé

### Étape 1 — Photo initiale (l'état des lieux, AVANT tout changement)

Dans une session admin (`sudo -u postgres psql`), note dans `notes-exercice-03.md` :

1. Les valeurs actuelles des paramètres clés **et leur `context`** (faut-il restart ou reload ?) :
   ```sql
   SELECT name, setting, context FROM pg_settings
   WHERE name IN ('max_connections', 'shared_buffers', 'work_mem', 'log_min_duration_statement');
   ```
2. Le **taux de cache** de la base (le pourcentage de lectures servies par la RAM) :
   ```sql
   SELECT round(100.0 * blks_hit / nullif(blks_hit + blks_read, 0), 1) AS taux_cache_pct
   FROM pg_stat_database WHERE datname = 'bibliotheque';
   ```
3. Le nombre de connexions actives : `SELECT count(*) FROM pg_stat_activity;`

### Étape 2 — Activer l'observateur (le journal des requêtes lentes)

1. Active la journalisation des requêtes de plus de **200 ms** avec `ALTER SYSTEM SET` (pas d'édition de fichier).
2. Recharge la configuration **sans couper le service** (`pg_reload_conf()`).
3. **Vérifie** que la valeur est bien prise en compte (`SHOW log_min_duration_statement;`).

### Étape 3 — Provoquer une requête lente (volontairement, pour test)

```sql
SELECT count(*) FROM generate_series(1, 5000000) g WHERE g % 3 = 0;
-- generate_series(1, 5000000) : fabrique 5 millions de nombres (une fonction « série »)
-- g % 3 = 0 : le modulo (« reste de la division par 3 ») garde 1 nombre sur 3 — le SGBD doit tout parcourir
```

Puis lis le journal (la sortie des messages du SGBD) depuis le **service systemd** (pas de chemin à deviner) :

```bash
sudo journalctl -u postgresql --no-pager --since "5 minutes ago" | grep -i duration
# journalctl : lit les messages du service ; -u postgresql : cible CE service ;
# --no-pager : tout affiche d'un coup (pas de vue interactive) ; --since "5 minutes ago" : fenêtre de temps ;
# grep -i duration : garde les lignes contenant « duration » (-i = ignore majuscules/minuscules)
```

### Étape 4 — Mesurer puis agir : `EXPLAIN ANALYZE`

1. Exécute et note le **plan d'exécution** (la « recette » que suit le SGBD) :
   ```sql
   EXPLAIN ANALYZE SELECT titre FROM livres WHERE annee_publication = 1943;
   -- EXPLAIN : montre le plan SANS exécuter ; ANALYZE : exécute et affiche le temps RÉEL de chaque étape
   ```
   Repère le mot **`Seq Scan`** (« lecture séquentielle » : le SGBD lit **toutes** les lignes, comme feuilleter tout le classeur).
2. Crée un **index** (le « sommaire du classeur » qui évite de tout feuilleter) et **re-mesure** :
   ```sql
   CREATE INDEX idx_livres_annee ON livres (annee_publication);  -- sommaire sur la colonne année
   EXPLAIN ANALYZE SELECT titre FROM livres WHERE annee_publication = 1943;   -- re-mesure : quoi de neuf ?
   ```
   > 💡 Cette étape illustre la boucle complète : **mesurer → agir → re-mesurer**. Un index est une structure interne du SGBD (un sommaire trié) qui accélère les recherches — au prix d'un peu d'espace disque et d'écriture.

### Étape 5 — Réfléchir (dans `notes-exercice-03.md`)

1. Pourquoi monter `max_connections` à 10 000 serait une **fausse bonne idée** ?
2. Pourquoi copier la « config parfaite d'Internet » sans mesurer est **dangereux** ?
3. Laquelle de ces 3 ressources **ne se règle pas** dans `postgresql.conf` : la mémoire cache, le nombre de connexions, **l'espace disque restant** ? Et qui doit surveiller celle-là ?

---

## Livrable

**`notes-exercice-03.md`** : la photo initiale (paramètres + context + taux de cache), la commande `ALTER SYSTEM` + `SHOW` de vérification, la ligne du journal avec le temps de la requête lente, les **deux plans** `EXPLAIN ANALYZE` (avant/après index, avec le temps), et les 3 réponses de l'étape 5.

Correction détaillée dans **`03-correction.md`**. Aide-mémoire : **`04-commandes-references.md`**.
