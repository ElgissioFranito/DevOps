# Exercice pratique — Écrire des scripts Bash fiables

> **Bloc 3 · Leçon 3** — Exercice à faire en autonomie.
> Contexte : tu dois écrire **`backup.sh`**, un script de sauvegarde fiable d'un dossier d'application vers un dossier de backups, **en suivant toutes les bonnes pratiques de la leçon** (`set -euo pipefail`, `trap`, validation, journalisation). Objectif : pas de faux succès, pas de fichier temporaire orphelin, pas de message trompeur.

---

## 🎯 Objectif de l'exercice

Écrire un script de sauvegarde **réutilisable et sûr** qui :
- nécessite un argument (dossier source) sinon affiche un usage et échoue ;
- vérifie que la source existe ;
- cree un fichier temporaire via `mktemp` et le **nettoie en cas de sortie** avec `trap` ;
- archive et compresse la source dans le dossier backup (commande `tar`) ;
- horodate chaque étape vers **stderr** (pour une éventuelle redirection) ;
- retourne un code de sortie cohérent (0 si succès, 1 en erreur).

---

## 📋 Étape 1 — Squelette

Crée `~/mon-backup/backup.sh` avec, en introduction :

```bash
#!/usr/bin/env bash
set -euo pipefail
```

Une fonction `log()` qui affiche `[horodatage] message` sur **stderr**. (Rappel : `1>&2` ou `>&2`.)

---

## 📋 Étape 2 — Validation des arguments et prérequis

1. Vérifie qu'il y a **exactement 1 argument** (`$# -ne 1`) ; sinon usage sur stderr + `exit 1`.
2. Stocke la source dans une variable locale propre `SRC`.
3. Vérifie que `$SRC` est **un dossier** (option `-d`) ; sinon message + erreur.
4. Vérifie que le dossier cible `~/backups` existe, sinon le créer (`mkdir -p`).

---

## 📋 Étape 3 — Trap + fichier temporaire

1. Crée un fichier temporaire sûr : `TMP=$(mktemp)`.
2. Définis une fonction `cleanup()` qui supprime `$TMP`.
3. Associe `trap cleanup EXIT`.

---

## 📋 Étape 4 — L'action : tar

Crée une archive gzippée de la source dans `~/backups` :

```bash
tar -czf "$TMP" -C "$(dirname "$SRC")" "$(basename "$SRC")"
```

Puis déplace le fichier temporaire dans le dossier backup avec un nom horodaté :

```bash
mv "$TMP" "$HOME/backups/$(basename "$SRC")-$(date +%Y%m%d-%H%M).tar.gz"
```

---

## 📋 Étape 5 — Test manuel

```bash
mkdir -p ~/mon-backup/source && echo "contenu" > ~/mon-backup/source/app.txt
./backup.sh ~/mon-backup/source
./backup.sh            # doit afficher l'usage et echouer
./backup.sh /dosier-inexistant   # doit echouer proprement
```

Vérifie :
- le backup existe dans `~/backups/*.tar.gz` ;
- `echo $?` donne 0 pour le premier cas, 1 pour les cas d'erreur ;
- le fichier temporaire `mktemp` a été **nettoyé** (aucun fichier résiduel en `/tmp/tmp.*`).

---

## 📋 Étape 6 — Auto-vérification

1. Que fait exactement `set -euo pipefail` (chaque option) ?
2. Pourquoi `trap cleanup EXIT` plutôt que `cleanup` à la fin du script ?
3. Pourquoi `(dirname)/basename` plutot que `cd` vers la source ?
4. Que se passe-t-il si `tar` échoue alors que `set -e` est actif ?
5. Pourquoi écrire les logs sur **stderr** ?

---

## 🏁 Rendu attendu

Un script `~/mon-backup/backup.sh` exécutable, une exécution réussie + 2 échecs proprement gérés, et tes 5 réponses écrites.

> Compare ensuite avec `03-correction.md`.
