# Leçon 5 — Migrations de schéma et de données

> **Bloc 7 · Bases de données & Data Operations** — Leçon 5 sur 8
> 🧭 **Pont depuis la Leçon 4** : tu sais maintenant protéger tes données (backup, restauration testée). La roadmap exige ici : *« migration de schéma, migration de données, versionnement, rollback, outils (Flyway, Liquibase, Prisma Migrate) »*. Le lien est direct : la **migration** est l'opération qui fait **peur** aux équipes sans backup — car elle **modifie la structure** de la base en production. Tu vas apprendre à la faire **proprement, versionnée dans Git, répétable et réversible**. Chaque migration sérieuse commence par... un backup (tu sais pourquoi et comment).

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Expliquer** pourquoi un script SQL exécuté « à la main » en production est une bombe à retardement.
2. **Comprendre** le principe de la migration **versionnée** (fichiers ordonnés, journal des applications, reproductibilité).
3. **Écrire** des migrations avec **Flyway** (l'outil du cœur de la leçon — stack Java/Spring) et savoir ce que sont **Liquibase** et **Prisma Migrate**.
4. **Distinguer** migration de **schéma** (la structure) et migration de **données** (le contenu).
5. **Construire** un **rollback** : le plan de retour en arrière (et savoir quand le « forward fix » est la vraie réponse).
6. **Relier** le tout : backup avant migration (Leçon 4), versionnement Git (Blocs 1 et 4).

---

## 2. Explication simple

### 2.1 Le « pourquoi » : le cauchemar des scripts à la main

Chaque développeur a vécu cette scène : la nouvelle table existe **chez lui**, pas **chez le collègue**, et **pas du tout** en production.

```
Le script « à la main » :
  dev 1 → exécute création-employes.sql chez lui        ✓
  dev 2 → l'exécute aussi, mais un mois plus tard       ✓ (déjà la moitié des objets → erreurs)
  production → exécuté vendredi 18h par... quelqu'un    ✓ (et personne ne sait exactement ce qui a tourné)
```

> 💡 **Analogie** : faire évoluer une base avec des scripts à la main, c'est **rénover une maison** en donnant à chaque artisan une note post-it différente : à force, personne ne sait quels murs ont bougé. La migration versionnée, c'est le **permis de construire daté et archivé** : chaque modification est un dossier numéroté, rangé, rejouable dans l'ordre.

La solution : chaque changement de la base est un **fichier versionné dans Git**, portant un **numéro d'ordre**. Un outil (Flyway) les joue **dans l'ordre, une seule fois chacun**, et note dans une table interne ce qui a été appliqué. Résultat : sur une base neuve ou à 3 ans d'écart, exécuter les migrations donne **exactement le même schéma**.

### 2.2 Le « comment » : le principe Flyway

**Flyway** (l'outil le plus répandu de l'écosystème Java/Spring — ton stack) fonctionne sur trois règles simples :

1. **Nommer** chaque fichier : `V1__description.sql`, `V2__description.sql`... (`V` = version, **deux underscores**, une description en minuscules et sans espaces — la convention est stricte).
2. **Appliquer** : la commande `migrate` joue les fichiers **non encore appliqués**, dans l'ordre des numéros.
3. **Journaliser** : Flyway crée et maintient une table interne (`flyway_schema_history`) qui note : quel fichier, quand, avec quel **checksum** (une empreinte du contenu — le même mécanisme d'empreinte que le `scram-sha-256` de la Leçon 2, mais appliqué au fichier).

Conséquence du checksum : si tu **modifies un fichier déjà appliqué**, Flyway le détecte (l'empreinte ne correspond plus) et **refuse de continuer**. C'est une protection : l'historique ne se réécrit pas.

### 2.3 Schéma vs données : deux migrations différentes

La roadmap distingue les deux — c'est une distinction de **risque** :

| | Migration de **schéma** | Migration de **données** |
|---|---|---|
| Elle touche | la **structure** (tables, colonnes, index) | le **contenu** (les lignes) |
| Exemple | ajouter la colonne `email` à `membres` | renommer tous les « M. Dupont » en « Dupont » |
| Elle est | **réversible en théorie** (on peut la refaire à l'envers) | **irréversible en pratique** (l'ancienne valeur est perdue) |
| Le réflexe | la versionner (Flyway) | la versionner **et** tester sur une copie (Leçon 4 !) |

**Le cas mixte redouté** : ajouter une colonne NOT NULL **sur une table déjà pleine**. La bonne séquence moderne (« expand/contract », la double écriture) :

```
1. EXPAND   : ajouter la colonne NULLABLE          (aucune ligne cassée)
2. REMPLIR  : migration de données (ligne par ligne, par lots)
3. CONTRACT : resserrer en NOT NULL une fois remplie
```

Un `ALTER TABLE ... ADD COLUMN ... NOT NULL` direct sur une table chargée, c'est le plan qui bloque l'application en pleine journée.

### 2.4 Le « quand » : le rollback et le « forward fix »

**Rollback** = le plan de retour en arrière. Deux écoles :

- **Down script** (l'école Liquibase, et Prisma Migrate qui génère des `down`) : chaque migration a sa **jumelle inverse** (`V2__ajoute_colonne.sql` ↔ down : `DROP COLUMN`). Clair en théorie, fragile en pratique : le « down » de la migration de données est souvent **impossible à écrire honnêtement** (les anciennes valeurs n'existent plus).
- **Forward fix** (l'école Flyway) : on **n'annule pas**, on **corrige en avançant** — `V3__correction.sql`. La base ne recule jamais, l'historique reste linéaire. C'est la pratique dominante 2025-2026.

> 💡 **Analogie** : le rollback « down », c'est reculer la voiture dans un tunnel ; le forward fix, c'est sortir par la sortie suivante et faire un demi-tour propre. À vitesse d'incident, la sortie suivante est presque toujours plus sûre.

Et le vrai filet de l'incident grave reste la Leçon 4 : **restaurer le backup d'avant-migration**. Le rollback SQL corrige une migration ratée ; le backup récupère un désastre.

### 2.5 Les trois outils de la roadmap

| Outil | Écosystème | Idée clé | Ton niveau |
|---|---|---|---|
| **Flyway** | Java/Spring (ton stack) | fichiers SQL `V1__nom.sql` + table d'historique | ✅ **Le cœur de la leçon** |
| **Liquibase** | Java, multi-bases | un fichier **changelog** (XML/YAML/SQL) qui décrit les changements + génère les « rollback » | ✅ Savoir le situer |
| **Prisma Migrate** | Node/Next.js/NestJS | le schéma est décrit **dans le code** (le modèle Prisma), l'outil **génère** les migrations SQL | ✅ Savoir le situer |

> 💡 **Pourquoi apprendre l'outil de « ton » écosystème et citer les autres ?** Parce que le **concept** est le même partout : des changements numérotés, appliqués une seule fois, journalisés, rejouables. Une fois Flyway maîtrisé, Liquibase et Prisma se lisent en 10 minutes — tu comprends la mécanique, pas seulement la syntaxe.

---

## 📖 Vocabulaire / Abréviations

- **Migration** : une modification **contrôlée et versionnée** de la structure ou des données de la base.
- **Migration de schéma** : qui touche la structure (tables, colonnes, index).
- **Migration de données** : qui touche le contenu (les lignes) — irréversible en pratique.
- **Versionnement** : chaque changement porte un numéro d'ordre et vit dans Git.
- **Reproductible** : rejouer toutes les migrations donne **exactement le même schéma**, sur toute machine.
- **Flyway** : l'outil qui applique les migrations SQL numérotées (`V1__...`) et les journalise.
- **`flyway_schema_history`** : la table interne de Flyway (quel fichier, quand, quel checksum).
- **Checksum** : l'empreinte du contenu d'un fichier — détecte toute modification d'une migration déjà appliquée.
- **`migrate`** : la commande qui applique les migrations manquantes, dans l'ordre.
- **`info` / `validate`** : voir l'état des migrations / vérifier les checksums (la santé).
- **Rollback** : le plan de retour en arrière.
- **Down script** : la migration jumelle inverse (l'école Liquibase/Prisma).
- **Forward fix** : corriger en avançant (`V3__correction.sql`) — l'école Flyway.
- **Expand/contract** : la séquence sécurisée pour ajouter une contrainte sur une table pleine (expand → remplir → contract).
- **NULLABLE / NOT NULL** : colonne qui accepte / refuse la valeur vide.
- **Liquibase** : l'outil à **changelog** (XML/YAML/SQL) avec rollback générés.
- **Prisma Migrate** : l'outil de l'écosystème Node où le schéma vit dans le code.
- **ACL** (rappel Leçon 4) : les droits sur les objets — qui doivent accompagner la migration (les `GRANT` de la Leçon 2).
- **Environnement** : dev / test / prod — des bases **séparées** (rappel Leçon 1), migrées par les mêmes fichiers.

---

## 3. Exemples concrets

> 🔁 On enchaîne : le principe est posé ; voici **les fichiers et les commandes exacts**, commentés ligne par ligne. Tout est local et gratuit.

### 3.1 L'arborescence d'un projet migré

```
mon-projet/
├── src/main/resources/db/migration/     ← l'emplacement par défaut de Spring Boot pour Flyway
│   ├── V1__creer_table_emprunts.sql     ← première migration (le schéma initial)
│   ├── V2__ajoute_colonne_retour.sql    ← évolution suivante
│   └── V3__corrige_contrainte_retour.sql← le « forward fix »
└── notes-exercice-05.md                 ← tes notes (livrable)
```

### 3.2 Trois migrations commentées

**`V1__creer_table_emprunts.sql`** (migration de schéma) :

```sql
CREATE TABLE emprunts (
  id         GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  livre_id   INT NOT NULL,
  membre_id  INT NOT NULL,
  emprunte_le TIMESTAMPTZ NOT NULL DEFAULT now(),
  rendu_le    TIMESTAMPTZ,                       -- NULL = pas encore rendu (choix métier)
  CONSTRAINT fk_emprunt_livre  FOREIGN KEY (livre_id)  REFERENCES livres(id),
  CONSTRAINT fk_emprunt_membre FOREIGN KEY (membre_id) REFERENCES membres(id)
  -- FOREIGN KEY : la clé étrangère de la Leçon 1 — le SGBD REFUSE un emprunt d'un livre inexistant
);

-- Les droits : une migration qui crée une table DOIT les penser (Leçon 2)
GRANT SELECT, INSERT, UPDATE ON ALL TABLES IN SCHEMA public TO app_biblio;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO lecteur_biblio;
```

**`V2__ajoute_colonne_retour.sql`** (schéma, en douceur — pas de NOT NULL brutal) :

```sql
ALTER TABLE emprunts ADD COLUMN reste_a_payer NUMERIC(8, 2);
-- NULLABLE d'abord (expand) : les 10 000 lignes existantes ne sont pas cassées
```

**`V3__corrige_contrainte_retour.sql`** (le forward fix — on corrige en avançant) :

```sql
UPDATE emprunts SET reste_a_payer = 0.00 WHERE reste_a_payer IS NULL;
ALTER TABLE emprunts ALTER COLUMN reste_a_payer SET NOT NULL;
-- Remplir (par migration de données) PUIS resserrer (contract) : la séquence expand/contract de la leçon
```

### 3.3 Exécuter avec Flyway (CLI — ou via Spring Boot)

```bash
flyway -url=jdbc:postgresql://localhost:5432/bibliotheque \
       -user=postgres -password='...' migrate
# flyway : l'outil CLI ; -url : la chaîne de connexion (jdbc = le pilote Java, format vu avec Spring) ;
# migrate : applique les migrations MANQUANTES, dans l'ordre — jamais deux fois les mêmes

flyway -url=... -user=... info
# info : l'état des migrations (pending / success / failed) — ta photo de santé

flyway -url=... -user=... validate
# validate : vérifie les CHECKSUMS — un fichier modifié après coup est repéré ICI
```

Sortie attendue (simplifiée) :

```
Schema "public" is up to date. No migration necessary.
| Version | Description             | Type | State   |
| 1       | creer table emprunts    | SQL  | Success |
| 2       | ajoute colonne retour   | SQL  | Success |
| 3       | corrige contrainte      | SQL  | Pending |   ← le prochain migrate l'appliquera
```

**Via Spring Boot** (le quotidien réel) : ajouter `flyway-core` au `pom.xml`, poser les fichiers dans `db/migration/`, et l'application **migrera à son démarrage** — les migrations deviennent une étape du déploiement (le pont vers le Bloc 11, CI/CD).

### 3.4 Le rituel d'avant-migration (le réflexe Leçon 4)

```bash
sudo -u postgres pg_dump -Fc -d bibliotheque -f "backups/avant-V3-$(date +%F).dump"
# LA migration sérieuse commence ICI — le backup qui permet de tout récupérer si V3 tourne mal
```

---

## 4. Mise en pratique

> 🧪 **Exercice 5 (livrable : `notes-exercice-05.md`)** — sur ta base `bibliotheque`, tu vas **migrer proprement**, puis **corriger une erreur par forward fix**. Déroule l'exercice → `02-exercice.md`, puis compare ta démarche à la correction → `03-correction.md`, et garde `04-commandes-references.md` sous la main.

Le fil de l'exercice (aperçu) :

1. **Backup d'abord** (le rituel Leçon 4 — automatique maintenant, n'est-ce pas ?).
2. Écris `V1__creer_table_emprunts.sql` (table + FK + `GRANT` aux rôles de la Leçon 2).
3. Écris `V2__ajoute_colonne_retour.sql` (colonne NULLABLE — l'« expand »).
4. **Simule l'erreur** : `V3` avec un `ALTER ... SET NOT NULL` direct (des lignes NULL existent) → observe le refus net de PostgreSQL.
5. **Corrige en avançant** : `V4` = `UPDATE` de remplissage + `SET NOT NULL` (le « contract ») — le forward fix.
6. Documente : **ce que `flyway info` / `validate` t'ont appris**, et **pourquoi on n'édite jamais une migration appliquée**.

---

## 5. Bonnes pratiques & pièges

### 5.1 À faire

- ✅ **Backup AVANT chaque migration en prod** (Leçon 4) — non négociable, testé.
- ✅ **De petites migrations** : un fichier = un changement réversible en tête. Facile à relire, facile à rejouer.
- ✅ **Numéros croissants et uniques** : `V1`, `V2`, `V3`... jamais deux fichiers du même numéro.
- ✅ **Les `GRANT` vivent DANS les migrations** (Leçon 2) : un schéma sans droits n'est pas déployable.
- ✅ **Tester chaque migration sur une copie** (dev/une base de test) avant la prod.
- ✅ **`flyway validate` dans la CI** (Bloc 11) : les checksums sont vérifiés **avant** le déploiement.
- ✅ **Documenter le rollback** attendu dans le commentaire d'en-tête du fichier, même si l'école est « forward fix ».

### 5.2 À éviter (les pièges classiques)

- ❌ **Éditer une migration déjà appliquée** : le checksum ne correspond plus, Flyway bloque — et pire, les environnements divergent. **Corrige en avançant** (`V(n+1)`).
- ❌ **Exécuter un script à la main en prod** « juste cette fois » : c'est exactement le scénario du 2.1 qui recommence.
- ❌ **`ADD COLUMN ... NOT NULL` direct sur une table pleine** : la séquence expand → remplir → contract existe pour ça.
- ❌ **Migration de données « en un seul gros UPDATE »** sur des millions de lignes : découpe par lots (`WHERE id BETWEEN ...`), ou la transaction bloque tout.
- ❌ **Oublier les données** lors d'un changement de structure (renommer une colonne sans migrer ses valeurs).
- ❌ **Une migration non testée sur une copie** : la prod n'est pas ton bac à sable.
- ❌ **Pas de plan de rollback écrit** AVANT d'appliquer : le jour de l'incident, on improvise mal.

> 💡 **L'analogie du permis de construire** revient : la migration versionnée est ton permis archivé ; l'éditer après coup, c'est gratter sur le permis pendant que l'inspecteur regarde.

---

## 6. Structure (comment tout s'emboîte)

```
GIT (le versionnement, Blocs 1 et 4)
   └── db/migration/V1..Vn   ← la SOURCE DE VÉRITÉ du schéma (pas la prod !)
          │
          ├── Flyway (migrate/info/validate)   ← le moteur qui applique et journalise
          │      └── flyway_schema_history      ← le journal interne (checksums)
          │
          ├── Leçon 2 : les GRANT qui accompagnent chaque table créée
          ├── Leçon 4 : le backup pg_dump AVANT chaque migration
          └── Bloc 11 (CI/CD) : validate + migrate au déploiement
```

**L'idée d'ensemble** : la **source de vérité du schéma n'est plus la base de production** — c'est le dossier Git des migrations. La prod n'est que le **dernier environnement à recevoir** la vérité, jamais celui qui la définit.

---

## 7. Outils & ressources

- **Flyway** — https://flywaydb.org (documentation, CLI, intégration Spring Boot).
- **Liquibase** — https://www.liquibase.org (changelog, rollback générés).
- **Prisma Migrate** — https://www.prisma.io/migrate (schéma-as-code, écosystème Node).
- Spring Boot + Flyway : `spring-boot-starter-data-jpa` + `flyway-core` (dépendance unique, emplacement `db/migration/`).
- Ta **Leçon 4** (`04-Backup-et-restauration/01-lecon.md`) pour le rituel de backup.
- Ta **Leçon 2** (`02-Utilisateurs-roles-permissions-et-connexions/01-lecon.md`) pour les `GRANT` à embarquer dans les migrations.

---

## 8. Résumé express

- Une migration = un **fichier SQL versionné dans Git**, numéroté (`V1__nom.sql`), appliqué **une seule fois**, journalisé (table `flyway_schema_history` + **checksums**).
- **Schéma ≠ données** : la migration de données est **irréversible en pratique** → backup avant, test sur copie.
- **NOT NULL sur table pleine** = séquence **expand → remplir → contract**, jamais le `ALTER` brutal.
- **Rollback** : l'école Flyway = **forward fix** (on corrige en avançant) ; le backup (Leçon 4) reste le filet du désastre.
- **On n'édite jamais une migration appliquée** — c'est le checksum qui le rappelle, et la règle d'or de la discipline.
- Outils : **Flyway** (ton stack Java/Spring — maîtrisé), **Liquibase** et **Prisma Migrate** (situés : même concept, autre syntaxe).
- La **source de vérité du schéma est Git**, pas la production.

---

## 9. Prochaine étape

**Leçon 6 — Réplication et haute disponibilité** : tes données sont maintenant versionnées, sauvegardées, restaurables. Reste la **panne du serveur lui-même** : un seul PostgreSQL = un point de défaillance unique. On va apprendre à en avoir **deux qui copient en direct** — et à basculer quand l'un tombe.