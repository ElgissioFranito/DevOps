# Leçon 7 — Cache applicatif : Redis

> **Bloc 7 · Bases de données & Data Operations** — Leçon 7 sur 8
> 🧭 **Pont depuis la Leçon 6** : ta base est protégée de la perte (Leçon 4), du schéma sauvage (Leçon 5) et de la panne du serveur (Leçon 6). Reste la **performance en lecture** — le sujet que la roadmap appelle « le bloc manquant » : *avant d'ajouter des serveurs ou de scaler la base, la mise en cache est souvent le levier de performance le plus rentable*. Le lien avec ta Leçon 3 est direct : tu as appris à **mesurer** (`EXPLAIN ANALYZE`, les index) ; ici tu apprends le levier **au-dessus** de la base : ne pas interroger la base du tout, quand la réponse est déjà connue.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Expliquer** pourquoi et quand mettre des données en cache — et pourquoi c'est le levier **le plus rentable** avant de scaler.
2. **Décrire** le mécanisme : **cache hit / miss**, le modèle **clé-valeur**, et le **TTL** (la durée de vie).
3. **Jouer** avec Redis en ligne de commande (`SET`, `GET`, `EX`, `DEL`) et savoir ce que sont **Memcache** et **Infinispan** (définis, sans creuser — roadmap).
4. **Mettre en cache** le résultat d'une requête coûteuse dans une application (Spring Cache `@Cacheable` — ta stack — avec la variante NestJS).
5. **Gérer l'invalidation** (la partie la plus difficile) et expliquer le risque de **données périmées** (« stale data »).
6. **Prouver** la règle d'or : **le cache tombe ≠ l'application tombe**.

---

## 2. Explication simple

### 2.1 Le « pourquoi » : sans cache, chaque lecture repasse par le tiroir

Sans cache, la **même question** interroge la base encore et encore :

```
Sans cache (chaque visiteur repose la question) :
  requête → application → base de données (lente, sollicitée à CHAQUE requête)

Avec cache :
  requête → application → cache (Redis) → réponse quasi instantanée si déjà présente
                    ↓ (si absent : « cache miss »)
                          base de données
                    ↓
                    la réponse est rangée dans le cache (pour les prochains)
```

> 💡 **Analogie** : la base, c'est le **dossier archivé au sous-sol** (complet, fiable, mais il faut descendre). Le cache, c'est le **post-it sur ton écran** avec la réponse écrite dessus (rapide, mais il finit par se décoller). Tu ne jette pas l'archive : tu évites de descendre au sous-sol **50 fois par heure pour la même réponse**.

**Pourquoi c'est le levier « le plus rentable » (la roadmap le dit)** : ajouter un serveur ou un réplica coûte de l'argent **en continu** ; mettre en cache une requête coûteuse coûte **une heure** de travail et soulage la base de 90 % de ses lectures répétées.

### 2.2 Le « comment » : Redis, le carnet clé-valeur en mémoire

**Redis** (REmote DIctionary Server) est un serveur qui garde des **paires clé → valeur** **en mémoire** (la RAM — rapide, mais effacée à l'extinction : c'est précisément pour cela qu'il n'est **pas** la base).

```
SET user:42 "{...}" EX 3600   # rangé pendant 3600 secondes (1 heure) — EX = l'expiration
GET user:42                    # lecture quasi instantanée
DEL user:42                    # retiré tout de suite (l'invalidation)
```

Redis sait aussi faire plus riche (listes, ensembles, files d'attente) — **pour le bloc, la paire clé-valeur suffit**.

### 2.3 Les règles du jeu : hit, miss, TTL, invalidation, éviction

| Terme | Il veut dire | Analogie post-it |
|---|---|---|
| **Cache hit** | la réponse est **trouvée** dans le cache | le post-it est encore là ✓ |
| **Cache miss** | la réponse **n'y est pas** → on interroge la base | pas de post-it → on descend au sous-sol |
| **TTL** (*Time To Live*, durée de vie) | la durée après laquelle l'entrée **expire toute seule** | le post-it se décolle tout seul au bout d'une heure |
| **Invalidation** | supprimer/mettre à jour une entrée **devenue fausse** (la base a changé) | rayer le post-it quand le dossier a changé |
| **Éviction** | ce que Redis fait quand le cache est **plein** (il retire les moins utilisés) | l'écran n'est pas infini : le post-it le plus vieux disparaît |

**La partie la plus difficile — l'invalidation** (la citation célèbre : *« il n'y a que deux choses difficiles en informatique : l'invalidation de cache et nommer les variables »*) : quand la base change (un `UPDATE` sur `livres`), la réponse cachée devient **fausse**. Deux réflexes :

- **L'invalidation explicite** : au `UPDATE`, on `DEL` la clé (juste, mais il faut penser à chaque écriture) ;
- **Le TTL court** : on accepte quelques secondes de données périmées (simple, mais imprécis).

### 2.4 Le « quand » : le cache de session, et quand NE PAS cacher

Deux usages qui se croisent :

- **Le cache de requête coûteuse** : le résultat du rapport, la liste des livres populaires — la lecture qui revient sans cesse.
- **Le cache de session** : stocker les **sessions utilisateurs** (qui est connecté, son panier) plutôt qu'en base — rapide, et la base reste pour les données fiables.

Et le **quand PAS cacher** — aussi important :

```
NE JAMAIS cacher :
  une donnée JAMAIS relue deux fois   (le cache ne servirait à rien)
  une donnée CRITIQUE à la seconde    (un solde lu juste après un paiement —
                                       le même réflexe que le lag du réplica, Leçon 6)
  une donnée DÉJÀ rapide              (cacher sans avoir MESURÉ — la Leçon 3 :
                                       on ne cache pas sans EXPLAIN ANALYZE d'abord)
```

---

## 📖 Vocabulaire / Abréviations

- **Cache** : le stockage temporaire rapide, devant la base (l'écran aux post-it).
- **Redis** : le serveur **clé-valeur en mémoire**, le cache le plus utilisé aujourd'hui.
- **Clé-valeur** : le modèle « un nom → une donnée » (pas de tables, pas de jointures).
- **Cache hit / miss** : trouvé / pas trouvé dans le cache.
- **TTL** (*Time To Live*) : la durée de vie d'une entrée (elle expire toute seule).
- **`EX`** : l'option Redis qui pose l'expiration (en secondes).
- **Invalidation** : retirer une entrée **devenue fausse** (la base a changé).
- **Stale data** (« données périmées ») : la réponse cachée qui **ne correspond plus** à la base.
- **Éviction** : ce que Redis retire quand le cache est **plein** (les moins utilisés d'abord).
- **Cache de session** : stocker les sessions utilisateurs dans le cache, plutôt qu'en base.
- **Memcache** (*Memcached*) : l'ancien concurrent de Redis — clé-valeur **pur**, plus simple (pas de structures avancées, pas de persistance). Encore utilisé pour un cache basique.
- **Infinispan** : cache distribué de l'écosystème **Java/JBoss**, alternative à Redis — niche, environnement Red Hat/JBoss (à définir, sans creuser — roadmap).
- **Spring Cache** : l'abstraction de cache de Spring — les annotations `@EnableCaching` / `@Cacheable` / `@CacheEvict`.
- **`@Cacheable`** : « le résultat de cette méthode se cache » (Spring gère le hit/miss à ta place).
- **`@CacheEvict`** : « à cet appel, l'entrée du cache est **invalidée** » (au `UPDATE`, au `DELETE`).
- **redis-py** : la librairie Python pour parler à Redis.
- **NestJS** : le framework Node/TypeScript — la variante de la roadmap (via `cache-manager`).
- **`localhost` / 6379** : « cette machine elle-même » / le **port** de Redis (l'équivalent du 5432 de PostgreSQL — Bloc 5).

---

## 3. Exemples concrets

> 🔁 On enchaîne : le mécanisme est posé ; voici **les commandes exactes**, commentées ligne par ligne. Tout est local et gratuit.

### 3.1 Installer et jouer le carnet

```bash
sudo apt update && sudo apt install -y redis-server   # installe Redis (le serveur + l'outil redis-cli)
sudo systemctl status redis-server --no-pager | head -3   # il tourne ? (service vu au Bloc 2)
```

```bash
redis-cli                                        # le client Redis (l'équivalent de psql pour PostgreSQL)
127.0.0.1:6379> SET salutation 'bonjour' EX 30   # rangé pour 30 secondes (EX = expiration en secondes)
127.0.0.1:6379> GET salutation                   # → "bonjour" (le HIT)
127.0.0.1:6379> TTL salutation                   # → les secondes restantes avant l'expiration
127.0.0.1:6379> DEL salutation                   # l'INVALIDATION manuelle (le post-it rayé)
127.0.0.1:6379> GET salutation                   # → (nil) : le MISS
127.0.0.1:6379> exit
```

> 💡 **Le port 6379** : le guichet de Redis (comme 5432 pour PostgreSQL — Bloc 5). En local il écoute sur `localhost` ; en production, il se protège comme la base (mot de passe `requirepass`, pare-feu — Leçon 2, Bloc 5).

### 3.2 Le cache côté application : Spring Cache (ta stack)

Côté Spring, tu **n'écris pas** le hit/miss à la main : des **annotations** le déclarent, et Spring le gère.

```java
// 1. L'activer une seule fois (dans la classe principale) :
@EnableCaching
@SpringBootApplication
public class Application { ... }

// 2. Cacher le résultat (le READ — l'équivalent du r.get + r.set de l'exercice) :
@Cacheable(value = "topCategories")          // le « nom du post-it » : la valeur cachée porte cette clé
public List<CategorieStats> topCategories() {
    return statsRepository.topCategories();  // la requête coûteuse : n'est lancée QUE au miss
}

// 3. L'invalidation (le WRITE — l'équivalent du r.delete) :
@CacheEvict(value = "topCategories")         // à CET appel, l'entrée est retirée
public CategorieStats majCategorie(Long id, CategorieStats c) {
    return statsRepository.save(c);          // la base change → le post-it doit mourir
}
```

Et le **réglage du TTL** (dans `application.properties`) :

```properties
spring.cache.redis.time-to-live=PT60S       # PT60S = 60 secondes (format ISO de durée)
```

> 💡 **Pourquoi les annotations et pas du Redis à la main ?** Parce que le hit/miss/TTL/invalidation est un **motif répétitif** : le déclarer une fois évite de le coder mal cent fois. Et si demain le cache devient Memcache ou Infinispan (la roadmap), l'annotation reste — seul le moteur change.

### 3.3 La variante NestJS (la roadmap la demande)

```typescript
// NestJS + cache-manager : le même motif, autre syntaxe
import { CacheModule, Cache } from '@nestjs/common';
// Register : le module avec son TTL (en millisecondes) ; inject : le service Cache
CacheModule.register({ ttl: 60_000 });        // 60 secondes

// Dans un service :
const dansCache = await this.cacheManager.get('topCategories');    // le hit ?
if (dansCache) return dansCache;                                   // ✓
const reponse = await this.statsService.calcul();                  // le miss : la base
await this.cacheManager.set('topCategories', reponse, { ttl: 60_000 });
return reponse;
```

### 3.4 Le cache de session

```java
// Spring Session + Redis : les sessions (qui est connecté) vivent dans le cache
// dépendance : spring-session-data-redis — 2 lignes de config, rien à coder
spring.session.store-type=redis
spring.session.redis.flush-mode=on-save       # la session est rangée à chaque sauvegarde
```

> 💡 **Pourquoi les sessions au cache et pas en base ?** Elles sont lues **à chaque requête** de chaque utilisateur connecté : les garder en base alourdit PostgreSQL pour une donnée **éphémère** (une session expire avec le TTL). La base reste pour les données **fiables** — le cache pour l'**éphémère fréquent**.

### 3.5 Prouver la règle d'or : le cache tombe ≠ l'application tombe

```bash
sudo systemctl stop redis-server     # on ARRÊTE le cache volontairement (l'incident simulé)
```

```python
# Dans le script, entourer l'appel Redis d'un try/except (le « garde-fou ») :
try:
    dans_cache = r.get(cle)                       # si Redis répond : le hit normal
except redis.exceptions.ConnectionError:
    dans_cache = None                             # Redis est tombé → comme un MISS
# → au « miss », le script interroge la BASE : l'application continue, juste plus lentement
```

```
Le cache tombe :
  requête → application → cache ✗ (ConnectionError) → comme un miss → la base répond
  → l'application FONCTIONNE (plus lentement) — jamais de panne totale à cause du cache
```

> 💡 **Pourquoi c'est une règle d'or (et non un détail)** : le cache est un **accélérateur**, pas un **organe vital**. Si l'application meurt quand Redis meurt, on a construit un **deuxième point de défaillance unique** — exactement ce que la Leçon 6 vient d'enseigner à éviter.

---

## 4. Bonnes pratiques modernes (2025-2026)

1. **Toujours un TTL** (même long) : une entrée **sans expiration** est un post-it collé à vie — la source n° 1 de données périmées. Le TTL est le filet **par défaut** de l'invalidation oubliée.
2. **Mesurer AVANT de cacher** (`EXPLAIN ANALYZE` — Leçon 3) : on cache une requête **coûteuse et fréquente**, pas « tout ce qui bouge ».
3. **L'invalidation à côté de l'écriture** : le `@CacheEvict` (ou le `DEL`) vit **dans la même méthode** que le `UPDATE` — jamais dans un fichier « à part » qu'on oublie.
4. **Clés nommées avec structure** : `cache:top_categories`, `session:42` — le **préfixe** dit à quoi sert la clé (l'invalidation en masse devient possible : `DEL cache:*`).
5. **Le cache est optionnel** (try/except, règle 3.5) : la panne du cache = l'application **ralentit**, jamais **meurt**.
6. **Sécuriser Redis comme la base** (en prod) : `requirepass` (le mot de passe), réseau fermé (`bind` + pare-feu — Bloc 5), TLS si le trajet quitte la machine.
7. **Surveiller le taux de hit** (`INFO stats` → `keyspace_hits / keyspace_misses`) : un cache que personne ne relit coûte de la RAM pour rien (le miroir du taux de cache de la Leçon 3, côté base).
8. **Redis en prod = managé quand possible** (ElastiCache côté AWS — le pendant du RDS de la Leçon 6) : le fournisseur gère la panne, tu gères le TTL.

---

## 5. Pièges à éviter

### Piège 1 — Cacher « pour aller plus vite » sans avoir mesuré

```
❌ MAUVAIS : @Cacheable sur TOUTES les méthodes du repository
   → de la RAM consommée, des données périmées partout, un gain invisible (les requêtes étaient déjà rapides).

✅ CORRECT : EXPLAIN ANALYZE d'abord (Leçon 3) → cacher LA requête coûteuse ET fréquente
   → un gain mesuré avant, prouvé après (le taux de hit).
```

**Pourquoi** : c'est l'optimisation aveugle — le piège n° 1 de la Leçon 3, version cache.

### Piège 2 — Le TTL infini (le post-it collé à vie)

```bash
SET top_categories "..."        # ❌ sans EX : l'entrée vit JUSQU'À l'invalidation manuelle (souvent oubliée)
SET top_categories "..." EX 60  # ✅ expire toute seule : le filet de l'oubli
```

**Pourquoi** : l'oubli d'invalidation se corrige tout seul avec un TTL ; sans TTL, il dure des mois.

### Piège 3 — Cacher une donnée critique à la seconde

```
❌ MAUVAIS : cacher le SOLDE d'un emprunt lu juste après un paiement
   → l'utilisateur voit un solde périmé : pire qu'une lenteur, c'est une MÉCONNAISSANCE.

✅ CORRECT : le solde se lit à la source (la base) — comme la relecture critique au primary (Leçon 6).
   On cache les listes, les agrégats, les sessions — pas le chiffre dont la vie dépend.
```

**Pourquoi** : le même réflexe que le lag du réplica — la donnée dont l'utilisateur dépend **à l'instant** ne se lit pas dans une copie.

### Piège 4 — L'invalidation oubliée (la citation célèbre en action)

```java
// ❌ MAUVAIS : la requête cachée, le UPDATE sans @CacheEvict
@Cacheable(value = "topCategories") public List<...> topCategories() { ... }
public void majCategorie(...) { repo.save(...); }        // le post-it devient FAUX et vit encore

// ✅ CORRECT : l'invalidation À CÔTÉ de l'écriture (+ le TTL en filet)
@Cacheable(value = "topCategories") public List<...> topCategories() { ... }
@CacheEvict(value = "topCategories")
public void majCategorie(...) { repo.save(...); }
```

**Pourquoi** : c'est LA difficulté classique du cache (la citation du 2.3). La double protection : **l'invalidation pour le juste, le TTL pour l'oubli**.

### Piège 5 — Faire de Redis un organe vital

```python
# ❌ MAUVAIS : l'appel Redis sans filet — Redis tombe, l'application meurt
dans_cache = r.get(cle)

# ✅ CORRECT : try/except — Redis tombé = comme un miss, la base répond
try:
    dans_cache = r.get(cle)
except redis.exceptions.ConnectionError:
    dans_cache = None
```

**Pourquoi** : un accélérateur qui devient un point de défaillance unique (le SPOF de la Leçon 6) est une régression.

### Piège 6 — Redis ouvert sur le réseau sans mot de passe

```conf
# ❌ MAUVAIS (en prod) : bind 0.0.0.0 sans requirepass
#    → n'importe qui d'Internet écrit dans ton cache (des robots le scannent en permanence)

# ✅ CORRECT :
bind 127.0.0.1        # local seulement, ou le réseau privé du serveur applicatif
requirepass 'fort_et_unique'   # le mot de passe (comme le rôle de la Leçon 2)
```

**Pourquoi** : Redis **croit** à tout ce qu'on lui dit — un cache ouvert est une **écriture ouverte** sur ton application.

---

## 6. Exercice pratique

> 🔁 **Comment s'articulent les fichiers** : la théorie est terminée, passons à la pratique — tu vas **mesurer le gain**, vivre le **stale data**, et prouver la règle d'or. L'exercice complet est dans **`02-exercice.md`** (à faire **avant** la correction).

En résumé, tu vas — **en local, gratuitement** :

1. installer Redis et jouer le carnet (`SET`/`GET`/`TTL`/`DEL`) ;
2. **mesurer AVANT** (le réflexe Leçon 3 : `EXPLAIN ANALYZE` sur une requête coûteuse) ;
3. mettre en cache le résultat (script Python avec **hit/miss/TTL 60 s**) et **mesurer le gain** ;
4. **vivre l'invalidation** : modifier la base pendant que le cache vit (le stale data), puis corriger (`DEL` ou TTL court) ;
5. **prouver la règle d'or** : arrêter Redis, vérifier que l'application **continue** ;
6. **décider** : quelles données du fil rouge méritent un cache, avec quel TTL, et pourquoi.

Livrable : `notes-exercice-07.md`.

---

## 7. Correction détaillée de l'exercice

La correction complète (sorties attendues, la règle d'invalidation par donnée, la grille de décision) est dans **`03-correction.md`**, qui réécrit la checklist finale et donne des conseils.

---

## 8. Checklist de validation

- [ ] Je peux expliquer le cache avec l'analogie du **sous-sol et du post-it**, et pourquoi c'est le levier le plus rentable avant de scaler.
- [ ] Je sais décrire **hit / miss / TTL / invalidation / éviction** en une phrase chacun.
- [ ] Je sais jouer avec Redis en ligne de commande (`SET ... EX`, `GET`, `TTL`, `DEL`).
- [ ] Je sais **mesurer** le gain : le prix du miss (`EXPLAIN ANALYZE`) et le temps du hit (l'exercice l'a prouvé).
- [ ] Je sais mettre en cache côté application : `@Cacheable` / `@CacheEvict` + TTL (Spring), et je connais la variante NestJS.
- [ ] Je peux expliquer le **stale data** (l'expérience de l'étape 4) et la double protection : **invalidation pour le juste, TTL pour l'oubli**.
- [ ] Je peux **prouver** que le cache tombe ≠ l'application tombe (le try/except, l'expérience de l'étape 5).
- [ ] Je sais ce que sont **Memcache** et **Infinispan** en une phrase chacun (définis, sans creuser — roadmap).
- [ ] Je sais dire quelles données **ne se cachent jamais** (critique à la seconde, jamais relue, déjà rapide).

---

> 🧭 **Prochaine étape — la dernière leçon du bloc** : le **Leçon 8** est le **projet récapitulatif « base en production »** — le critère de validation de la roadmap : *« mettre une base PostgreSQL en production, la sauvegarder, la restaurer, effectuer une migration et expliquer comment éviter une perte de données »*. Tu vas assembler **les 7 leçons** en un seul livrable : le **runbook** (le carnet de conduite d'un service). C'est la synthèse qui transforme les connaissances en preuve.
