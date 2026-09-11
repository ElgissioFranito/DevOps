# Exercice — Leçon 4 : Backup et restauration

> **Bloc 7 · Leçon 4** — Exercice en autonomie, **100 % en local**. Tu vas réellement **casser** ta base puis la **réparer** : c'est l'exercice le plus important du bloc — et comme tout se fait **chez toi**, le risque est zéro.
>
> 🔁 **Comment s'articulent les fichiers** : lis `01-lecon.md` (théorie + vocabulaire), fais cet exercice **sans** regarder la solution, puis compare avec `03-correction.md`. Aide-mémoire : `04-commandes-references.md`.

---

## Contexte

La base `bibliotheque` contient désormais la table `livres` (Leçon 1) et les rôles `app_biblio` / `lecteur_biblio` (Leçon 2). Demain, tu dois **préparer le fichier `membres`** (la liste des emprunteurs) — et, en professionnel, tu refuses d'écrire une table importante **sans filet**. Plan : **sauvegarder → vérifier le backup → (horreur) subir la panne → restaurer → vérifier → remettre en service**.

---

## Énoncé

### Étape 1 — Préparer les données à protéger

```bash
mkdir -p backups        # crée le dossier des sauvegardes, RELATIF au projet (-p : ne râle pas s'il existe déjà)
```

Dans une session admin (`sudo -u postgres psql`), sur `bibliotheque` :

```sql
CREATE TABLE membres (
  id    GENERATED ALWAYS AS IDENTITY PRIMARY KEY,   -- auto-numéroté (Leçon 1)
  nom   VARCHAR(120) NOT NULL,
  email VARCHAR(200) NOT NULL UNIQUE                -- UNIQUE : deux lignes ne peuvent pas avoir le même email
);

INSERT INTO membres (nom, email) VALUES
  ('Amina Diallo',  'amina@example.com'),
  ('Jean Martin',   'jean@example.com'),
  ('Sofia Rossi',   'sofia@example.com'),
  ('Luc Bernard',   'luc@example.com'),
  ('Nour Haddad',   'nour@example.com');

SELECT count(*) FROM membres;      -- attendu : 5
```

### Étape 2 — Le backup complet (+ les rôles !)

1. Sauvegarde la **base** en format custom avec la **date du jour** dans le nom :
   ```bash
   sudo -u postgres pg_dump -Fc -d bibliotheque -f "backups/bibliotheque-$(date +%F).dump"
   # -Fc : format custom compressé ; -d : la base ; -f : le fichier destination ;
   # $(date +%F) : substitution shell qui insère la DATE du jour (AAAA-MM-JJ)
   ```
2. Sauvegarde les objets **globaux** (les rôles) :
   ```bash
   sudo -u postgres pg_dumpall --globals-only -f "backups/globals-$(date +%F).sql"
   # --globals-only : uniquement les rôles et droits globaux (pas les données des bases)
   ```
3. Vérifie les fichiers : `ls -lh backups` (l'option `-l` : liste détaillée ; `-h` : tailles lisibles). Note leurs **tailles** dans tes notes.
4. **Jette un œil au contenu du dump** sans le restaurer :
   ```bash
   pg_restore -l "backups/bibliotheque-$(date +%F).dump" | head -20
   # -l : LISTE le contenu de l'archive (tables, index, ACL...) ; head -20 : garde les 20 premières lignes
   ```
   Repère la mention **`ACL`** (Access Control List — les droits) : c'est la preuve que les `GRANT` de la Leçon 2 sont **dans le dump**.

### Étape 3 — Simuler la panne (séquence : d'abord en sécurité, puis pour de vrai)

1. **Le test sans risque** (rappel de la Leçon 1) :
   ```sql
   -- (1) Ouvre une transaction : tout ce qui suit peut être annulé avec ROLLBACK.
   BEGIN;
   -- (2) LANCE la mauvaise commande (sans le WHERE).
   DELETE FROM membres;
   -- (3) VÉRIFIE le désastre : 0 ligne restante.
   SELECT count(*) FROM membres;
   -- (4) ANNULE tout.
   ROLLBACK;
   -- (5) PREUVE : tes 5 lignes sont toujours là.
   SELECT count(*) FROM membres;
   -- SANS le BEGIN, la suppression s'appliquerait DIRECTEMENT (COMMIT automatique) :
   -- c'est normal, ce n'est pas un piège — c'est le comportement de base de SQL.
2. **Simule la panne** (chez toi, sans risque — c'est le but de l'exercice) :
   ATTENTION : cette commande est DÉFINITIVE (pas de ROLLBACK possible ici —
   on est hors transaction, donc autocommit : chaque commande s'applique aussitôt).
   ```sql
   -- (1) D'abord, RELIS ton dump : pg_restore -l "backups/bibliotheque-$(date +%F).dump"
   --     (vérifie que les lignes TABLE membres / TABLE DATA membres y sont BIEN).
   -- (2) Ensuite seulement, casse :
   DROP TABLE membres;                     -- la table disparaît AVEC ses données
   SELECT count(*) FROM membres;           -- ❌ « ERROR: relation "membres" does not exist »
   ```
   > ⚠️ **Note le silence** : aucun avertissement, pas de corbeille, pas de Ctrl+Z. C'est **exactement** pour cela qu'on sauvegarde.

### Étape 4 — Restaurer (d'abord à côté, puis en place)

1. Restaure **dans une base de test** (on ne manipule jamais « en production » d'abord) :
   ```bash
   sudo -u postgres psql -c 'CREATE DATABASE bibliotheque_restaure;'   # -c : exécute UNE commande SQL puis quitte
   sudo -u postgres pg_restore -d bibliotheque_restaure "backups/bibliotheque-$(date +%F).dump"
   # -d : la base cible de la restauration
   ```
2. **Vérifie par les comptages** (la preuve) :
   ```sql
   -- dans bibliotheque_restaure :
   SELECT count(*) FROM membres;    -- attendu : 5 — LES MÊMES
   SELECT count(*) FROM livres;     -- attendu : le même nombre que dans bibliotheque
   ```
3. **Remets la vraie base en service** :
   ```bash
   sudo -u postgres psql -c 'DROP DATABASE bibliotheque;'        # efface la base cassée
   sudo -u postgres psql -c 'CREATE DATABASE bibliotheque;'      # la recrée vide
   sudo -u postgres pg_restore -d bibliotheque "backups/bibliotheque-$(date +%F).dump"
   ```
4. **Reconnecte-toi avec le rôle de l'application** (la vraie vérification) :
   ```bash
   psql -h localhost -U app_biblio -d bibliotheque
   ```
   Teste : `SELECT count(*) FROM livres;` (✅ fonctionne — les **GRANT des tables** sont revenus avec le dump). Puis teste **avant** de continuer : `psql -h localhost -U lecteur_biblio -d bibliotheque` → si la connexion est **refusée**, note pourquoi (aide : le `GRANT CONNECT` de la Leçon 2 portait sur la **base** — et quel outil sauvegarde les droits sur les bases ?).

### Étape 5 — Réfléchir (dans `notes-exercice-04.md`)

1. Pourquoi le dump de la base **seul** n'a pas suffi à tout remettre en service ? (le lien avec `pg_dumpall`)
2. Pourquoi la restauration **à côté** (dans `bibliotheque_restaure`) avant la mise en place ?
3. Quelle **fréquence** de backup choisirais-tu pour ce projet, et quel **RPO** correspond ?
4. Où rangerais-tu les copies de `backups/` pour respecter le 3-2-1 (nomme 2 emplacements concrets) ?

---

## Livrable

**`notes-exercice-04.md`** : les commandes de sauvegarde (+ tailles des fichiers), l'extrait `pg_restore -l` avec la mention `ACL`, le moment de la panne (le message d'erreur exact), les comptages de vérification, le résultat du test `app_biblio`/`lecteur_biblio`, et les 4 réponses de l'étape 5.

Correction détaillée dans **`03-correction.md`**. Aide-mémoire : **`04-commandes-references.md`**.
