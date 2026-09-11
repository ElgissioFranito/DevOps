# Exercice 7 — Mettre en cache une requête coûteuse (et gérer l'invalidation)

> **Objectifs** : installer Redis, mesurer le gain, mettre en cache avec TTL, vivre l'**invalidation**, prouver la règle « le cache tombe ≠ l'application tombe ».
> **Durée** : ~1h · **Prérequis** : Leçons 1-6 (base `bibliotheque` + rôles + backups + migrations).
> **Livrable** : `notes-exercice-07.md` dans ce dossier.

## Étape 1 — Installer et découvrir Redis

```bash
sudo apt update && sudo apt install -y redis-server   # installe Redis (le serveur + l'outil redis-cli)
sudo systemctl status redis-server --no-pager | head -3   # il tourne ? (service vu au Bloc 2)
```

Puis joue le carnet :

```bash
redis-cli                                        # le client Redis (comme psql pour PostgreSQL)
127.0.0.1:6379> SET salutation 'bonjour' EX 30   # rangé pour 30 secondes (EX = expiration)
127.0.0.1:6379> GET salutation                   # → "bonjour" (le hit)
127.0.0.1:6379> TTL salutation                   # → les secondes restantes avant l'expiration
127.0.0.1:6379> DEL salutation                   # l'invalidation manuelle
127.0.0.1:6379> exit
```

Note ce qui se passe si tu refais `GET salutation` **après** 30 secondes (le miss).

## Étape 2 — Mesurer AVANT (le réflexe de la Leçon 3)

Crée des données de charge, puis mesure :

```sql
-- dans bibliotheque (session admin) : une table de 200 000 lignes
CREATE TABLE stats_livres AS
SELECT g AS id, 'livre-' || g AS titre, (g % 500) AS categorie
FROM generate_series(1, 200000) g;

CREATE INDEX ON stats_livres (categorie);   -- l'index de la Leçon 3
EXPLAIN ANALYZE SELECT categorie, count(*) FROM stats_livres GROUP BY categorie;
```

Note le temps d'exécution : c'est le **prix du miss**.

## Étape 3 — Le cache côté application (un script Python)

Crée `cache-demo.py` (dans le dossier de l'exercice) :

```python
# pip3 install redis psycopg2-binary    (les 2 librairies : redis-py et le pilote PostgreSQL)
import json, redis, psycopg2

r = redis.Redis(host='localhost', port=6379, decode_responses=True)   # le client Redis
cn = psycopg2.connect(dbname='bibliotheque', user='app_biblio', password='...')  # ta chaîne (Leçon 2)

def top_categories():
    cle = 'cache:top_categories'                       # la CLÉ du cache (le nom du post-it)
    dans_cache = r.get(cle)                            # le hit ?
    if dans_cache:
        return json.loads(dans_cache), 'HIT'           # ✓ trouvé : la base n'est pas sollicitée
    cur = cn.cursor()                                  # le miss : on interroge la base
    cur.execute('SELECT categorie, count(*) FROM stats_livres GROUP BY categorie')
    reponse = cur.fetchall()
    r.set(cle, json.dumps(reponse), ex=60)             # rangé pour 60 s (le TTL)
    return reponse, 'MISS'

print(top_categories())   # la 1re fois : MISS (base interrogée)
print(top_categories())   # ensuite : HIT (quasi instantané)
```

Exécute `python3 cache-demo.py` **deux fois** et note : MISS puis HIT — et le **temps** de chaque appel (`time python3 cache-demo.py`).

## Étape 4 — Vivre l'invalidation (les données périmées)

1. Relance le script : **HIT** (la réponse cachée est là).
2. **Change la base** pendant que le cache vit :
   ```sql
   -- dans bibliotheque :
   INSERT INTO stats_livres (titre, categorie) VALUES ('nouveau', 42);
   ```
3. Relance le script : **HIT** — mais la réponse est **fausse** (la catégorie 42 n'y est pas) : c'est le **stale data**.
4. **Corrige par l'invalidation** : ajoute au script, à côté de l'écriture en base, `r.delete('cache:top_categories')` (le `DEL`) — ou choisis un **TTL plus court** (5 s) et accepte 5 secondes de périmé.

Note dans tes notes : **quelle approche** tu choisis pour quelle donnée (l'invalidation explicite pour le juste ; le TTL court pour le simple).

## Étape 5 — Prouver la règle d'or (le cache tombe ≠ l'application tombe)

```bash
sudo systemctl stop redis-server     # on ARRÊTE le cache volontairement
```

Dans le script, entoure l'appel Redis d'un **try/except** (si Redis ne répond pas, on va **directement à la base** — l'application continue, juste plus lentement). Relance : l'application doit **fonctionner**. Relance `sudo systemctl start redis-server` après.

## Étape 6 — Rédiger (dans `notes-exercice-07.md`)

1. Le temps **miss vs hit** (mesuré à l'étape 3).
2. Ce que tu as observé à l'étape 4 (le périmé) et ta règle d'invalidation choisie.
3. La preuve de l'étape 5 (le cache arrêté, l'application qui tourne).
4. **La décision** : sur le fil rouge `bibliotheque`, quelles **2 données** méritent un cache, avec quel **TTL**, et **pourquoi** (les sessions ? la liste des livres populaires ? le solde d'un emprunt ? — pense au piège du « solde à la seconde », comme le lag du réplica).

> 🧭 **Astuce anti-frustration** : si `redis-py` te résiste, tu peux faire l'étape 3 **en bash** avec `redis-cli` et `psql` — le mécanisme (hit/miss/TTL/invalidation) est identique.