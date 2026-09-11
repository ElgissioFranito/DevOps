# Introduction au Bloc 7 — Bases de données & Data Operations

> **À lire en premier**, avant la Leçon 1. Ce fichier te dit :
> - de quoi parle ce bloc et **pourquoi il est central en DevOps**,
> - ce qu'il te faut **préparer** avant de commencer,
> - le **vocabulaire** que tu vas croiser (recueilli dans les 8 leçons),
> - les **8 leçons** du bloc et le **fil rouge** qui les relie,
> - ce que tu sauras faire à la fin (le critère « bloc acquis »).

---

## 1. De quoi parle ce bloc ? (la vision d'ensemble)

Ce bloc répond à une question simple : **que devient ta base de données en production ?** Pas « comment écrire du SQL » (c'était le Bloc 2, Leçon 6), ni « comment louer une base dans le cloud » (c'était le Bloc 6, Leçon 5 — RDS, les bases « managées ») : **comment l'administrer toi-même**, de la création à la panne simulée, en passant par les droits, le réglage, la sauvegarde et la mise en cache.

### Le fil rouge : la base `bibliotheque`

Tout le bloc suit **la même base**, `bibliotheque` (le catalogue d'une bibliothèque municipale : les tables `livres`, `membres`, `emprunts`) et **les mêmes rôles** :
- `app_biblio` — l'application Spring Boot : elle **lit et écrit** (jamais d'administration) ;
- `lecteur_biblio` — l'analyste : il ne fait que **lire** (jamais modifier).

La Leçon 8 crée une base dédiée **`bibliotheque_prod`** (la « prod ») : même logique, autre environnement — comme `bibliotheque_dev` / `bibliotheque_test` / `bibliotheque_prod` de la Leçon 1 (une base par environnement).

### Les 8 leçons (et leur logique)

| # | Leçon | La question | L'analogie |
|---|---|---|---|
| 1 | SGBD relationnel et PostgreSQL | **Quoi** ranger ? (tables, types, SQL) | Le classeur et son secrétaire |
| 2 | Utilisateurs, rôles, permissions | **QUI** peut toucher à quoi ? (moindre privilège) | L'immeuble à clés |
| 3 | Configuration et ressources | **Comment** ça tourne ? (mesurer → régler → vérifier) | La selle du vélo |
| 4 | Backup et restauration | **Et si on perd tout ?** (dumps, testés, 3-2-1) | La photocopie au coffre |
| 5 | Migrations versionnées (Flyway) | **Comment** évoluer sans improviser ? | Le permis de construire |
| 6 | Réplication et haute disponibilité | **Et si le SERVEUR tombe ?** (RTO/RPO) | Le chef et son sous-chef |
| 7 | Cache applicatif (Redis) | **Comment** aller vite sans fragiliser ? | Le post-it sur l'écran |
| 8 | **Projet** : la base en production | **La preuve** : le runbook vécu | Le carnet de vol |

Chaque dossier de leçon contient 4 fichiers : `01-lecon.md` (théorie), `02-exercice.md` (pratique en autonomie), `03-correction.md` (pas à pas + checklist réécrite), `04-commandes-references.md` (l'aide-mémoire à garder à côté).

> 🔁 **Comment s'articulent les fichiers** : lis d'abord `01-lecon.md` (elle renvoie à l'exercice à la fin), fais `02-exercice.md` **sans regarder la solution**, compare avec `03-correction.md`, et garde `04-commandes-references.md` ouvert pendant tout le bloc.

> 🧭 **Le critère « Bloc acquis » de la roadmap** : *« mettre une base PostgreSQL en production, la sauvegarder, la restaurer, effectuer une migration et expliquer comment éviter une perte de données »* (+ le cache : choisir un TTL et expliquer le stale data). La Leçon 8 est construite exactement pour le démontrer.

---

## 2. Prérequis (ce qu'il faut AVANT de commencer)

- **Bloc 2 (Linux)** : PostgreSQL **installé** (Leçon 6), `systemctl`, `nano`, `journalctl`, `df -h`, `cron`.
- **Bloc 3 (Scripting)** : lire un script Python (`cache-demo.py`, Leçon 7) et écrire du Bash.
- **Bloc 4 (Git)** : le versionnement (les migrations de la Leçon 5 et le runbook de la Leçon 8 vivent dans Git).
- **Bloc 5 (Réseau et sécurité)** : ports (5432, 6379), pare-feu, TLS, RBAC, secrets et coffre.
- **Bloc 6 (Cloud)** : ce que fait une base « managée » (RDS) — pour comprendre **ce que le Bloc 7 fait à la place, à la main**.
- **Python 3 + pip** : pour `redis-py` et `psycopg2-binary` (Leçon 7) — `pip3 install redis psycopg2-binary`.
- **Un éditeur** : `nano` suffit ; VS Code pour les notes `notes-exercice-XX.md`.

> 💡 **100 % local et gratuit** : tout le bloc se fait **chez toi** (PostgreSQL + Redis installés en 2 minutes). Docker arrive au Bloc 9 — on ne l'utilise pas ici (la Leçon 6 t'explique pourquoi, et garde un TP optionnel pour après). **Rien de coûteux ne sera lancé sans te prévenir.**

---

## 3. Vocabulaire du bloc (les mots qui reviendront partout)

Glossaire de survie — chaque leçon a le sien, complet. Voici les mots qui traversent **toutes** les leçons :

| Terme | C'est quoi ? (1 phrase) | Première vraie rencontre |
|---|---|---|
| **SGBD** | Le logiciel qui gère la base (PostgreSQL, MySQL...) | Leçon 1 |
| **SQL** | Le langage pour parler au SGBD | Leçon 1 |
| **Table / ligne / clé primaire** | Le tiroir / la fiche / son numéro unique | Leçon 1 |
| **Rôle** | Le compte du SGBD (`postgres`, `app_biblio`...) — à ne pas confondre avec un utilisateur de ton application | Leçon 2 |
| **Moindre privilège** | Donner seulement les droits nécessaires, rien de plus | Leçon 2 |
| **`GRANT` / `REVOKE`** | Accorder / retirer un droit | Leçon 2 |
| **`pg_hba.conf`** | Le fichier de règles d'entrée (qui, d'où, comment) | Leçon 2 |
| **peer / scram-sha-256** | Sans mot de passe en local / avec mot de passe chiffré | Leçon 2 |
| **Guichet (`max_connections`)** | Une connexion occupée = une table du restaurant prise | Leçons 1 et 3 |
| **Taux de cache** | La part des lectures servies par la RAM (viser > 99 %) | Leçon 3 |
| **`EXPLAIN ANALYZE`** | Le plan d'exécution réel d'une requête | Leçon 3 |
| **Backup / restore** | La copie / la remise en service depuis la copie | Leçon 4 |
| **`pg_dump` / `pg_dumpall`** | La base / les rôles (les deux, toujours) | Leçon 4 |
| **WAL** | Le carnet de bord du SGBD (journal avant les données) | Leçon 4 |
| **PITR** | Restaurer **à un instant précis** (photo + carnet rejoué) | Leçon 4 |
| **3-2-1** | 3 copies, 2 supports, 1 hors site | Leçon 4 |
| **Migration (Flyway)** | Un changement numéroté (`V1__...`), appliqué une fois, journalisé | Leçon 5 |
| **Checksum** | L'empreinte d'un fichier — on n'édite jamais une migration appliquée | Leçon 5 |
| **Forward fix** | Corriger **en avançant** (`V(n+1)`), jamais en reculant | Leçon 5 |
| **Primary / réplica** | Le chef qui écrit / le sous-chef qui suit et lit | Leçon 6 |
| **Lag** | Le retard du réplica (le même réflexe que le stale data) | Leçon 6 |
| **Failover / promotion** | La bascule : le réplica devient primary | Leçon 6 |
| **RTO / RPO** | Durée d'interruption acceptable / données perdues acceptables — **avec unité** | Leçon 6 |
| **SPOF** | Le point de défaillance unique (à ne pas créer, même au cache) | Leçons 6 et 7 |
| **Cache hit / miss** | Trouvé dans le cache / pas trouvé (→ la base) | Leçon 7 |
| **TTL** | La durée de vie d'une entrée (le filet de l'oubli) | Leçon 7 |
| **Invalidation / stale data** | Retirer l'entrée fausse / la réponse périmée | Leçon 7 |
| **Runbook** | Le carnet de conduite du service (le livrable de la Leçon 8) | Leçon 8 |

> 📌 **Les mots « à définir, sans creuser » de la roadmap** (tu sauras les définir en 1 phrase, sans en faire plus) : **MongoDB** et **OracleDB** (Leçon 1), **Liquibase** et **Prisma Migrate** (Leçon 5), **Patroni** et **pg_auto_failover** (Leçon 6, TP optionnel après le Bloc 9), **Memcache** et **Infinispan** (Leçon 7), **HikariCP** et **PgBouncer** (mentionnés en Leçon 3).

---

## 4. Ce que tu sauras faire à la fin (et la suite logique)

Si tu coches la checklist de la Leçon 8, la roadmap considère le bloc **acquis** : *« mettre une base PostgreSQL en production, la sauvegarder, la restaurer, effectuer une migration et expliquer comment éviter une perte de données »* — par un **runbook vécu**, pas par des notes de cours.

> 🧭 **Prochaine étape (et fin du bloc)** — Le **Bloc 8 — Infrastructure as Code** : jusqu'ici, tu as tout construit **à la main** (commandes, fichiers, cron). Le bloc suivant transforme ton runbook en **code exécutable** : Terraform créera le réseau, la machine, le stockage, la base managée et les accès. Ce que tu as décrit en mots devient du code. Tu es prêt — commence par la Leçon 1 : crée la base `bibliotheque` et la table `livres`.