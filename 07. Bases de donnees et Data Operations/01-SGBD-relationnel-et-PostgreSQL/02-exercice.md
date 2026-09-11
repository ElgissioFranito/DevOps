# Exercice — Leçon 1 : Le SGBD relationnel et PostgreSQL

> **Bloc 7 · Leçon 1** — Exercice en autonomie, **100 % en local** (PostgreSQL installé au Bloc 2, Leçon 6). Aucun compte cloud, aucun coût.
>
> 🔁 **Comment s'articulent les fichiers de cette leçon** : lis d'abord `01-lecon.md` (la théorie + le vocabulaire + la checklist), puis fais **cet exercice** sans regarder la solution, et compare ensuite avec `03-correction.md` (correction pas à pas + checklist réécrite). L'aide-mémoire `04-commandes-references.md` reste à côté de toi.

---

## Contexte

Tu viens d'être embauché(e) sur le fil rouge (application Spring Boot + Angular). Le développeur frontend a besoin que la **base `bibliotheque`** existe, avec une **table `livres`** prête à recevoir des données, et il veut des **requêtes SQL** fiables. Ton travail : créer tout cela avec `psql` et **observer** ce que la base consomme.

> ⚠️ **Prérequis** : PostgreSQL installé et démarré (Bloc 2, Leçon 6). Si ce n'est pas fait :
> ```bash
> sudo apt update && sudo apt install -y postgresql   # installe PostgreSQL (apt = le gestionnaire de paquets d'Ubuntu)
> sudo systemctl start postgresql                     # démarre le service (systemctl = pilote les services, vu au Bloc 2)
> ```

---

## Énoncé

### Étape 1 — Créer la base et la table

1. Connecte-toi au SGBD en tant qu'**administrateur local** (`sudo -u postgres psql`).
2. Liste les bases existantes (`\l`), puis **crée** la base `bibliotheque` et **connecte-toi** dessus.
3. Crée la table `livres` avec ces colonnes :
   - `id` : numéro automatique, clé primaire (utilise `GENERATED ALWAYS AS IDENTITY` — la bonne pratique 2025, pas l'ancienne `SERIAL`) ;
   - `titre` : texte court (max 200), **obligatoire** ;
   - `auteur` : texte court (max 120), **obligatoire** ;
   - `annee_publication` : nombre entier ;
   - `prix` : décimal **exact** avec 8 chiffres dont 2 après la virgule, valeur par défaut `0.00` ;
   - `cree_le` : date + heure avec fuseau, remplie automatiquement à l'heure actuelle.
4. Vérifie avec `\dt` puis `\d livres` (et note à quoi sert la colonne « Séquence » dans la sortie).

### Étape 2 — Le CRUD (créer, lire, modifier, supprimer)

Dans `bibliotheque` :

1. **C**réer : insère **3 livres** (dont *Le Petit Prince*, 1943, 7.99) — utilise `RETURNING *` pour voir ce que PostgreSQL a réellement rangé.
2. **L**ire : affiche
   - les livres de plus de 5 € ;
   - les livres triés par année de publication (du plus ancien au plus récent) ;
   - **seulement 2 résultats** (pense `LIMIT`).
3. **M**odifier : passe le prix du *Petit Prince* à 8.99.
4. **S**upprimer : efface **un seul** livre, **par son `id`** (jamais sans condition !).

### Étape 3 — Observer les ressources

1. Affiche la **taille** de la base `bibliotheque` (`pg_database_size` + `pg_size_pretty`).
2. Affiche la valeur de `max_connections` (le nombre maximal de « guichets » de connexion).
3. Affiche le **nombre de connexions actives** (`pg_stat_activity`).

### Étape 4 — Traquer le piège (test de sécurité)

Exécute **dans cet ordre** (et note ce que tu observes) :

```sql
BEGIN;               -- ouvre une « transaction » : tout ce qui suit peut être annulé
DELETE FROM livres;  -- tu as OUBLIÉ le WHERE...
ROLLBACK;            -- annule tout : tes 2 lignes restantes sont sauvées
SELECT * FROM livres;  -- preuve : rien n'a été supprimé
```

Note dans tes notes ce qui se serait passé **sans** le `BEGIN`/`ROLLBACK`.

---

## Livrable

Un fichier **`notes-exercice-01.md`** à la racine du projet, contenant :
- la définition de ta table `livres` (copie du `CREATE TABLE`) ;
- tes requêtes CRUD commentées + 1 ligne d'observation chacune ;
- la taille de la base, `max_connections`, le nombre de connexions actives ;
- ce que tu as appris avec le test `BEGIN`/`ROLLBACK`.

Correction détaillée dans **`03-correction.md`**. Aide-mémoire : **`04-commandes-references.md`**.
