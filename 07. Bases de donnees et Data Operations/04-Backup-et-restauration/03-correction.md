# Correction — Leçon 4 : Backup et restauration

> **Bloc 7 · Leçon 4** — Correction pas à pas de `02-exercice.md`.
>
> 🔁 **Comment lire cette correction** : compare chaque étape ; chaque étape explique **pourquoi**. Le moment clé est l'étape 4 (la panne et ses suites) : c'est là que la théorie devient muscle. Checklist réécrite + conseils à la fin.

---

## Étape 1 — Préparer les données (attendu)

```sql
CREATE TABLE membres (
  id    GENERATED ALWAYS AS IDENTITY PRIMARY KEY,   -- auto-numéroté (Leçon 1)
  nom   VARCHAR(120) NOT NULL,
  email VARCHAR(200) NOT NULL UNIQUE
);
-- UNIQUE : une CONTRAINTE — le SGBD refuse deux lignes avec le même email.
-- C'est exactement le genre de règle que le SGBD applique MIEUX que le code applicatif.

INSERT INTO membres (nom, email) VALUES
  ('Amina Diallo',  'amina@example.com'),
  ('Jean Martin',   'jean@example.com'),
  ('Sofia Rossi',   'sofia@example.com'),
  ('Luc Bernard',   'luc@example.com'),
  ('Nour Haddad',   'nour@example.com');

SELECT count(*) FROM membres;      -- attendu : 5
```

## Étape 2 — Le backup complet (attendu)

```bash
mkdir -p backups        # dossier RELATIF au projet (on exécute depuis la racine du projet)

sudo -u postgres pg_dump -Fc -d bibliotheque -f "backups/bibliotheque-$(date +%F).dump"
sudo -u postgres pg_dumpall --globals-only -f "backups/globals-$(date +%F).sql"
ls -lh backups
```

Sortie attendue (les tailles dépendent de tes données) :

```
total 48K
-rw-r--r-- 1 postgres postgres 9,5K 11 sept. 10:15 bibliotheque-2026-09-11.dump
-rw-r--r-- 1 postgres postgres 3,2K 11 sept. 10:15 globals-2026-09-11.sql
```

```bash
pg_restore -l "backups/bibliotheque-$(date +%F).dump" | head -20
```

Ce qu'il faut **repérer** dans la liste : les lignes `TABLE membres`, `TABLE DATA membres` (la structure + les données) et les lignes **`ACL`** (les droits — le résultat de tes `GRANT` de la Leçon 2, donc **les droits de TABLES sont dans le dump**).

**Pourquoi la date dans le nom** : dans 3 mois, `bibliotheque-2026-09-11.dump` se parle tout seul — tu sauras ce qu'il contient sans l'ouvrir, et tu sauras **lequel supprimer** quand la rétention sera pleine (Leçon 3 : le disque ne se règle pas).

## Étape 3 — La panne (attendu)

```sql
-- Test SANS risque :
-- (1) Ouvre la transaction.
BEGIN;
-- (2) Lance la mauvaise commande (sans le WHERE).
DELETE FROM membres;
-- (3) Vérifie le désastre : 0 ligne restante.
SELECT count(*) FROM membres;
-- (4) ANNULE tout.
ROLLBACK;
-- (5) PREUVE : count(*) = 5, tes 5 lignes sont toujours là ✅
SELECT count(*) FROM membres;
-- ⚠️ COMMANDE DÉFINITIVE ci-dessous (hors transaction = autocommit).
-- D'abord, RELIS ton dump (Étape 2) : les lignes TABLE membres / TABLE DATA y sont BIEN ?
DROP TABLE membres;                     -- la table disparaît AVEC ses données (pour de vrai)
SELECT count(*) FROM membres;
-- ❌ ERROR:  relation "membres" does not exist
```

**Ce que tu devrais noter** : la transaction (`BEGIN`/`ROLLBACK`) a bien protégé le premier essai ; le `DROP TABLE` n'a **aucun** filet (il n'est pas annulable par `ROLLBACK` une fois confirmé, et surtout **pas de corbeille**). C'est la sensation physique qui ancre le réflexe : *avant une opération risquée → un backup*.

> 💡 **Pourquoi le `DROP TABLE` n'est pas couvert par une transaction ?** Techniquement il PEUT être dans une transaction (`BEGIN; DROP TABLE ...; ROLLBACK;` fonctionne). Mais la réalité des incidents : on ne le fait JAMAIS dans une transaction (les outils, les scripts, l'urgence). La protection fiable n'est pas le réflexe — c'est le **backup + le test**.

## Étape 4 — Restaurer (attendu)

**4.1 — Restaurer à côté** (le test, toujours d'abord) :

```bash
sudo -u postgres psql -c 'CREATE DATABASE bibliotheque_restaure;'
sudo -u postgres pg_restore -d bibliotheque_restaure "backups/bibliotheque-$(date +%F).dump"
```

**4.2 — Vérifier par les comptages** :

```sql
-- dans bibliotheque_restaure :  SELECT count(*) FROM membres;  → 5  ✅
-- dans bibliotheque_restaure :  SELECT count(*) FROM livres;   → le même qu'avant la panne ✅
```

> 💡 **Pourquoi « à côté » ?** La restauration est la **preuve** du backup. Si elle échoue, tu veux le savoir sur `bibliotheque_restaure` (sans conséquence), pas voir `pg_restore` s'arrêter **à mi-parcours** sur la vraie base (le piège 5 de la leçon).

**4.3 — Remettre en service** :

```bash
sudo -u postgres psql -c 'DROP DATABASE bibliotheque;'
sudo -u postgres psql -c 'CREATE DATABASE bibliotheque;'
sudo -u postgres pg_restore -d bibliotheque "backups/bibliotheque-$(date +%F).dump"
```

**4.4 — La vraie vérification (avec les rôles clients)** :

```bash
psql -h localhost -U app_biblio -d bibliotheque
# → SELECT count(*) FROM livres; ✅ les GRANT de TABLES reviennent AVEC le dump (les ACL sont dedans)
# → INSERT INTO livres (titre, auteur) VALUES ('Test', 'X'); ✅ fonctionne

psql -h localhost -U lecteur_biblio -d bibliotheque
# → ❌ probablement REFUSÉ (« password authentication failed » ou « permission denied » selon le point de blocage)
```

**Le piège que tu viens de vivre (l'essentiel de la leçon)** :

- `pg_dump` sauvegarde **la base** : tables, données, index, et les **ACL des objets** (les `GRANT` sur les tables).
- Il ne sauvegarde **PAS** : les **rôles** eux-mêmes (ils vivent dans le **cluster**, à côté des bases) ni les **droits sur les bases** (le `GRANT CONNECT` — attaché à la base, que tu viens de recréer **vide**).
- Conséquence : remise en service complète = **dump de base** + **dumpall des rôles** + **ton script versionné** des droits (Leçon 2 : *« tout ce qui est écrit à la main doit être versionné »* — voilà POURQUOI).

Si tu as rejoué `globals-...sql` (`sudo -u postgres psql -f "backups/globals-$(date +%F).sql"`) : les rôles sont recréés — mais les `GRANT CONNECT` **de cette base** ne reviennent pas (ils étaient dans l'ancienne base détruite). Le script versionné reste la réponse propre.

## Étape 5 — Les questions de réflexion (attendu)

**1. Pourquoi le dump de la base seul n'a pas suffi ?**
Parce que le **service** = base + **rôles** + **droits sur les bases**. Les rôles vivent dans le cluster (→ `pg_dumpall --globals-only`), les droits `CONNECT` sur la base sont détruits avec elle (→ script versionné). La leçon générale : **on ne restaure pas un fichier, on restaure un service** — et tout ce qui manque se liste **avant** la panne, en testant.

**2. Pourquoi restaurer à côté d'abord ?**
La restauration est la **preuve** du backup. Si elle échoue, tu veux le savoir sur `bibliotheque_restaure` (sans conséquence), pas en arrêtant l'application pour découvrir une archive corrompue. C'est le principe du **canari** (comme le test du VPN avant la bascule, Bloc 5) : on essaie l'issue de secours **avant** l'incendie.

**3. Fréquence et RPO pour ce projet ?**
Exemple cohérent : un apprenant sur un projet perso accepte de perdre **1 jour** de travail → backup **quotidien** (cron 02h30) → **RPO = 24 h**. Une vraie application avec des écritures continues : horaire + WAL → **RPO ≈ 1 h**. La règle : **la fréquence découle du RPO**, jamais l'inverse.

**4. Où ranger les copies (3-2-1) ?**
Copie 1 : le dossier `backups/` de la machine (le dump brut). Copie 2 : un **support différent** (disque externe, autre machine). Copie 3, **hors site** : un **bucket S3 chiffré** (Bloc 6, Leçon 4 — versioning + lifecycle pour la purge). Concrètement : `aws s3 cp backups/bibliotheque-... s3://mon-bucket-backups/` — la commande exacte de ton Bloc 6.

## ✅ Checklist de validation (réécrite)

- [ ] Je nomme les 3 causes de perte (humaine, matérielle, attaque) et j'explique *« un backup non testé n'est pas un backup »*.
- [ ] Je sauvegarde une base (`pg_dump -Fc`, date dans le nom) **et** les rôles (`pg_dumpall --globals-only`) — et je sais dire POURQUOI les deux.
- [ ] Je vérifie un backup sans restaurer (`pg_restore -l` → TABLE, TABLE DATA, **ACL** ; `ls -lh` → tailles).
- [ ] Je restaure **à côté** d'abord, **je compte** pour vérifier, puis je remets en service (`DROP`/`CREATE`/`pg_restore`).
- [ ] J'ai vécu le piège des **`GRANT CONNECT` perdus** et je sais que la réponse est le **script versionné** (Leçon 2) + `pg_dumpall`.
- [ ] Je sais expliquer le **WAL** (carnet du caissier), l'**incrémental** et le **PITR** (photo + carnet jusqu'à une heure cible).
- [ ] Je connais le **3-2-1** et je sais choisir une fréquence à partir d'un **RPO**.
- [ ] Je fais un backup **avant** toute opération risquée (le réflexe appris en cassant `membres`).

## 💡 Conseils

- **Rejoue cet exercice une fois par mois** : le réflexe « sauvegarder → casser → restaurer → vérifier » en 15 minutes vaut tous les plannings de sécurité. (Un vrai pro garde ce test **automatisé** — c'est un script Bash parfait pour le Bloc 3.)
- **Les comptages sont ta preuve** : après toute restauration, `SELECT count(*)` sur chaque table importante — comparé aux chiffres **d'avant** la panne, notés dans tes notes.
- **La date dans les noms** (`$(date +%F)`) est ta mémoire : sans elle, dans 3 mois, tu ne sauras plus quel dump est frais.
- **Le lien avec la suite** : la Leçon 5 automatisera les **changements de structure** (migrations versionnées). Chaque migration sérieuse **commence par un backup** — tu sais maintenant pourquoi, et **comment**.

---

> 🎉 **Fin de la correction de la Leçon 4.** Si tout est coché, tu réponds au critère de la roadmap : *« expliquer comment éviter une perte de données »* — parce que tu l'as **vécu** : sauvegardé, cassé, restauré, vérifié.
