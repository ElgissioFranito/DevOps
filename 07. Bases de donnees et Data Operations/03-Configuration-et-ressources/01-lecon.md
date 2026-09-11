# Leçon 3 — Configuration et ressources du SGBD

> **Bloc 7 · Bases de données & Data Operations** — Leçon 3 sur 8
> 🧭 **Pont depuis la Leçon 2** : ta base est maintenant **bien gardée** (rôles, permissions, connexions maîtrisées). Mais « bien gardée » ne veut pas dire « bien réglée » : une base peut être sécurisée **et** lente, saturée, muette (sans journaux). La roadmap demande ici : *« configuration, ressources »*. Cette leçon t'apprend la boucle de travail du DevOps sur une base : **observer → régler → vérifier** — et s'appuie directement sur l'observation commencée en Leçon 1 (taille, `max_connections`, `pg_stat_activity`).

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Expliquer** pourquoi les réglages par défaut sont **prudents** (pensés pour marcher partout) et pourquoi il faut les **adapter** à ta machine et à ta charge.
2. **Trouver** le fichier de configuration `postgresql.conf` **sans deviner** son chemin, et connaître les **paramètres clés** (mémoire, connexions, journaux).
3. **Modifier** la configuration proprement avec `ALTER SYSTEM` et la recharger (`reload`) sans couper le service.
4. **Observer** la santé de la base : taux de cache, connexions, requêtes lentes (`EXPLAIN ANALYZE`, journal des requêtes).
5. **Diagnostiquer** les 3 symptômes classiques d'une base qui « suffoque » (disque plein, guichets épuisés, cache trop petit).

---

## 2. Explication simple

### 2.1 Le « pourquoi » : les défauts sont prudents, pas optimaux

Quand tu as installé PostgreSQL (Bloc 2), le SGBD est arrivé avec des réglages **par défaut**. Ils sont volontairement **prudents** : conçus pour fonctionner sur **n'importe quelle machine**, du petit portable au gros serveur. Conséquence : ils sont presque toujours **trop petits** (et parfois mal placés) pour une charge réelle.

```
Défauts d'usine           →  « ça tourne partout »   →  mais lent sur TA machine
Machine + charge réelles  →  réglages adaptés        →  rapide, observable, prévisible
```

> 💡 **Analogie** : un vélo neuf a la selle **au milieu** : ça roule pour tout le monde, mais personne n'est à l'aise. Tu ne changes pas le vélo : tu ajustes la selle **à ta taille**. La configuration, c'est la selle du SGBD.

> 📌 **Limite franche de l'analogie** (pour ne pas être piégé) : sur un vélo, un mauvais réglage fait « juste » mal au dos. Sur un SGBD, un mauvais réglage peut **empêcher le démarrage du service** (`shared_buffers` plus grand que la RAM, par exemple — voir le Piège 2). D'où la boucle de la leçon : **observer → régler UN paramètre → vérifier** — jamais tout d'un coup, jamais sans mesure.

**Le « pourquoi » du métier DevOps** : « l'application est lente » est un signalement, pas un diagnostic. Ton travail est de **mesurer** (Leçon 1 t'a donné les outils), **ajuster** un paramètre, puis **revérifier** — jamais de deviner.

### 2.2 Le « comment » (partie 1) : `postgresql.conf` et `ALTER SYSTEM`

La configuration vit dans le fichier **`postgresql.conf`**. Règle d'or de cette leçon (héritée du skill de ce projet) : **ne devine jamais son chemin** — demande-le au SGBD lui-même :

```bash
CONF="$(sudo -u postgres psql -tAc 'SHOW config_file;')"
# CONF : variable shell = le chemin du fichier de configuration, AFFICHÉ par le SGBD lui-même.
# Options de psql : -t (sortie brute, sans cadre) ; -A (sans alignement) ; -c (exécute la commande SQL entre guillemets).
sudo nano "$CONF"        # ouvre le fichier affiché (nano : Ctrl+O sauve, Ctrl+X quitte — Bloc 2)
```

Deux façons modernes de régler un paramètre :

```
Façon 1 : éditer postgresql.conf à la main   → bien pour du réglage DURABLE et versionné (Git, Bloc 8)
Façon 2 : ALTER SYSTEM SET ...               → écrit dans postgresql.auto.conf, qui PRIME sur le fichier de base
```

Les paramètres clés à connaître (avec les analogies du restaurant de la Leçon 1) :

| Paramètre | À quoi il sert | Analogie restaurant | Réglage prudent par défaut |
|---|---|---|---|
| `max_connections` | le nombre maximal de **guichets** de connexion | les tables du restaurant | 100 |
| `shared_buffers` | la RAM que le SGBD garde pour son **cache interne** | le plan de travail central | 128 Mo (très petit) |
| `work_mem` | la RAM allouée **à chaque tri/requête** | le plan de travail de chaque cuisinier | 4 Mo |
| `log_min_duration_statement` | **journalise** toute requête plus lente que X (ex. `500ms`) | le comptable qui note les commandes trop longues | désactivé (`-1`) |
| `effective_cache_size` | information donnée au **planificateur** (qui choisit comment lire les données) | ce que le chef **croit** disponible | 4 Go |

Et la question que tout débutant se pose : *« si je change un paramètre, faut-il couper le service ? »* PostgreSQL répond dans la colonne **`context`** de la table `pg_settings` :

- **`postmaster`** → il faut un **restart** (le SGBD redémarre : coupure !) — ex. `shared_buffers` ;
- **`sighup`** → un simple **reload** suffit (relit la configuration sans couper personne) — ex. les paramètres de journaux ;
- **`user`** → réglable **par session**, avec `SET work_mem = '32MB';` (n'affecte que ta session).

### 2.3 Le « comment » (partie 2) : observer la santé de la base

Théorie du réglage posée — passons à la **mesure**, la partie que tu feras le plus souvent. Quatre instruments :

**1. Le taux de cache** (le pourcentage de lectures servies par la **RAM** au lieu du disque) :

```sql
SELECT round(100.0 * blks_hit / nullif(blks_hit + blks_read, 0), 1) AS taux_cache_pct
FROM pg_stat_database WHERE datname = 'bibliotheque';
-- blks_hit : blocs (pages de données) servis DEPUIS le cache mémoire
-- blks_read : blocs lus SUR LE DISQUE (lents)
-- nullif(..., 0) : évite la division par zéro sur une base neuve (aucune lecture)
```

> 💡 **Analogie** : le cuisinier (le SGBD) sert-il depuis son plan de travail (`blks_hit`) ou doit-il courir au garde-manger (`blks_read`) ? **Au-dessus de 99 %** : il a tout sous la main. **Sous 95 %** : il court toute la journée — le plan de travail (`shared_buffers`) est trop petit.

**2. Les connexions actives** (`pg_stat_activity`, vu en Leçon 1) : compte-les et compare-les à `max_connections`. Proche du plafond = les tables du restaurant sont toutes occupées.

**3. Le plan d'exécution** (`EXPLAIN ANALYZE`) : le SGBD te montre **comment** il exécute ta requête :

```sql
EXPLAIN ANALYZE SELECT titre FROM livres WHERE annee_publication = 1943;
-- Seq Scan = « lecture séquentielle » : il lit TOUTES les lignes (feuilleter tout le classeur)
-- Index Scan = il saute DIRECTEMENT aux bonnes lignes grâce à un sommaire (un index)
-- ANALYZE ajoute le temps RÉEL mesuré de chaque étape
```

Un **index** est ce « sommaire trié » interne ; le créer (`CREATE INDEX ... ON table (colonne)`) coûte un peu de disque et d'écriture, mais évite de feuilleter le classeur entier à chaque recherche.

**4. Le journal des requêtes lentes** (`log_min_duration_statement`) : une fois activé, toute requête plus lente que le seuil est **écrite dans le journal du service** — c'est ta preuve mesurée, pas une impression.

### 2.4 Le « quand » : les 3 symptômes d'une base qui suffoque

| Symptôme (ce que tu observes) | Cause probable | Premier réflexe |
|---|---|---|
| `FATAL: sorry, too many clients already` (l'application n'obtient pas de guichet) | connexions épuisées | compter `pg_stat_activity` ; chercher QUI tient des connexions ; réfléchir à un **pool** de connexions (voir « Bonnes pratiques ») |
| Requêtes de plus en plus lentes, taux de cache bas | cache trop petit ou requêtes qui lisent tout | `EXPLAIN ANALYZE` ; regarder les `Seq Scan` sur grandes tables |
| Plus rien ne s'écrit, erreurs « No space left on device » | **disque plein** | ce n'est pas un paramètre : purger/étendre le disque, surveiller (Bloc 12) |

**Le « quand » régler ?** Toujours dans cet ordre : **observer d'abord** (les 4 instruments), **changer un seul paramètre à la fois**, **reload avant restart** quand c'est possible, **re-mesurer après**. Une config qu'on ne mesure pas, c'est une devinette avec des effets de bord.

---

## 📖 Vocabulaire / Abréviations

- **postgresql.conf** : le fichier de configuration du SGBD (chemin obtenu avec `SHOW config_file;` — jamais deviné).
- **postgresql.auto.conf** : fichier écrit par `ALTER SYSTEM`, qui **prime** sur `postgresql.conf`.
- **ALTER SYSTEM** : commande SQL qui change un paramètre durablement (écrit dans `postgresql.auto.conf`).
- **Paramètre** : un réglage du SGBD (`max_connections`, `shared_buffers`, `work_mem`…).
- **`max_connections`** : le nombre maximal de connexions simultanées (les guichets).
- **`shared_buffers`** : la RAM réservée au **cache interne** du SGBD (le plan de travail central).
- **`work_mem`** : la RAM allouée **par tri/requête** (le plan de travail de chaque cuisinier).
- **`effective_cache_size`** : une **estimation** du cache total, utilisée par le planificateur pour choisir un plan.
- **`log_min_duration_statement`** : le seuil au-delà duquel une requête est **journalisée** (le comptable des lenteurs).
- **`pg_settings`** : la table qui liste tous les paramètres, leur valeur et leur **`context`**.
- **`context`** : le type de changement requis (`postmaster` = restart, `sighup` = reload, `user` = par session).
- **Reload / restart** : relire la configuration sans couper / couper et relancer le service (Leçon 2).
- **`pg_stat_database`** : table de statistiques par base (lectures cache/disque, transactions…).
- **`blks_hit` / `blks_read`** : blocs servis depuis la RAM / lus sur le disque.
- **Taux de cache** : `blks_hit / (blks_hit + blks_read)` — l'indicateur du plan de travail.
- **Plan d'exécution** : la « recette » choisie par le SGBD pour exécuter une requête.
- **`EXPLAIN` / `EXPLAIN ANALYZE`** : montrer le plan / l'exécuter avec les temps réels.
- **Seq Scan** (« lecture séquentielle ») : lire toutes les lignes de la table.
- **Index Scan** : passer par un index (un sommaire) pour atteindre directement les lignes.
- **Index** : structure interne triée qui accélère les recherches (au prix de disque et d'écriture).
- **`pg_stat_activity`** : la table des connexions actives (Leçon 1).
- **`generate_series`** : fonction qui fabrique une série de nombres (utile pour les tests).
- **Modulo (`%`)** : le reste d'une division (`7 % 3 = 1`).
- **Pool de connexions** : un « distribueur » qui réutilise un petit nombre de connexions pour beaucoup d'utilisateurs.
- **journalctl** : l'outil systemd (Bloc 2) pour lire les messages d'un service.
- **systemd** : le gestionnaire de services de Linux (vu au Bloc 2).
- **nullif(a, b)** : fonction SQL qui renvoie `NULL` si `a = b` (utile pour éviter une division par zéro).

---

## 3. Exemples concrets

> 🔁 On enchaîne : le vocabulaire est posé ; voici **les commandes exactes**, commentées ligne par ligne. Tout est local et gratuit.

### 3.1 Faire la photo initiale (avant tout changement)

```bash
sudo -u postgres psql       # session administrateur (peer)
```

```sql
-- La table pg_settings liste TOUS les paramètres, leur valeur et leur « context »
SELECT name, setting, unit, context FROM pg_settings
WHERE name IN ('max_connections', 'shared_buffers', 'work_mem',
               'effective_cache_size', 'log_min_duration_statement');
-- setting = la valeur actuelle ; unit = l'unité (8kB, MB...) ;
-- context = postmaster (restart) / sighup (reload) / user (par session)

-- Le taux de cache (partie servie par la RAM)
SELECT round(100.0 * blks_hit / nullif(blks_hit + blks_read, 0), 1) AS taux_cache_pct
FROM pg_stat_database WHERE datname = 'bibliotheque';

-- Les connexions actives, par utilisateur
SELECT usename, application_name, count(*)
FROM pg_stat_activity GROUP BY usename, application_name;
-- GROUP BY : regroupe les lignes pour compter par utilisateur et par application
```

### 3.2 Régler proprement avec `ALTER SYSTEM` (sans éditer de fichier)

```sql
-- Journalise toute requête de plus de 200 millisecondes
ALTER SYSTEM SET log_min_duration_statement = '200ms';
-- ALTER SYSTEM : écrit le réglage dans postgresql.auto.conf (qui prime sur postgresql.conf)

-- Relit la configuration SANS couper les connexions (un reload, pas un restart)
SELECT pg_reload_conf();
-- (l'équivalent en ligne de commande : sudo systemctl reload postgresql — vu en Leçon 2)

-- VÉRIFIE que la valeur est bien prise en compte (le réflexe « re-mesurer »)
SHOW log_min_duration_statement;      -- attendu : 200ms
```

> 💡 **Où vit `postgresql.auto.conf` ?** Dans le même dossier que `postgresql.conf` (`SHOW config_file;` te l'affiche). **Pourquoi préférer l'édition du fichier (versionnée dans Git) pour un réglage durable ?** Parce qu'un fichier versionné est **reproductible** : une autre machine (ou une migration IaC — Bloc 8) le rejoue à l'identique. `ALTER SYSTEM` est parfait pour un réglage d'**exploitation** rapide, à reporter ensuite dans le fichier versionné.

### 3.3 Mesurer une requête lente (et en avoir la preuve)

```sql
-- Une requête VOLONTAIREMENT lente (elle parcourt 5 millions de nombres)
SELECT count(*) FROM generate_series(1, 5000000) g WHERE g % 3 = 0;
-- generate_series : fabrique une série de nombres ; % (modulo) : reste de la division
-- → le SGBD doit TOUT parcourir : la requête prend une à deux secondes
```

Puis la preuve dans le journal du service (lecture via systemd — pas de chemin à deviner) :

```bash
sudo journalctl -u postgresql --no-pager --since "5 minutes ago" | grep -i duration
# journalctl : lit les messages du service ; -u postgresql : cible CE service ;
# --no-pager : affiche tout d'un coup (pas de vue interactive) ; --since "5 minutes ago" : fenêtre de temps ;
# | : envoie la sortie à la commande suivante (les « pipes », vus au Bloc 2) ;
# grep -i duration : garde les lignes contenant « duration » (-i = majuscules ou minuscules)
```

Sortie attendue (le format varie un peu selon la version) :

```
duration: 1350.123 ms  statement: SELECT count(*) FROM generate_series(1, 5000000) g WHERE g % 3 = 0;
```

### 3.4 Mesurer puis agir : le plan d'exécution et l'index

```sql
EXPLAIN ANALYZE SELECT titre FROM livres WHERE annee_publication = 1943;
-- Attendu (petite table) : « Seq Scan on livres » + un temps très court — normal, la table est minuscule

CREATE INDEX idx_livres_annee ON livres (annee_publication);
-- CREATE INDEX : construit le « sommaire trié » sur la colonne annee_publication

EXPLAIN ANALYZE SELECT titre FROM livres WHERE annee_publication = 1943;
-- Re-mesure : selon le volume, le plan peut passer à « Bitmap/Index Scan » — tu as MESURÉ le gain, pas deviné
```

> 💡 **Pourquoi ce ping-pong mesure→action→mesure ?** C'est la boucle DevOps de base. Un index sur une petite table peut ne **rien** changer (lire 10 lignes, c'est vite fait) — seul `EXPLAIN ANALYZE` te dit si l'action valait le coup. Leçon retenue : **les outils disent la vérité, les impressions ne la disent pas**.

### 3.5 Régler la mémoire : les points de départ honnêtes

```
shared_buffers       ≈ 25 % de la RAM de la machine DÉDIÉE à la base   (règle de départ, à mesurer ensuite)
effective_cache_size ≈ 50-75 % de la RAM                                (une ESTIMATION donnée au planificateur)
work_mem             ≈ 8-32 Mo pour commencer                           (attention : par tri, et il peut y avoir plusieurs tris par requête)
```

> ⚠️ Ces chiffres sont des **points de départ**, pas des vérités : sur une machine où tourne **aussi** l'application (ton cas en local), ils doivent être **plus petits**. Et `shared_buffers` demande un **restart** (context `postmaster`) : à ne faire que dans une fenêtre de maintenance — la coupure se planifie (Bloc 2 : fenêtres de maintenance des services).

### 3.6 Le pool de connexions (l'alternative intelligente à `max_connections` géant)

```
Sans pool :  5 000 visiteurs → 5 000 connexions → dépasse max_connections → « too many clients »
Avec pool  :  5 000 visiteurs → [pool de 20 connexions réutilisées] → la base respire
```

> 💡 **Analogie** : un **pool**, c'est un taxi collectif : plutôt qu'une voiture par passager, quelques voitures qui font des allers-retours. En pratique : côté application (le pool interne de Spring Boot — `HikariCP`, son outil par défaut) ou côté infrastructure (PgBouncer, un « distribueur » spécialisé). On le **mentionne** ici ; la mise en place vient avec les conteneurs (Bloc 9+).

---

## 4. Bonnes pratiques modernes (2025-2026)

1. **Observer avant de régler** : photo initiale (`pg_settings`, taux de cache, connexions) **avant** le premier `ALTER SYSTEM`. Un réglage sans mesure, c'est une devinette.
2. **Un paramètre à la fois** : si tu changes cinq réglages et que ça va plus vite (ou moins vite), tu ne sauras jamais lequel.
3. **Reload avant restart** : la colonne `context` de `pg_settings` te dit ce qui est possible (`sighup` = reload, sans coupure). Le **restart** se planifie en fenêtre de maintenance.
4. **Versionner la configuration** : le réglage durable vit dans un fichier versionné dans Git (`postgresql.conf` ou script) — `ALTER SYSTEM` pour l'urgence d'exploitation, puis report. La reproductibilité complète viendra avec l'IaC (**Bloc 8**).
5. **Journaliser les lenteurs** : `log_min_duration_statement` actif en permanence (seuil raisonnable, ex. 500 ms en production) — c'est le premier indicateur gratuit de santé applicative.
6. **Superviser continuellement** : le taux de cache et les connexions se surveillent **dans le temps**, pas une fois (l'observabilité complète viendra au **Bloc 12**).
7. **Préférer le pool de connexions à un `max_connections` géant** : chaque connexion coûte de la mémoire et un guichet ; un pool (HikariCP côté Spring Boot, PgBouncer côté infra) sert des milliers d'utilisateurs avec quelques dizaines de connexions.
8. **Sur une base managée (RDS, Bloc 6)** : les paramètres se règlent dans l'**interface du fournisseur** (groupes de paramètres) — la logique de cette leçon s'applique à l'identique, seul l'endroit change.

---

## 5. Pièges à éviter

### Piège 1 — Monter `max_connections` à 10 000 « pour ne plus être bloqué »

```
❌ MAUVAIS : max_connections = 10000
   → chaque connexion = un processus + de la mémoire : la machine s'effondre AVANT d'atteindre 10 000.

✅ CORRECT : mesurer qui tient les connexions + un pool (20-50 connexions réutilisées)
   → des milliers d'utilisateurs servis par quelques dizaines de guichets.
```

**Pourquoi** : les guichets coûtent cher même vides. Le problème n'est presque jamais « pas assez de guichets », mais « des guichets tenus trop longtemps ».

### Piège 2 — Copier la « config parfaite d'Internet » sans mesurer

```sql
-- ❌ MAUVAIS : coller un shared_buffers = 8GB trouvé sur un forum, sur ta machine de 4 Go
ALTER SYSTEM SET shared_buffers = '8GB';      -- le SGBD peut refuser de démarrer... ou manger toute la RAM

-- ✅ CORRECT : mesurer la machine, partir prudent, mesurer l'effet
SELECT setting, unit FROM pg_settings WHERE name = 'shared_buffers';   -- d'abord LIRE
ALTER SYSTEM SET shared_buffers = '512MB';                             -- puis AJUSTER à ta taille
```

**Pourquoi** : une config est **personnelle à une machine et à une charge**. Le forum parle d'un serveur de 128 Go dédié — pas du tien.

### Piège 3 — `restart` pour tout, « puisque ça applique sûr »

```bash
# ❌ MAUVAIS : couper tout le monde pour un réglage qui n'en avait pas besoin
sudo systemctl restart postgresql

# ✅ CORRECT : vérifier le context du paramètre, puis recharger sans coupure
SELECT name, context FROM pg_settings WHERE name = 'log_min_duration_statement';  -- sighup = reload suffit
sudo systemctl reload postgresql
```

**Pourquoi** : chaque `restart` = une **panne planifiée** par ton choix — la pire espèce.

### Piège 4 — Régler sans re-mesurer (le « c'est réglé, on a fini »)

```sql
-- ❌ MAUVAIS : ALTER SYSTEM, puis on passe au sujet suivant (sans vérifier l'effet)
ALTER SYSTEM SET work_mem = '32MB';

-- ✅ CORRECT : la boucle complète
SHOW work_mem;                       -- vérifier la valeur
EXPLAIN ANALYZE <la requête lente>;  -- re-mesurer le comportement
```

**Pourquoi** : le but n'est pas de « mettre 32MB » mais de **rendre le service rapide et prévisible**. Sans re-mesure, tu ne sais même pas si tu as aidé.

### Piège 5 — Ignorer les journaux (la boîte noire)

```
❌ MAUVAIS : « l'application est lente » et personne n'ouvre jamais le journal du service
✅ CORRECT : log_min_duration_statement actif + lecture régulière (journalctl — commande ci-dessus)
```

**Pourquoi** : le journal des requêtes lentes est le **comptable** de la base : il dit exactement quelle requête, quand, et à quel prix. Sans lui, tu diagnostiques à la boule de cristal.

### Piège 6 — Oublier que le disque ne se règle pas dans `postgresql.conf`

```
❌ MAUVAIS : « la base ne répond plus » → chercher un paramètre magique
✅ CORRECT : vérifier l'ESPACE disque d'abord (df -h — vu au Bloc 2), purger/étendre, PUIS enquêter
```

**Pourquoi** : un disque plein n'est pas un problème de configuration — c'est une ressource qui s'épuise, à surveiller en continu (Bloc 12).

---

## 6. Exercice pratique

> 🔁 **Comment s'articulent les fichiers** : la théorie est terminée, passons à la pratique. L'exercice complet est dans **`02-exercice.md`** (à faire **avant** de lire la correction).

En résumé, tu vas — **en local, gratuitement** :

1. faire la **photo initiale** de la base (paramètres + `context`, taux de cache, connexions) ;
2. **activer l'observateur** : journal des requêtes lentes via `ALTER SYSTEM` + `pg_reload_conf()` + vérification `SHOW` ;
3. **provoquer** une requête lente et la **trouver dans le journal** (preuve mesurée) ;
4. **mesurer puis agir** : `EXPLAIN ANALYZE` avant/après un index (la boucle complète) ;
5. **réfléchir** : 3 questions de diagnostic (fausse bonne idée du `max_connections` géant, config copiée d'Internet, la ressource qui ne se règle pas).

Livrable : `notes-exercice-03.md`.

---

## 7. Correction détaillée de l'exercice

La correction complète (sorties attendues, lecture de la colonne `context`, interprétation du taux de cache et des plans `EXPLAIN`) est dans **`03-correction.md`**, qui réécrit la checklist finale et donne des conseils.

---

## 8. Checklist de validation

- [ ] Je peux expliquer pourquoi les réglages par défaut sont prudents (et l'analogie de la selle du vélo).
- [ ] Je sais **trouver** `postgresql.conf` dynamiquement (`SHOW config_file;`) et connaître les 5 paramètres clés et leur rôle.
- [ ] Je sais changer un paramètre avec `ALTER SYSTEM`, le **recharger sans coupure** (`pg_reload_conf()`) et **vérifier** (`SHOW`).
- [ ] Je sais lire la colonne `context` de `pg_settings` et dire si un paramètre demande **reload** ou **restart**.
- [ ] Je sais **mesurer** la santé : taux de cache (`pg_stat_database`), connexions (`pg_stat_activity`), plans (`EXPLAIN ANALYZE`), journal des lenteurs (`journalctl`).
- [ ] Je connais les 3 symptômes d'une base qui suffoque et mon premier réflexe pour chacun.
- [ ] Je peux expliquer le pool de connexions (taxi collectif) et pourquoi il vaut mieux qu'un `max_connections` géant.

---

> 🧭 **Prochaine étape** : ta base est **bien gardée** (Leçon 2) et **bien réglée** (cette leçon). Mais un réglage parfait ne protège de rien face à un **disque qui meurt** ou une **erreur humaine** (`DELETE` sans `WHERE` hors transaction). La **Leçon 4** (backup et restauration) est le sujet le plus important de tout le bloc : sauvegarder, restaurer — et **tester** la restauration, parce qu'*un backup non testé n'est pas un backup*.
