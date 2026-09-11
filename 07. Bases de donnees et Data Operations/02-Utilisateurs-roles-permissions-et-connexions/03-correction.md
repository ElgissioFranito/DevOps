# Correction — Leçon 2 : Utilisateurs, rôles, permissions et connexions

> **Bloc 7 · Leçon 2** — Correction pas à pas de `02-exercice.md`.
>
> 🔁 **Comment lire cette correction** : compare chaque étape avec ce que tu as fait. Chaque étape explique **pourquoi** le choix technique (pas seulement « comment »). La checklist de validation est réécrite à la fin, avec des conseils.

---

## Étape 1 — Créer les rôles (attendu)

```bash
sudo -u postgres psql       # se connecte en tant qu'administrateur local (méthode « peer »)
```

```sql
CREATE ROLE app_biblio LOGIN PASSWORD 'Biblio-App-2026-moulin!coffre'
  NOSUPERUSER NOCREATEDB NOCREATEROLE;

CREATE ROLE lecteur_biblio LOGIN PASSWORD 'Biblio-Lecteur-2026-papier!plume';

\du        -- liste les rôles
```

**Sortie attendue** (simplifiée) :

```
     Nom du rôle   |            Attributs
-------------------+-----------------------------------
 app_biblio        |
 lecteur_biblio    |
 postgres          | Superuser, Créer des bases, ...
```

**Pourquoi ces choix techniques** :

- **`LOGIN`** : sans lui, le rôle est un simple « groupe » qui ne peut pas se connecter. Avec lui, c'est un utilisateur.
- **La phrase de passe** : longue et composée de mots (plus facile à retenir qu'un charabia court, plus difficile à deviner qu'un mot du dictionnaire). Elle vit dans ton **gestionnaire de mots de passe** (Bitwarden, KeePassXC) — pas dans tes notes, pas dans Git.
- **`NOSUPERUSER NOCREATEDB NOCREATEROLE`** : ces trois refus sont la **traduction technique** du moindre privilège. `\du` doit confirmer des **attributs vides** pour tes deux rôles (comparé à `postgres` : Superuser).

> 💡 **Bonne habitude** : pour changer un mot de passe plus tard, utilise `\password app_biblio` (dans psql) — il te le demande **en caché** et il ne finit ni dans l'historique du shell, ni dans les logs.

## Étape 2 — Accorder les permissions (attendu)

```sql
-- ENTRER DANS LA BASE (depuis la session admin)
GRANT CONNECT ON DATABASE bibliotheque TO app_biblio;
GRANT CONNECT ON DATABASE bibliotheque TO lecteur_biblio;

\c bibliotheque        -- l'admin entre dans la base pour les droits internes

-- UTILISER LE SCHÉMA public (sans USAGE, rien ne se voit dans le dossier)
GRANT USAGE ON SCHEMA public TO app_biblio;
GRANT USAGE ON SCHEMA public TO lecteur_biblio;

-- APP : lire et écrire, tables existantes...
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_biblio;
-- ...et futures :
ALTER DEFAULT PRIVILEGES IN SCHEMA public
  GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO app_biblio;

-- APP : les séquences (sinon le premier INSERT échoue : il demande à la séquence un numéro)
GRANT USAGE ON ALL SEQUENCES IN SCHEMA public TO app_biblio;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT USAGE ON SEQUENCES TO app_biblio;

-- ANALYSTE : lecture seule, existantes et futures
GRANT SELECT ON ALL TABLES IN SCHEMA public TO lecteur_biblio;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO lecteur_biblio;

-- FERMER LA PORTE LARGE (« tout le monde » n'a pas à entrer ici)
REVOKE CONNECT ON DATABASE bibliotheque FROM PUBLIC;
REVOKE CREATE ON SCHEMA public FROM PUBLIC;
```

**Explications des choix (pourquoi pas plus simple ?)** :

- **`ALTER DEFAULT PRIVILEGES`** : un `GRANT ... ON ALL TABLES` ne touche que les tables **existantes au moment du GRANT**. Sans cette commande, la prochaine table créée (par exemple par une migration de la Leçon 5) ne serait pas accessible à `app_biblio` — et tu chercherais longtemps pourquoi « ça marchait hier ».
- **`USAGE ON SEQUENCES`** : la colonne `id` (Leçon 1) est remplie par une **séquence** (le compteur). Écrire une ligne = demander un numéro à ce compteur. Sans `USAGE`, l'`INSERT` échoue avec « permission denied for sequence livres_id_seq » — si tu as croisé cette erreur pendant l'exercice, voilà pourquoi, et c'est une belle preuve que tu as compris le mécanisme.
- **`REVOKE ... FROM PUBLIC`** : `PUBLIC` est le pseudo-rôle « **tout le monde** ». Par défaut, PostgreSQL laisse « tout le monde » se connecter aux bases et créer des objets dans le schéma `public` (le comportement a été durci à partir de PostgreSQL 15, mais les installations plus anciennes restent ouvertes). La logique du **refus par défaut** : on ferme la porte large **avant** de distribuer les clés précises.

## Étape 3 — Tester par la connexion (attendu)

**Preuve n° 1 — le test de connexion lui-même** :

```bash
psql -h localhost -p 5432 -U app_biblio -d bibliotheque
# -h : l'adresse du serveur (localhost = cette machine) ; -p : le port (5432 = guichet PostgreSQL, Bloc 5)
# -U : le rôle ; -d : la base
# → « Mot de passe pour l'utilisateur app_biblio : » : tu SAISIS la phrase de passe (scram-sha-256 à l'œuvre)
```

**Preuve n° 2 — les droits d'écriture de l'application** :

```sql
INSERT INTO livres (titre, auteur, annee_publication) VALUES ('1984', 'George Orwell', 1949);
-- ✅ RÉUSSIT (INSERT + USAGE sur la séquence accordés)
SELECT titre FROM livres;
-- ✅ RÉUSSIT (SELECT accordé)
CREATE TABLE pirate (x INT);
-- ❌ ÉCHOUE : « ERROR: permission denied for schema public »
-- Preuve : aucun GRANT CREATE n'a été donné — le refus par défaut fait son travail
```

**Preuve n° 3 — la lecture seule de l'analyste** :

```sql
-- psql -h localhost -U lecteur_biblio -d bibliotheque
SELECT titre FROM livres;    -- ✅ RÉUSSIT : lecture accordée
DELETE FROM livres;          -- ❌ ÉCHOUE : « ERROR: permission denied for table livres »
```

**Pourquoi ce test est le plus important de l'exercice** : c'est le moment où le moindre privilège passe de la **théorie** à la **preuve**. Le message d'erreur que tu as vu n'est pas un problème — c'est **exactement** ce que tu as construit. En production, ce type de test se rejoue à chaque changement (et se **versionne** comme un test automatisé).

> 💡 **Si ta connexion `-h localhost` a échoué** (« password authentication failed ») : soit la phrase de passe était mal tapée (copie-la depuis ton gestionnaire), soit `pg_hba.conf` n'applique pas `scram-sha-256` sur ta version (regarde la ligne `host all all 127.0.0.1/32` — l'étape 4 t'a montré où elle vit).

## Étape 4 — `pg_hba.conf` (attendu)

```bash
sudo -u postgres psql
```

```sql
SHOW hba_file;
-- Sortie attendue (selon ta version) : un chemin qui ressemble à /etc/postgresql/16/main/pg_hba.conf
-- Remarque : le « 16 » est le numéro de VERSION de PostgreSQL — ne le devine pas, lis-le.
\q
HBA="$(sudo -u postgres psql -tAc 'SHOW hba_file;')"   # relit le chemin affiché par le SGBD (options -tAc = sortie brute, commande SQL)
sudo nano "$HBA"    # ouvre le fichier ; Ctrl+O sauve, Ctrl+X quitte
```

**Ce que tu devrais avoir noté** :

- `local   all   postgres   peer` : la ligne qui rendait possible `sudo -u postgres psql` depuis le début (Bloc 2). **Sans réseau** (socket locale), ton compte Linux doit correspondre au rôle demandé.
- `host   all   all   127.0.0.1/32   scram-sha-256` : la ligne qui explique ta connexion `-h localhost` — via le **réseau** (TCP), un mot de passe est exigé, et ce n'est possible que depuis cette machine (`127.0.0.1/32` : le CIDR du Bloc 5, « exactement cette adresse »).
- **L'ordre compte** : si tu ajoutes une règle précise (ex. `host bibliotheque app_biblio 192.168.1.50/32 scram-sha-256`), elle doit être **au-dessus** des règles génériques `host all all`, car **la première qui correspond gagne**.
- **Application** : `sudo systemctl reload postgresql` — `reload` relit la configuration **sans couper** les connexions actives (alors que `restart` coupe tout : panne évitable).

> 🔁 **Liens croisés à retenir** : le **pare-feu système** (Bloc 5, Leçon 3) décide « le port 5432 est-il joignable depuis cette machine ? » ; `pg_hba.conf` décide « ce rôle peut-il entrer, et comment ? » ; les **security groups** (Bloc 6) sont l'équivalent cloud des deux. Trois couches, une même logique : **la porte se ferme en premier, les droits se distribuent ensuite**.

## Étape 5 — Où stocker le mot de passe (attendu)

| Emplacement | Verdict | Justification |
|---|---|---|
| a) en dur dans `application.properties`, versionné dans Git | ❌ **Jamais** | Git **conserve l'historique** : même retiré, le mot de passe reste dans les commits passés et se propage à chaque `push` |
| b) variable d'environnement `DB_PASSWORD` fournie au démarrage, valeur venant d'un coffre de secrets | ✅ **C'est LA réponse** | Le code ne contient que le **nom** de la variable ; la valeur vit dans un endroit chiffré, centralisé et contrôlé (Bloc 6, IAM), et sa rotation est simple |
| c) collé dans un message Teams | ❌ **Jamais** | Un canal de discussion n'est ni chiffré ni contrôlé : le secret survit à toutes les purges, accessible à toute la conversation |

> 🔁 **Rappel du pont** : ce rappel du Bloc 6 n'était pas décoratif — la Leçon 6 (IAM) te faisait créer des clés avec `aws configure` et éviter les secrets en dur ; ici, la base suit exactement la même loi.

## ✅ Checklist de validation (réécrite)

- [ ] Je peux expliquer le moindre privilège (l'analogie des clés de l'immeuble) et pourquoi l'application ne se connecte **jamais** avec `postgres`.
- [ ] Je sais créer un rôle avec `LOGIN`, une phrase de passe, et **sans pouvoirs d'administration** (`NOSUPERUSER NOCREATEDB NOCREATEROLE`) — et je le vérifie avec `\du`.
- [ ] Je sais accorder (`GRANT`) et retirer (`REVOKE`) des permissions précises, y compris pour les tables **futures** (`ALTER DEFAULT PRIVILEGES`) et les **séquences** (`USAGE ON SEQUENCES`).
- [ ] Je sais **prouver par le test** qu'un rôle peut faire ce qu'il doit et rien de plus (j'ai vu les messages « permission denied » attendus).
- [ ] Je comprends `pg_hba.conf` : ses 5 colonnes, l'**ordre des règles** (la première qui correspond gagne), et la différence **peer** / **scram-sha-256**.
- [ ] Je connais les couches empilées : pare-feu (Bloc 5) → `pg_hba.conf` (ici) → permissions SQL (ici) → security groups (Bloc 6).
- [ ] Je connais `systemctl reload` (relire sans couper) vs `restart` (couper tout) et quand utiliser chacun.
- [ ] Je sais où vit le mot de passe de l'application : **variable d'environnement + coffre-fort** (jamais le code, jamais Git).

## 💡 Conseils

- **Prouve toujours par le test** : une permission se démontre par « j'ai vu l'erreur permission denied », jamais par « ça devrait marcher ».
- **Versionne ce que tu écris à la main** : tes `CREATE ROLE` et `GRANT` du jour méritent d'être dans Git (fichier de notes ou script) — en Leçon 5, les outils de migration automatiseront ce réflexe.
- **Phrase de passe > mot de passe** : `Biblio-App-2026-moulin!coffre` est plus solide et plus facile à retenir que `X3$9kL!`.
- **Trois couches, une logique** : pare-feu (peut-on joindre le port ?) → `pg_hba.conf` (ce rôle peut-il entrer ?) → `GRANT` (que peut-il faire une fois entré ?). Si un jour « quelqu'un accède à quelque chose qu'il ne devrait pas », descends ces couches **une par une** — c'est une démarche de diagnostic, pas de devinette.
- **Prochaine étape logique** : ta base est **bien gardée**. La Leçon 3 la rend **bien réglée** (configuration, mémoire, connexions, journaux) — et s'appuie sur l'observation des ressources que tu as commencée en Leçon 1.

---

> 🎉 **Fin de la correction de la Leçon 2.** Si tout est coché, tu maîtrises le « qui peut toucher à quoi » — et tu es prêt(e) pour la Leçon 3 : le « comment la base tourne et ce qu'elle consomme ».
