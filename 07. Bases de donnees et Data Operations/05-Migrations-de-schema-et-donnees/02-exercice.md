# Exercice 5 — Migrer proprement (et corriger en avançant)

> **Objectifs** : versionner un changement de schéma, gérer le cas « NOT NULL sur table pleine », et pratiquer le **forward fix**.
> **Durée** : ~1h · **Prérequis** : Leçons 1-4 faites (base `bibliotheque`, rôles `app_biblio` et `lecteur_biblio`).
> **Livrable** : `notes-exercice-05.md` dans ce dossier.

## Étape 0 — Le rituel

1. **Backup d'abord** (le réflexe de la Leçon 4 — c'est lui qui rend l'exercice sans risque) :

```bash
sudo -u postgres pg_dump -Fc -d bibliotheque -f "backups/avant-migrations-05-$(date +%F).dump"
```

2. Vérifie tes rôles de la Leçon 2 existent toujours : `\du` (tu dois voir `app_biblio` et `lecteur_biblio`).

## Étape 1 — Créer le dossier des migrations

Dans ton projet (ou un dossier dédié `db/migration/`), prépare :

```
db/migration/
├── V1__creer_table_emprunts.sql
├── V2__ajoute_colonne_retour.sql
└── (V3 et V4 viendront plus tard)
```

## Étape 2 — V1 : la table des emprunts (schéma + droits)

Écris `V1__creer_table_emprunts.sql` :

- Table `emprunts` : `id` (IDENTITY PK), `livre_id`, `membre_id` (NOT NULL), `emprunte_le` (TIMESTAMPTZ, défaut `now()`), `rendu_le` (TIMESTAMPTZ **nullable** — un emprunt en cours n'est pas encore rendu).
- Deux **FOREIGN KEY** vers `livres(id)` et `membres(id)`.
- Les `GRANT` : `app_biblio` lit/écrit, `lecteur_biblio` lit seul (rappel Leçon 2 : le principe du moindre privilège).

## Étape 3 — V2 : l'évolution en douceur (l'expand)

Écris `V2__ajoute_colonne_retour.sql` : ajouter la colonne `reste_a_payer NUMERIC(8,2)` — **nullable**. Insère 3-4 emprunts dans la base (donc des lignes avec `reste_a_payer = NULL`).

> ❓ **Question (note-la)** : pourquoi ajouter la colonne **nullable** plutôt que NOT NULL direct ? Quelle conséquence aurait le NOT NULL direct ici ?

## Étape 4 — V3 : l'erreur volontaire

Écris `V3__pas_bonne_idee.sql` avec **uniquement** :

```sql
ALTER TABLE emprunts ALTER COLUMN reste_a_payer SET NOT NULL;
```

Applique (`migrate`). PostgreSQL **refuse** (des lignes NULL existent). Note le message d'erreur exact.

## Étape 5 — V4 : le forward fix (le contract)

**Ne corrige PAS V3.** Écris plutôt `V4__corrige_contrainte_retour.sql` :

1. `UPDATE` de remplissage (les lignes NULL passent à `0.00`).
2. `ALTER TABLE ... SET NOT NULL` (le resserrage, maintenant possible).

Applique. Puis réponds :

- Pourquoi ne pas avoir édité `V3` (pense **checksum** et environnements qui divergent) ?
- Dans quel ordre l'UPDATE et le SET NOT NULL devaient-ils s'exécuter, et pourquoi ?

## Étape 6 — La santé et le bilan

```bash
flyway -url=jdbc:postgresql://localhost:5432/bibliotheque -user=postgres -password='...' info
flyway -url=jdbc:postgresql://localhost:5432/bibliotheque -user=postgres -password='...' validate
```

- Note l'état de **chaque** migration dans `info` (y compris l'état de V3 : failed ?).
- Que fait `validate` ? Pourquoi est-il précieux **avant** un déploiement ?

## Étape 7 — Le bilan (dans `notes-exercice-05.md`)

1. La liste de tes fichiers, dans l'ordre, avec l'état final de chacun.
2. Les messages d'erreur de l'étape 4 (copiés-collés).
3. Tes réponses aux questions des étapes 3, 5 et 6.
4. **En une phrase** : pourquoi « la source de vérité du schéma est Git, pas la production » ?

> 🧭 **Astuce anti-frustration** : si tu n'as pas de CLI Flyway sous la main, tu peux **simuler** l'outillage en appliquant les fichiers à la main dans l'ordre et en notant dans une table `ma_schema_history(version, fichier, applique_le)` — tu recodes le principe de Flyway, ce qui vaut mieux que de le subir.