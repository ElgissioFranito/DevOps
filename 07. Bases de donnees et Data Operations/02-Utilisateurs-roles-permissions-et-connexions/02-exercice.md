# Exercice — Leçon 2 : Utilisateurs, rôles, permissions et connexions

> **Bloc 7 · Leçon 2** — Exercice en autonomie, **100 % en local** (PostgreSQL du Bloc 2). Aucun coût.
>
> 🔁 **Comment s'articulent les fichiers** : tu as lu `01-lecon.md` (théorie + vocabulaire). Fais **cet exercice sans regarder la solution**, puis compare avec `03-correction.md` (pas à pas + checklist réécrite). L'aide-mémoire `04-commandes-references.md` reste à côté.

---

## Contexte

La base `bibliotheque` de la Leçon 1 fonctionne. Demain, **deux « clients »** vont s'y connecter :

- **l'application Spring Boot** (elle doit lire et écrire) — un rôle nommé `app_biblio` ;
- **un collègue analyste** (il ne doit **que lire**, jamais modifier) — un rôle nommé `lecteur_biblio`.

Ton travail : créer ces rôles, **accorder exactement les bons droits**, vérifier **par le test** que chaque rôle peut faire ce qu'il doit — et **rien** de plus.

> ⚠️ **Prérequis** : la base `bibliotheque` avec sa table `livres` existe (Leçon 1). Si besoin, refais l'exercice 1.

---

## Énoncé

### Étape 1 — Créer les rôles (en tant qu'administrateur)

```bash
sudo -u postgres psql       # se connecte en tant qu'administrateur local (« postgres »)
```

1. Crée le rôle **`app_biblio`** : droit de connexion (`LOGIN`), mot de passe au **format « phrase de passe »** (ex. `Biblio-App-2026-moulin!coffre`), et **aucun pouvoir d'administration** (`NOSUPERUSER NOCREATEDB NOCREATEROLE`).
2. Crée le rôle **`lecteur_biblio`** : même principe, autre phrase de passe.
3. Liste les rôles avec `\du` et repère la colonne « Attributs » : elle doit être **vide de pouvoirs** (`SUPERUSER`, `Créer une base`... ne doivent pas apparaître).

### Étape 2 — Accorder les permissions

1. **À `app_biblio`** :
   - le droit de se connecter à la base `bibliotheque` (`CONNECT`) ;
   - le droit d'utiliser le schéma `public` (`USAGE`) ;
   - la lecture, l'ajout, la modification, la suppression **des tables existantes** (`SELECT, INSERT, UPDATE, DELETE`) ;
   - la même chose **pour les tables créées plus tard** (`ALTER DEFAULT PRIVILEGES`) ;
   - le droit d'utiliser les **séquences** (`USAGE ON SEQUENCES` — nécessaires aux colonnes auto-numérotées, sinon l'`INSERT` échouera).
2. **À `lecteur_biblio`** :
   - `CONNECT`, `USAGE`, et **uniquement** `SELECT` sur les tables (existantes et futures).

### Étape 3 — Tester par la connexion (le test qui prouve tout)

Ouvre un **nouveau terminal** (garde la session admin ouverte) :

```bash
psql -h localhost -p 5432 -U app_biblio -d bibliotheque
# -h = l'hôte (l'adresse du serveur ; localhost = cette machine) ; -p = le port (5432 = guichet PostgreSQL, vu au Bloc 5)
# -U = l'utilisateur (le rôle) ; -d = la base ; le mot de passe est alors demandé (scram-sha-256)
```

Puis, **dans cette session `app_biblio`** :

1. `INSERT INTO livres (titre, auteur, annee_publication) VALUES ('1984', 'George Orwell', 1949);` → doit **réussir**.
2. `SELECT titre FROM livres;` → doit réussir.
3. `CREATE TABLE pirate (x INT);` → doit **échouer** (pas le droit de créer des tables).

Recommence avec `psql -h localhost -U lecteur_biblio -d bibliotheque` et **dans cette session** :

4. `SELECT titre FROM livres;` → doit réussir.
5. `DELETE FROM livres;` → doit **échouer** avec « permission denied » : **c'est la preuve que ta protection marche**.

### Étape 4 — Regarder et comprendre `pg_hba.conf`

1. Dans la session **admin**, exécute `SHOW hba_file;` → affiche le chemin exact du fichier (le numéro de version change selon ton installation — n'invente pas le chemin, **lis-le**).
2. Ouvre le fichier (`sudo nano <chemin affiché>`), repère :
   - la ligne `local   all   postgres   peer` (qui explique pourquoi `sudo -u postgres psql` marchait) ;
   - la ligne `host all all 127.0.0.1/32 scram-sha-256` (qui explique pourquoi ta connexion `-h localhost` a fonctionné).
3. Note dans tes notes : **quelles lignes** autorisent quoi, et pourquoi **l'ordre** des règles compte (la première qui correspond gagne).

### Étape 5 — Réfléchir : où stocker le mot de passe de l'application ?

Dans `notes-exercice-02.md`, compare ces trois emplacements et dis lesquels sont acceptables :

- a) écrit en dur dans `application.properties` versionné dans Git ;
- b) variable d'environnement `DB_PASSWORD` fournie au démarrage, valeur venant d'un coffre de secrets ;
- c) collé dans un message Teams au débutant.

Justifie en 2 phrases chacun (rappelle-toi le Bloc 6 : secrets et coffre).

---

## Livrable

Un fichier **`notes-exercice-02.md`** :
- les commandes de création des 2 rôles (avec `\du` comme preuve des attributs) ;
- les commandes `GRANT` ;
- le **résultat des 5 tests** (réussite/échec, avec le message d'erreur exact pour l'échec) ;
- tes observations `pg_hba.conf` ;
- le choix justifié de l'étape 5.

Correction détaillée dans **`03-correction.md`**. Aide-mémoire : **`04-commandes-references.md`**.
