# Leçon 6 — Réplication et haute disponibilité

> **Bloc 7 · Bases de données & Data Operations** — Leçon 6 sur 8
> 🧭 **Pont depuis la Leçon 5** : ton schéma est versionné dans Git, tes migrations sont répétables, tes données ont un backup testé. Mais tout cela repose sur **un seul serveur**. La roadmap exige ici : *« réplication (primary/replica), bascule, haute disponibilité »*. La question change : ce n'est plus « vais-je perdre mes données ? » (le Leçon 4 a répondu), ni « mon schéma est-il maîtrisé ? » (le Leçon 5 a répondu), mais **« combien de temps mon service reste-t-il coupé si le serveur tombe ? »**.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Expliquer** pourquoi un serveur unique est un **point de défaillance unique** (SPOF) — et pourquoi le backup n'y répond pas.
2. **Décrire** la réplication **primary → réplica** : rôles, flux, et le **WAL** comme carnet de bord copié en direct.
3. **Choisir** entre réplication **synchrone** et **asynchrone** selon le métier (le couple performance / risque de perte).
4. **Utiliser** un réplica pour les **lectures** (rapports, dashboards) — et connaître le piège du **lag** (retard).
5. **Comprendre** la **bascule** (promotion du réplica) et la **haute disponibilité** (failover automatique, quorum, anti split-brain).
6. **Chiffrer** un plan de reprise avec les deux indicateurs : **RTO** et **RPO**.
7. **Relier** au managé (RDS Multi-AZ, bloc 6) : ce que le cloud automatise — et ce qui reste ta responsabilité.

---

## 2. Explication simple

### 2.1 Le « pourquoi » : le déclic de la leçon

Faisons le bilan : rôles clos (Leçon 2), configuration maîtrisée (Leçon 3), backups testés (Leçon 4), migrations versionnées (Leçon 5). Il reste un adversaire que **rien de tout cela ne couvre** : la **panne du serveur lui-même** (disque, processus, machine, datacenter).

> 💡 **Analogie** : un backup parfait, c'est une **photocopie au coffre** : indispensable si l'original brûle... mais pendant l'incendie, le magasin est **fermé**. La question de cette leçon n'est plus la perte de données — c'est la **fermeture du magasin**.

Le backup ne fait **rien pendant la panne** : il sert **après**, à la restauration. La haute disponibilité, elle, vise à ce que le service **continue pendant** (ou revienne en minutes).

### 2.2 Le « comment » : primary, réplica, et le WAL copié en direct

```text
┌──────────────────┐    WAL (flux)    ┌──────────────────┐
│  PRIMARY         │  ──────────────► │  RÉPLICA         │
│  je lis, j'écris │                  │  je lis, je copie│
└──────────────────┘                  └──────────────────┘
```

- **Primary** (le « chef ») : le seul serveur qui accepte les **écritures** (`INSERT`, `UPDATE`, `DELETE`).
- **Réplica** (le « sous-chef ») : une copie à chaud qui suit chaque modification et accepte les **lectures**.

Le moteur de la copie, tu le connais déjà : le **WAL** (vu en Leçon 4) — le **journal des modifications** que PostgreSQL écrit **avant** les données (*write-ahead*), en segments de 16 Mo. En réplication, ce journal devient un **flux** : le primary l'**envoie** au réplica, qui le **rejoue** en direct.

> 💡 **Mon outil perso pour mémoriser** — *le chef et son sous-chef* : le chef note **tout** ce qu'il fait dans son carnet de bord (le WAL). Le sous-chef reçoit une **copie du carnet en direct** et rejoue chaque geste — sans jamais décider d'écrire. Le chef tombe malade → on **promeut** le sous-chef. L'ancien chef qui revient a manqué tout le flux : on le **re-clône** et on le recrute comme sous-chef.

### 2.3 Le « combien » : synchrone vs asynchrone

| Mode | Le primary attend-il le réplica ? | Prix | Risque / bénéfice |
|---|---|---|---|
| **Asynchrone** (défaut) | Non — il envoie et continue | Latence d'écriture **inchangée** | Les toutes dernières écritures peuvent manquer au réplica (le **lag**) |
| **Synchrone** | Oui — chaque `COMMIT` attend | Chaque écriture **plus lente** (aller-retour) | Une écriture validée est **sur les deux serveurs** : zéro perte au basculement |

Le réglage PostgreSQL : `synchronous_commit`. **Règle métier** : asynchrone pour la performance (logs, analytics, e-commerce à fort volume) ; synchrone quand perdre **une seconde** d'écriture est inacceptable (paiement, dossier médical).

### 2.4 Le « où » : lire ailleurs (et le piège du lag)

Le réplica n'est pas qu'une assurance — il **travaille** :

- Le **rapport** du lundi matin (agrégation de millions de lignes) tourne sur le réplica, sans ralentir le primary qui sert les clients.
- Dashboards, recherches, exports : tout ce qui **lit beaucoup** peut pointer vers le réplica.

> ⚠️ **Le piège du lag** : le réplica a un **retard** (souvent < 1 s, parfois quelques secondes). L'utilisateur qui vient d'enregistrer et est **renvoyé vers le réplica** pour vérifier pourrait « ne pas voir » sa propre écriture. Le réflexe : les écritures critiques **et leur relecture immédiate** passent par le primary ; le réplica reçoit les lectures **tolérantes au retard**.

Côté PostgreSQL, une seule question pour connaître le rôle d'un serveur :

```sql
SELECT pg_is_in_recovery();   -- t = « je suis un réplica, je rejoue le journal » · f = « je suis le primary »
```

### 2.5 La bascule : promotion, puis haute disponibilité

**Promotion manuelle** (sur le réplica) :

```sql
SELECT pg_promote();   -- « tu es le chef maintenant »
```

- Le réplica **arrête de rejouer** le journal et **accepte les écritures**.
- **L'application doit être redirigée à part** (nouvelle chaîne de connexion, DNS, proxy) — la promotion ne redirige personne à ta place.
- **Irréversible dans le sens du retour** : l'ancien primary qui revient a manqué tout le flux manqué — on le **re-clone** (`pg_basebackup`, Leçon 4) et on le re-déploie comme réplica. On ne le « réintègre » jamais tel quel.

**Haute disponibilité (failover automatique)** : faire la promotion à la main à 3 h du matin, c'est un humain éveillé — facteur de risque. La HA automatise la chaîne : **détecter** la panne → **élire** un nouveau primary → **promouvoir** → **rediriger**.

Deux outils de référence : **Patroni** (grappe de nœuds avec élection de leader) et **pg_auto_failover** (primary + réplica + **moniteur** qui observe et décide).

> 💡 **Le concept central : le quorum**. Un serveur isolé du réseau ne doit pas se proclamer primary tout seul. Sinon : *split-brain* — **deux** serveurs acceptent des écritures divergentes, le scénario catastrophe. D'où un **tiers décideur** (le moniteur de pg_auto_failover) ou une **règle de majorité** (Patroni). À retenir : **on ne promeut jamais sans majorité.**

### 2.6 RTO / RPO : les deux chiffres du plan de reprise

Un plan de reprise sans chiffres n'est pas un plan — c'est une intention.

- **RTO** (*Recovery Time Objective*) : combien de **temps** le service peut rester coupé.
- **RPO** (*Recovery Point Objective*) : combien de **données** (en temps) on peut perdre.

```text
PANNE ────────────────────────────────────────────►
   │◄──────── RTO : le service revient ────────►│
   │◄─ RPO : données perdues ─►│
```

Le trio de paris sur le même couple RTO/RPO :

| Configuration | RPO | RTO |
|---|---|---|
| Backup nocturne seul (Leçon 4) | jusqu'à 24 h | long (restauration manuelle) |
| Réplication asynchrone | secondes | court (promotion) |
| Synchrone + failover auto | **0** | minutes |

### 2.7 Le managé fait ça pour toi (et toi, tu vérifies)

Rappel du **bloc 6** (RDS) : **Multi-AZ** maintient une copie synchrone dans une autre **zone** et bascule automatiquement — c'est **exactement** cette mécanique, sous le capot. Nuance à connaître : la réplique Multi-AZ est **invisible** (réservée à la bascule, elle ne sert pas les lectures) ; pour lire ailleurs, on provisionne des **Read Replicas** — l'équivalent de notre réplica de lecture.

Ce qui reste **ta responsabilité**, même en managé :

- Les **rôles et autorisations** (Leçon 2) — le cloud ne sait pas qui a le droit d'écrire **chez toi**.
- Les **migrations versionnées** (Leçon 5) — le managé applique le schéma, il ne le conçoit pas.
- Le **test du plan de reprise** : une bascule jamais répétée est une bascule qui échouera.

---

## 📖 Vocabulaire / Abréviations

- **SPOF** (*Single Point of Failure*) : le composant unique dont la panne arrête tout — ici, le serveur de base de données.
- **Réplication** : la copie **en continu** d'un serveur vers un autre.
- **Primary** : le serveur qui accepte les **écritures** (le « chef »).
- **Réplica** (ou *standby*, *slave*) : la copie qui suit le primary et accepte les **lectures** (le « sous-chef »).
- **WAL** (*Write-Ahead Log*) : le journal des modifications, écrit **avant** les données, en segments de 16 Mo — le flux de la réplication.
- **Lag** : le **retard** du réplica par rapport au primary (en octets ou en secondes).
- **Synchrone / asynchrone** : le primary **attend** / **n'attend pas** la confirmation du réplica à chaque écriture.
- **`synchronous_commit`** : le réglage PostgreSQL qui choisit le mode.
- **`pg_is_in_recovery()`** : la fonction qui dit si le serveur est un réplica (`t`) ou le primary (`f`).
- **Promotion** : transformer un réplica en primary (`pg_promote()`).
- **Failover** : la **bascule** automatique vers le réplica quand le primary tombe.
- **Split-brain** : **deux** serveurs qui se croient primary et acceptent des écritures divergentes — le scénario catastrophe.
- **Quorum** : la **majorité** requise pour décider d'une promotion (anti split-brain).
- **Patroni** : l'outil de gestion de grappe avec élection de leader.
- **pg_auto_failover** : l'outil primary/réplica/**moniteur** — le moniteur décide des promotions.
- **RTO** (*Recovery Time Objective*) : la durée maximale **d'interruption** acceptée.
- **RPO** (*Recovery Point Objective*) : la quantité maximale de **données perdues** (en temps) acceptée.
- **Multi-AZ** (bloc 6) : la réplication synchrone **entre zones** chez AWS RDS, avec bascule automatique.
- **Read Replica** (bloc 6) : la réplique de **lecture** chez AWS RDS.
- **Haute disponibilité (HA)** : l'ensemble des dispositifs pour que le service **revienne en minutes**, pas en heures.

---

## 3. Exemples concrets

> 🔁 On enchaîne : les concepts sont posés ; voici les **vérifications et scénarios exacts**, commentés. (La mise en pratique d'une grappe complète passe par des conteneurs — outil enseigné au **Bloc 9** ; ici, tout est pensé pour que tu **comprennes et décides**, pas pour que tu administres une grappe.)

### 3.1 Interroger l'état d'un serveur

```sql
-- « Je suis quoi, moi ? » (à lancer sur n'importe quel serveur)
SELECT pg_is_in_recovery();
-- t → réplica (il « rejoue » le journal) · f → primary (il décide)
```

### 3.2 Voir la réplication depuis le primary

```sql
-- Qui me suit, et avec quel retard ? (côté primary)
SELECT application_name, state, sync_state,
       pg_wal_lsn_diff(pg_current_wal_lsn(), replay_lsn) AS lag_octets
FROM pg_stat_replication;
```

```
 application_name |   state   | sync_state | lag_octets
------------------+-----------+------------+------------
 replica_1        | streaming | async      |          0
```

- `state = streaming` : le flux est **vivant** (le sous-chef rejoue en direct).
- `lag_octets` : le retard en octets de journal — `0` = à jour. Si ce chiffre **grimpe**, ton sous-chef prend du retard (réseau saturé, réplica surchargé).

### 3.3 Choisir son mode : deux métiers, deux choix

```sql
-- Paiement en ligne (RPO = 0 exigé) : le synchrone
ALTER SYSTEM SET synchronous_standby_names = 'replica_1';
SELECT pg_reload_conf();   -- le primary attendra la confirmation du réplica

-- Analytics / logs (performance avant tout) : l'asynchrone
ALTER SYSTEM RESET synchronous_standby_names;
SELECT pg_reload_conf();   -- retour au mode par défaut
```

### 3.4 Le scénario de panne, narré

```text
1. 21h47 — le disque du primary meurt (le chef tombe)
2. L'application tombe en erreur : « impossible de se connecter »
3. Le moniteur (pg_auto_failover) le constate en ~10 s
4. Élection : le réplica est en retard de 0 octet → candidat valide
5. Promotion : le réplica devient primary (pg_promote())
6. Redirection : l'application pointe vers le nouveau primary (DNS/proxy)
7. 21h48 — le service est revenu. RTO ≈ 60 s. RPO = 0 (mode synchrone)
8. Demain — on re-clone une nouvelle réplique depuis le nouveau primary
   (l'ancien serveur est réinitialisé : pg_basebackup, Leçon 4)
```

À comparer avec la **même panne sans réplication** : le service reste mort jusqu'à la **restauration** (backup de la nuit, Leçon 4) — des **heures** de RTO, jusqu'à **24 h** de RPO.

### 3.5 Ce que l'application doit gérer (le pont vers ton stack)

Dans une application Spring, « lire ailleurs » se traduit par **deux sources de données** :

- **Primary** : les écritures (`save`, `update`) **et** les relectures critiques.
- **Réplica** : les lectures « rapport/dashboard » (souvent derrière une *routing datasource* — la répartition lecture/écriture).

Le point que les équipes oublient : le **test de bascule**. Le jour de la panne réelle, la chaîne « détecter → élire → promouvoir → rediriger » ne doit pas être découverte pour la première fois — on la **répète** en conditions contrôlées, et ça se note dans le plan de reprise.

---

## 4. Mise en pratique

> 🧪 **Exercice 6 (livrable : `notes-exercice-06.md`)** — un exercice de **décision** (pas de grappe à monter) : trois scénarios de panne, trois métiers, un plan de reprise à chiffrer. Déroule → `02-exercice.md`, compare → `03-correction.md`, et garde `04-commandes-references.md` (avec le TP Docker optionnel à jouer après le Bloc 9).

Le fil de l'exercice (aperçu) :

1. **Scénario 1** — boutique en ligne, le disque du primary meurt le 29 novembre : chiffre RTO/RPO pour chaque configuration et choisis.
2. **Scénario 2** — intranet de 30 collaborateurs, budget serré : prépare-toi à **justifier un choix différent** du scénario 1.
3. **Scénario 3** — le piège : le réplica qui prend du **lag** pendant le rapport du lundi. Diagnostique (`pg_stat_replication`) et corrige l'architecture applicative.
4. **Synthèse** — rédige le **plan de reprise** d'un des scénarios (RTO, RPO, étapes de bascule, test du plan).

---

## 5. Bonnes pratiques & pièges

### 5.1 À faire

- ✅ **Décider avec les chiffres** : pose le RTO et le RPO **avant** de choisir la technologie (pas l'inverse).
- ✅ **Un réplica qui travaille** : reports et dashboards sur le réplica — c'est lui qui rend la réplication « rentable » à la performance.
- ✅ **Surveiller le lag** (`pg_stat_replication`) : un retard qui grimpe est un **signal d'alerte** avant la panne.
- ✅ **Tester la bascule** (par trimestre) : la promotion, la redirection, puis le **re-clonage** de l'ancien — la chaîne complète.
- ✅ **Garder le backup** (Leçon 4) **avec** la réplication : une **fausse manipulation** (DROP TABLE à 14h02) se **réplique à 14h02** aussi — la réplication ne protège pas de l'erreur humaine, le backup + PITR si.
- ✅ **Écrire le plan de reprise** : RTO/RPO, étapes de bascule, responsable de chaque étape, fréquence de test.

### 5.2 À éviter

- ❌ **Croire que la réplication remplace le backup** : elle copie **tout**, y compris les **erreurs**. Backup + réplication = les deux filets (l'un protège **du passé**, l'autre **le présent**).
- ❌ **Choisir le synchrone « pour être tranquille »** sur une application à forte écriture : chaque `COMMIT` paie un aller-retour. Le choix se **chiffre** (RTO/RPO exigés), pas l'inverse.
- ❌ **Envoyer les utilisateurs vérifier leur écriture sur le réplica** : le piège du lag — la relecture critique reste sur le primary.
- ❌ **Promouvoir sans quorum** (sans moniteur, sans majorité) : la porte ouverte au **split-brain**.
- ❌ **Réintégrer l'ancien primary tel quel** après une bascule : il a manqué tout le flux — **re-clone** obligatoire (`pg_basebackup`).
- ❌ **Croire que le managé (Multi-AZ) te dispense de tout** : rôles (Leçon 2), migrations (Leçon 5) et **test du plan** restent les tiens.

> 💡 **L'analogie du chef et du sous-chef se prolonge** : un sous-chef qui **rejoue fidèlement** une erreur du chef n'est pas un garde-fou — c'est le **miroir** de l'erreur. Le garde-fou contre l'erreur humaine, c'est le **coffre** : le backup d'avant l'erreur (Leçon 4).

---

## 6. Structure (comment tout s'emboîte)

```
LE SERVEUR UNIQUE (Leçons 1-5 : schéma, rôles, config, backups, migrations)
   │
   ├── + RÉPLICATION : primary ──WAL──► réplica (synchrone ou asynchrone)
   │        ├── le réplica LIT (rapports, dashboards) → performance
   │        └── le réplica PEUT ÊTRE PROMU → disponibilité
   │
   ├── + HA : moniteur/quorum → failover AUTOMATIQUE (Patroni / pg_auto_failover)
   │        └── détecter → élire → promouvoir → rediriger (testé, pas improvisé)
   │
   └── + LE MANAGÉ (bloc 6) : Multi-AZ (secours invisible) + Read Replicas
            └── reste à toi : rôles, migrations, TEST du plan
```

**L'idée d'ensemble** : le **backup** répond à la **perte de données** (Leçon 4), la **réplication** répond à la **panne du service** (Leçon 6) — et les deux se complètent : l'une protège **du passé** (restaurer), l'autre protège **le présent** (continuer).

---

## 7. Outils & ressources

- **Patroni** — https://patroni.readthedocs.io (grappe HA avec élection de leader).
- **pg_auto_failover** — https://pg-auto-failover.readthedocs.io (primary/réplica/moniteur).
- Documentation PostgreSQL : chapitre *High Availability, Load Balancing, and Replication*.
- Le **bloc 6** (RDS) pour Multi-AZ et Read Replicas côté managé.
- Ta **Leçon 4** (`04-Backup-et-restauration/01-lecon.md`) : WAL, `pg_basebackup`, PITR — les fondations de la réplication.

---

## 8. Résumé express

- Un seul serveur = **SPOF** ; le backup restaure **après**, il ne rend jamais le service.
- **Réplication primary → réplica** : le primary **écrit**, le réplica **lit** et suit — le **WAL** (segments de 16 Mo) est le flux copié en direct.
- **Asynchrone** = rapide, fin du flux potentiellement perdue ; **synchrone** = écritures confirmées sur les deux, plus lentes (`synchronous_commit`).
- **Lire ailleurs** : rapports et dashboards sur le réplica — avec le piège du **lag** (la relecture critique reste sur le primary).
- **Promotion** (`pg_promote()`) : le réplica devient primary, l'application est redirigée à part, l'ancien primary est **re-cloné**.
- **HA** = failover automatique (Patroni / pg_auto_failover) avec **quorum** — jamais de promotion sans majorité (anti split-brain).
- **RTO** (durée d'interruption) et **RPO** (données perdues) : les deux chiffres d'un plan de reprise — et **backup + PITR** reste le seul filet contre l'erreur humaine répliquée.

---

## 9. Prochaine étape

**Leçon 7 — Cache Redis** : ta base est protégée de la perte (Leçon 4), du schéma sauvage (Leçon 5) et de la panne du serveur (Leçon 6). Reste la **performance en lecture** : quand 80 % des requêtes réclament la même réponse, on la met en **mémoire** — un cache devant la base.