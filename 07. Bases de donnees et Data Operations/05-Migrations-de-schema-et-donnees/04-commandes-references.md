# Commandes & références 5 — Migrations (Flyway, Liquibase, Prisma Migrate)

## 1. Conventions de nommage Flyway (la mémoire des mains)

```
V1__creer_table_emprunts.sql       ← V + numéro + DEUX underscores + description (minuscules, _)
V2__ajoute_colonne_retour.sql
V3__corrige_contrainte_retour.sql
U1__retour_arriere_exceptionnel.sql ← U (undo) : l'école du down-script (rare, à connaître)
```

- **`V`** = version (avant) · **`U`** = undo (rare) · le séparateur est **`__`** (deux underscores).
- Le numéro est l'**ordre d'application**. Jamais deux fichiers du même numéro.
- On **n'édite jamais** un fichier appliqué : le **checksum** (empreinte du contenu, stocké dans la table d'historique) le détecte et bloque.

## 2. Flyway CLI — les trois commandes vitales

```bash
# Appliquer ce qui manque (dans l'ordre) :
flyway -url=jdbc:postgresql://localhost:5432/bibliotheque \
       -user=postgres -password='secret' migrate

# Voir l'état (photo de santé : pending / success / failed) :
flyway -url=... -user=... info

# Vérifier les checksums (avant tout déploiement) :
flyway -url=... -user=... validate
```

Autres commandes utiles : `baseline` (adopter une base existante comme point de départ), `repair` (réaligner le journal après un incident — à utiliser en connaissance de cause), `clean` (⚠️ **détruit tout le schéma** : interdit en prod, désactivable).

## 3. Spring Boot + Flyway (le quotidien)

```xml
<!-- pom.xml : deux dépendances, rien d'autre -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
  <groupId>org.flywaydb</groupId>
  <artifactId>flyway-core</artifactId>
</dependency>
```

- Les fichiers vont dans `src/main/resources/db/migration/` (emplacement par défaut).
- Au démarrage de l'application, Spring **exécute `migrate`** : les migrations font partie du déploiement (pont vers le Bloc 11, CI/CD).
- Configuration éventuelle dans `application.properties` :

```properties
spring.flyway.enabled=true
spring.flyway.baseline-on-migrate=true   # pour adopter une base qui existait avant Flyway
```

## 4. Séquence expand/contract (à coller en tête de migration)

```sql
-- ÉTAPE 1 (expand) : colonne nullable — zéro rupture
ALTER TABLE emprunts ADD COLUMN reste_a_payer NUMERIC(8, 2);
-- ÉTAPE 2 (remplir) : migration de DONNÉES, par lots si la table est grosse
UPDATE emprunts SET reste_a_payer = 0.00 WHERE reste_a_payer IS NULL AND id <= 100000;
-- ÉTAPE 3 (contract) : resserrer une fois rempli
ALTER TABLE emprunts ALTER COLUMN reste_a_payer SET NOT NULL;
```

## 5. Liquibase — situer l'outil (exemple minimal)

```yaml
# changelog.yml — Liquibase décrit les changements dans un fichier dédié (pas du SQL brut)
databaseChangeLog:
  - changeSet:
      id: 2
      author: noor
      changes:
        - addColumn:
            tableName: emprunts
            columns:
              - column: { name: reste_a_payer, type: numeric(8,2) }
      rollback:                     # le « down » est DÉCRIT : l'école du rollback
        - dropColumn:
            tableName: emprunts
            columnName: reste_a_payer
```

Commandes : `liquibase update` (≈ migrate), `liquibase rollback <id>` (le vrai down), `liquibase status`.

## 6. Prisma Migrate — situer l'outil (schéma-as-code)

```prisma
// schema.prisma (écosystème Node) : le schéma vit DANS le code
model Emprunt {
  id           Int       @id @default(autoincrement())
  livreId      Int
  membreId     Int
  emprunteLe   DateTime  @default(now()) @map("emprunte_le")
  resteAPayer  Decimal?  @map("reste_a_payer") @db.Decimal(8, 2)
}
```

```bash
npx prisma migrate dev --name ajoute_colonne_retour   # génère + applique la migration SQL
npx prisma migrate deploy                              # applique en prod (le migrate de Flyway)
```

## 7. Le rituel complet d'une migration en prod (check-list)

```bash
# 1. Backup (Leçon 4) — le filet
sudo -u postgres pg_dump -Fc -d bibliotheque -f "backups/avant-V4-$(date +%F-%H%M).dump"
# 2. Vérifier la santé de l'historique
flyway -url=... -user=... validate
# 3. Appliquer
flyway -url=... -user=... migrate
# 4. Contrôler le résultat (l'objet existe, les droits sont posés)
sudo -u postgres psql -d bibliotheque -c "\d emprunts" -c "\dp emprunts"
```

## 8. Liens croisés du bloc

| Besoin | Où |
|---|---|
| Le backup d'avant-migration | `04-Backup-et-restauration/01-lecon.md` et `04-commandes-references.md` |
| Les `GRANT` à embarquer | `02-Utilisateurs-roles-permissions-et-connexions/04-commandes-references.md` |
| Les connexions (la chaîne JDBC réutilisée ici) | `02-Utilisateurs-roles-permissions-et-connexions/01-lecon.md` |