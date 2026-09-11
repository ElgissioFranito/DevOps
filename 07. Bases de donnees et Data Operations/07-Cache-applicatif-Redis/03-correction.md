# Correction 7 — Mettre en cache une requête coûteuse

> Compare **ta démarche** à celle-ci. Le but n'est pas le même script — c'est le **même mécanisme** : mesurer → cacher → vivre le périmé → invalidation → filet.

## Étape 1 — Redis installé et compris

```
127.0.0.1:6379> SET salutation 'bonjour' EX 30
OK
127.0.0.1:6379> GET salutation      → "bonjour"          (le HIT)
127.0.0.1:6379> TTL salutation      → 30 → 29 → ...      (les secondes qui défilent)
127.0.0.1:6379> DEL salutation      → 1                  (l'invalidation : 1 = supprimé)
127.0.0.1:6379> GET salutation      → (nil)              (le MISS — après DEL ou après 30 s)
```

**Ce que tu devrais noter** : `TTL` renvoie **-2** quand la clé **n'existe pas** (ou plus) et **-1** quand elle existe **sans expiration** — le piège 2 de la leçon, vu de tes yeux.

## Étape 2 — Le prix du miss (attendu)

```sql
CREATE TABLE stats_livres AS
SELECT g AS id, 'livre-' || g AS titre, (g % 500) AS categorie
FROM generate_series(1, 200000) g;
-- 200 000 lignes, 500 catégories : la requête GROUP BY a du grain à moudre

EXPLAIN ANALYZE SELECT categorie, count(*) FROM stats_livres GROUP BY categorie;
```

Temps attendu : quelques **dizaines de millisecondes** (grâce à l'index — Leçon 3). Ce chiffre, c'est **le prix du miss** : c'est lui que le cache évite 99 % du temps.

## Étape 3 — Le gain mesuré (attendu)

```
$ time python3 cache-demo.py
([...] , 'MISS')     ← la 1re fois : la base est interrogée (~le temps de l'EXPLAIN)
$ time python3 cache-demo.py
([...] , 'HIT')      ← ensuite : quelques MILLISECONDES (la RAM répond)
real    0m0.0xx s    ← le temps total s'effondre au 2e appel
```

**Ce que tu devrais noter** : le HIT est **plusieurs dizaines de fois** plus rapide — et surtout **la base n'a pas été sollicitée** : c'est le vrai gain (le serveur de base respire pour les autres).

## Étape 4 — Le stale data (le moment pédagogique)

```
HIT (le cache répond) → INSERT en base (le monde change) → HIT (le cache répond ENCORE)
→ mais la réponse ne contient PAS la catégorie 42 : PÉRIMÉE (le « stale data »)
```

**La règle attendue dans tes notes** — la grille par type de donnée :

| Donnée | Règle choisie | Pourquoi |
|---|---|---|
| La liste des livres populaires (relue sans cesse, change rarement) | **TTL 5 min** (+ invalidation au `UPDATE` si on veut le juste) | quelques minutes de périmé sont tolérables |
| Le résultat du rapport du jour | **TTL 1 h** | il est de toute façon recalculé à heure fixe |
| La **session** utilisateur | **TTL = la durée de la session** | éphémère par nature |
| Le **solde** lu juste après un paiement | **PAS de cache** — lecture à la source | critique à la seconde (le réflexe Leçon 6) |

**La phrase à retenir** : *l'invalidation pour le juste, le TTL pour l'oubli* — et **aucun cache** pour le critique à la seconde.

## Étape 5 — La règle d'or prouvée (attendu)

```bash
sudo systemctl stop redis-server     # l'incident : le cache meurt
```

Le script avec le filet :

```python
try:
    dans_cache = r.get(cle)                                  # Redis répond → le hit
except redis.exceptions.ConnectionError:
    dans_cache = None                                        # Redis tombé → comme un miss
```

```
$ python3 cache-demo.py        ← Redis ARRÊTÉ
([...] , 'MISS')               ← le script CONTINUE : la base répond
$ sudo systemctl start redis-server   # le cache revient, tout se recache au fil de l'eau
```

**Ce que tu devrais noter** : l'application a **fonctionné pendant l'incident** (juste plus lentement). Le cache est un **accélérateur**, pas un organe vital.

## Étape 6 — La décision sur le fil rouge (réponse attendue)

```
CACHÉ (Redis) :
  la liste des livres populaires      → TTL 5 min + invalidation au UPDATE
  les agrégats de stats_livres        → TTL 1 h
  les SESSIONS utilisateurs           → TTL = la durée de session (Spring Session, 3.4)

PAS CACHÉ (lecture à la source) :
  le statut d'un emprunt lu après validation  → critique à la seconde
  le détail d'un livre après modification     → la relecture immédiate (le réflexe lag, Leçon 6)
```

## ✅ Checklist de validation (réécrite)

- [ ] Je décris le cache avec l'analogie du **sous-sol et du post-it** — et je sais pourquoi c'est le levier le plus rentable avant de scaler.
- [ ] Je connais **hit / miss / TTL / invalidation / éviction** en une phrase chacun — et pourquoi l'invalidation est « la partie la plus difficile ».
- [ ] Je joue avec Redis (`SET ... EX`, `GET`, `TTL`, `DEL`) et je sais lire `TTL` = -1 / -2 (sans expiration / inexistante).
- [ ] J'ai **mesuré** : le prix du miss (`EXPLAIN ANALYZE`) vs le temps du hit — le gain est prouvé, pas deviné.
- [ ] Je sais cacher côté application : `@Cacheable` / `@CacheEvict` + le TTL (`PT60S`), et je situe la variante NestJS.
- [ ] J'ai **vécu** le stale data et je connais la grille par donnée (TTL court / long / invalidation explicite / jamais de cache).
- [ ] J'ai **prouvé** la règle d'or (Redis arrêté, l'application qui tourne — le try/except).
- [ ] Je sais définir **Memcache** et **Infinispan** en une phrase chacun (sans creuser — roadmap).
- [ ] Je sais sécuriser Redis en prod (`bind` + `requirepass`) et surveiller le taux de hit (`INFO stats`).

## 💡 Conseils

- **La grille par donnée est ta vraie compétence** : « cacher tout » est facile ; décider **quoi, combien de temps, avec quelle invalidation** est le métier — ta grille de l'étape 6 en est le livrable.
- **Le try/except n'est pas décoratif** : écris-le **systématiquement** dès le premier appel Redis de ta vie — l'équivalent du `WHERE` du `DELETE` (Leçon 1) : le filet qui coûte une ligne et sauve le service.
- **Le taux de hit se surveille** : `redis-cli INFO stats` → `keyspace_hits / (keyspace_hits + keyspace_misses)` — un cache que personne ne relit coûte de la RAM pour rien (le miroir du taux de cache de la Leçon 3).
- **Le pont vers la Leçon 8** : le cache s'ajoute à l'architecture — et le projet final te fera **rédiger** cette décision comme tout le reste, dans le **runbook**.

---

> 🎉 **Fin de la correction de la Leçon 7.** Si tout est coché, tu sais rendre le service **rapide** — et la dernière leçon transforme les 7 leçons en **preuve** : le projet récapitulatif.
