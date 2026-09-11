# Commandes & références 7 — Redis

## 1. Installer, vérifier, entrer

```bash
sudo apt update && sudo apt install -y redis-server       # le serveur + l'outil redis-cli
sudo systemctl status redis-server --no-pager | head -3   # il tourne ?
redis-cli                                                  # le client (l'équivalent de psql)
sudo systemctl reload redis-server                         # recharger la config (sans coupure)
sudo systemctl stop redis-server                           # l'arrêt (l'incident simulé)
```

## 2. redis-cli — les commandes vitales

```
SET cle 'valeur' EX 60     # rangé pour 60 s (EX = expiration en secondes) — TOUJOURS un EX
SET cle 'valeur' PX 60000  # idem en millisecondes (PX)
GET cle                    # lire (le HIT) — (nil) si absent (le MISS)
TTL cle                    # les secondes restantes — -2 : n'existe pas · -1 : SANS expiration (danger)
DEL cle                    # l'invalidation manuelle (renvoie le nombre supprimé)
EXPIRE cle 120             # poser/changer l'expiration d'une clé existante
PERSIST cle                # retirer l'expiration (l'inverse) — à éviter
KEYS cache:*               # lister les clés d'un préfixe (⚠️ interdit en prod : bloque — préférer SCAN)
SCAN 0 MATCH cache:* COUNT 100   # lister sans bloquer (le KEYS de production)
TYPE cle                   # le type de la valeur (string, list, set...)
INFO stats                 # keyspace_hits / keyspace_misses — le TAUX DE HIT à surveiller
DBSIZE                     # le nombre de clés (la taille du carnet)
FLUSHDB                    # ⚠️ vider TOUT — jamais en prod
exit
```

## 3. Le motif hit/miss/invalidation (le squelette à copier)

```python
# pip3 install redis psycopg2-binary
import json, redis, psycopg2

r = redis.Redis(host='localhost', port=6379, decode_responses=True)

def lire_avec_cache(cle, requete, ttl=60):
    try:                                        # LE FILET (règle d'or) :
        dans_cache = r.get(cle)                 # Redis tombé = comme un miss
    except redis.exceptions.ConnectionError:
        dans_cache = None
    if dans_cache:
        return json.loads(dans_cache), 'HIT'    # ✓ la RAM répond (la base n'est pas sollicitée)
    cn = psycopg2.connect(dbname='bibliotheque', user='app_biblio', password='...')
    cur = cn.cursor()
    cur.execute(requete)                        # le miss : la base répond
    reponse = cur.fetchall()
    r.set(cle, json.dumps(reponse), ex=ttl)     # rangé AVEC son TTL (jamais sans)
    return reponse, 'MISS'

# L'invalidation vit À CÔTÉ de l'écriture (jamais dans un fichier à part) :
def apres_mise_a_jour(cle):
    r.delete(cle)                               # le post-it rayé dès que la base change
```

## 4. Spring Boot (ta stack) — le même motif par annotations

```java
@EnableCaching                       // une seule fois (la classe principale)
@SpringBootApplication
public class Application { ... }

@Cacheable(value = "topCategories")                 // le READ : hit/miss gérés par Spring
public List<CategorieStats> topCategories() { ... }

@CacheEvict(value = "topCategories")                // le WRITE : l'invalidation à côté
public void majCategorie(...) { ... }
```

```properties
# application.properties
spring.cache.redis.time-to-live=PT60S      # le TTL (format ISO : PT60S = 60 s)
spring.session.store-type=redis            # le cache de session (dépendance spring-session-data-redis)
```

## 5. NestJS (la variante roadmap)

```typescript
CacheModule.register({ ttl: 60_000 });                    // le TTL (en millisecondes)
const dansCache = await this.cacheManager.get('topCategories');   // le hit ?
if (dansCache) return dansCache;                                  // ✓
const reponse = await this.statsService.calcul();                 // le miss : la base
await this.cacheManager.set('topCategories', reponse, { ttl: 60_000 });
await this.cacheManager.del('topCategories');                     // l'invalidation
```

## 6. Securiser Redis en production (redis.conf)

```conf
bind 127.0.0.1                  # local seulement (ou l'IP privée du serveur applicatif)
requirepass 'fort_et_unique'    # le mot de passe (le rôle de la Leçon 2 appliqué au cache)
maxmemory 256mb                 # la RAM max du carnet (le plan de travail a une taille)
maxmemory-policy allkeys-lru    # quand plein : retirer les MOINS UTILISÉS (l'éviction)
```

## 7. Le tableau de décision (à garder en tête)

| Donnée | Cacher ? | TTL | Invalidation |
|---|---|---|---|
| Liste des populaires (relue sans cesse) | ✓ | 5 min | au UPDATE (`@CacheEvict`) |
| Agrégats / rapport du jour | ✓ | 1 h | au recalcul |
| Session utilisateur | ✓ | la durée de session | à la déconnexion |
| Solde lu après un paiement | ✗ jamais | — | lecture à la source |
| Donnée jamais relue deux fois | ✗ (inutile) | — | — |
| Donnée déjà rapide (mesurée) | ✗ (avant d'avoir mesuré) | — | — |

## 8. Liens croisés du bloc

| Besoin | Où |
|---|---|
| Mesurer avant (EXPLAIN ANALYZE, index) | `03-Configuration-et-ressources/01-lecon.md` |
| La donnée critique à la seconde (le réflexe lag) | `06-Replication-et-haute-disponibilite/01-lecon.md` |
| Le mot de passe / bind (la sécurité réseau) | `02-Utilisateurs-roles-permissions-et-connexions/01-lecon.md` · Bloc 5 |
| Docker (Redis en conteneur) | Bloc 9 — Conteneurs |
