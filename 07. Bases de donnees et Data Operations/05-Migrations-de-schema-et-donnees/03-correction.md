# Correction 5 — Migrer proprement (et corriger en avançant)

> Compare **ta démarche** à celle-ci. Si tu as fait différemment mais que tu peux **justifier**, c'est bon signe — l'important est la logique versionnée, pas le copier-coller.

## Étape 0 — Le rituel

```bash
sudo -u postgres pg_dump -Fc -d bibliotheque -f "backups/avant-migrations-05-2025-01-15.dump"
# Backup horodaté AVANT de toucher au schéma : sans lui, l'exercice n'est plus « sans risque »
```

`\du` doit montrer `app_biblio` (rôle applicatif) et `lecteur_biblio` (lecture seule) — les rôles de la Leçon 2.

## Étape 2 — V1 corrigée (avec les pièges évités)

```sql
-- V1__creer_table_emprunts.sql
CREATE TABLE emprunts (
  id          GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  livre_id    INT NOT NULL,
  membre_id   INT NOT NULL,
  emprunte_le TIMESTAMPTZ NOT NULL DEFAULT now(),
  rendu_le    TIMESTAMPTZ,                              -- nullable par CHOIX métier (emprunt en cours)
  CONSTRAINT fk_emprunt_livre  FOREIGN KEY (livre_id)
    REFERENCES livres(id) ON DELETE RESTRICT,           -- RESTRICT : refuse de supprimer un livre emprunté
  CONSTRAINT fk_emprunt_membre FOREIGN KEY (membre_id)
    REFERENCES membres(id) ON DELETE RESTRICT
);

GRANT SELECT, INSERT, UPDATE ON ALL TABLES IN SCHEMA public TO app_biblio;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO lecteur_biblio;
-- Les droits VIVENT dans la migration : sans eux, le déploiement ne fonctionne pas (Leçon 2)
```

> 💡 Les pièges évités ici : `SERIAL` à la place de `IDENTITY` (l'ancienne syntaxe — le gabarit du Bloc 7 impose `IDENTITY`), et l'oubli des `GRANT` (le déploiement « marche chez le dev, casse en prod »).

## Étape 3 — V2 corrigée (l'expand)

```sql
-- V2__ajoute_colonne_retour.sql
ALTER TABLE emprunts ADD COLUMN reste_a_payer NUMERIC(8, 2);
```

Insertion de quelques emprunts (donc des lignes avec `reste_a_payer = NULL`).

**Réponse attendue à la question** : ajouter **nullable** est sans danger — les 10 000 lignes existantes reçoivent `NULL` et rien ne casse. Le NOT NULL **direct** aurait **refusé** l'`ALTER` (des lignes existantes violeraient la contrainte) — c'est exactement l'erreur que l'étape 4 provoque volontairement.

## Étapes 4-5 — L'erreur, puis le forward fix

Message attendu à l'étape 4 :

```
ERROR:  column "reste_a_payer" of relation "emprunts" contains null values
```

La correction **en avançant** (V4, jamais en éditant V3) :

```sql
-- V4__corrige_contrainte_retour.sql
UPDATE emprunts SET reste_a_payer = 0.00 WHERE reste_a_payer IS NULL;
-- 1) REMPLIR : les lignes existantes reçoivent une valeur métier (0 = rien à payer)

ALTER TABLE emprunts ALTER COLUMN reste_a_payer SET NOT NULL;
-- 2) CONTRACT : resserrer la contrainte — possible car plus aucune ligne NULL
```

**Réponses attendues** :
- **Pourquoi ne pas éditer V3 ?** Le **checksum** enregistré dans `flyway_schema_history` ne correspondrait plus : `validate` bloquerait, et les environnements (dev déjà migré avec « l'ancien V3 ») **divergent** de ceux qui prendraient le nouveau V3. La règle : l'historique est figé, on corrige en `V(n+1)`.
- **L'ordre** : l'`UPDATE` **d'abord** (remplir), le `SET NOT NULL` **ensuite** (resserrer). Dans l'autre ordre, le refus de l'étape 4 se reproduirait.

## Étape 6 — La santé

- `info` montre V3 en **Failed** (ou l'état d'erreur de ta version Flyway) et V4 en **Success** : l'historique **raconte l'histoire**, y compris l'échec — c'est une trace, pas une honte.
- `validate` **recalcule les checksums** de tous les fichiers appliqués et les compare au journal : il détecte **toute modification après coup**. Avant un déploiement, c'est le garde-fou qui empêche de partir avec un historique corrompu.

## Étape 7 — Le bilan attendu (grille d'auto-évaluation)

| Critère | C'est réussi si... |
|---|---|
| Versionnement | Tes 4 fichiers existent, numérotés, dans `db/migration/`, nommés `Vn__description_minuscules.sql` |
| Ordre logique | V1 (schéma+droits) → V2 (expand) → V4 (remplir puis contract) ; **aucune édition** de fichier appliqué |
| Compréhension NOT NULL | Tu sais expliquer pourquoi nullable d'abord, et quel refus produit le NOT NULL direct |
| Forward fix | Tu as corrigé **en avançant** et tu sais citer le rôle du **checksum** |
| Santé | Tu as lancé `info` et `validate`, et tu sais dire ce que chacun apporte |
| Pont Leçon 4 | Le backup d'Étape 0 existe et tu l'as cité dans ton bilan |
| La phrase finale | Quelque chose comme : « Le schéma est défini par les fichiers versionnés ; la prod ne fait que les recevoir. » |

## Les pièges classiques (auto-diagnostic)

1. **Avoir édité V3** pour « le réparer » → le checksum diverge → `validate` rouge. (Si tu l'as fait : recrée un V4 propre et remets le fichier V3 dans son état d'origine — exercice bonus de discipline.)
2. **`SET NOT NULL` avant l'`UPDATE`** → même refus qu'en V3.
3. **Oublier les `GRANT`** dans V1 → l'app ne voit pas sa table (déploiement cassé, Leçon 2).
4. **Renommer sans migrer** : si tu avais renommé une colonne, la migration de **données** (recopier les valeurs) devait accompagner le `ALTER TABLE ... RENAME`.