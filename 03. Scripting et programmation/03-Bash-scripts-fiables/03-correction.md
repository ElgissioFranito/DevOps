# Correction détaillée — Écrire des scripts Bash fiables

> **Bloc 3 · Leçon 3** — Correction pas-à-pas de `02-exercice.md`. Suis chaque étape et compare à ton script.

---

## ✅ Étape 1 — Squelette et fonction log

```bash
mkdir -p ~/mon-backup
cd ~/mon-backup
nano backup.sh
chmod +x backup.sh
```

```bash
#!/usr/bin/env bash
set -euo pipefail

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >&2
}
```

**Explication** :
- `set -euo pipefail` : échoue vite (tout échec → arrêt), protège des variables non définies, et remonte les échecs dans un pipe.
- La sortie `>&2` (stderr) : si un autre programme capture stdout (ex. `./backup.sh | tee log`), les messages d'état ne polluent pas les données de sortie. En script d'admin, on logue sur stderr.

---

## ✅ Étape 2 — Validation des arguments et prérequis

```bash
if [ "$#" -ne 1 ]; then
    echo "Usage : $0 dossier_source" >&2
    exit 1
fi

SRC="$1"
if [ ! -d "$SRC" ]; then
    log "Erreur : $SRC n'est pas un dossier existant"
    exit 1
fi

mkdir -p "$HOME/backups"
```

**Explication** :
- Respecter le principe **fail-fast** : on vérifie tout avant de faire quoi que ce soit qui pourrait échouer au milieu.
- `[ ! -d "$SRC" ]` teste l'inexistence d'un dossier. Les guillemets protègent des espaces dans le chemin.
- On `mkdir -p` la cible même si elle n'existe pas encore, sans erreur.

---

## ✅ Étape 3 — Trap + fichier temporaire

```bash
TMP=$(mktemp)
cleanup() {
    rm -f "$TMP"
}
trap cleanup EXIT
```

**Explication** :
- `mktemp` crée un fichier vide avec un **nom aléatoire sûr** dans `/tmp`, beaucoup plus sûr qu'un `/tmp/mon-backup` fixe (collision et sécurité).
- `trap cleanup EXIT` s'exécute **dès la sortie du script**, quel qu'en soit le motif (fin normale, erreur de `set -e`, `exit 1`). C'est précisément cela qui évite les fichiers temporaires orphelins.
- On ne met **pas** `cleanup` « à la fin » du script à la main : en cas d'échec au milieu, la fin ne serait jamais atteinte. `trap` couvre tous les chemins.

---

## ✅ Étape 4 — L'action : tar

```bash
tar -czf "$TMP" -C "$(dirname "$SRC")" "$(basename "$SRC")"
mv "$TMP" "$HOME/backups/$(basename "$SRC")-$(date +%Y%m%d-%H%M).tar.gz"
```

**Explication** :
- `tar -czf` : crée une archive compressée gzip (`-z`), écrite dans `$TMP`.
- `-C "$(dirname)"` puis `$(basename)`: on archive **relative** au dossier parent. Résultat : l'archive contient `source/...` proprement, pas un chemin absolu `/home/.../source/...` impossible à extraire ailleurs.
- `mv` déplace l'archive temporaire vers le backup final avec un nom horodaté (`YYYYMMDD-HHMM`). C'est à peu près le seul endroit où `$TMP` change de place ; le `trap` reste utile si `tar` échoue avant le `mv`.

---

## ✅ Étape 5 — Résultats des tests

```
$ ./backup.sh ~/mon-backup/source
[2026-09-03 10:15:00] Sauvegarde terminee
$ echo $?
0

$ ./backup.sh
Usage : ./backup.sh dossier_source
$ echo $?
1

$ ./backup.sh /dossier-inexistant
[2026-09-03 10:15:20] Erreur : /dossier-inexistant n'est pas un dossier existant
$ echo $?
1
```

Vérifie qu'aucun fichier `tmp.*` résiduel ne reste dans `/tmp` après chaque exécution (`ls /tmp/tmp.*` ne doit rien montrer).

> 💡 Noter la **cohérence** : chaque cas d'erreur retourne `1` ; dans une chaîne `./backup.sh src && ./next.sh`, une erreur arrêterait la suite proprement.

---

## ✅ Étape 6 — Réponses de l'auto-vérification

1. **`set -e`** : stoppe le script à la première commande en échec. **`set -u`** : erreur si une variable non définie. **`set -o pipefail`** : dans un pipeline `a | b`, l'échec de `a` est remonté même si `b` réussit.
2. **`trap cleanup EXIT`** : se déclenche sur **toute** sortie du script (succès **ou** échec). un `cleanup` à la fin ne s'exécute pas si on sort en erreur au milieu. Le trap est la garantie que le nettoyage a toujours lieu.
3. **`dirname`/`basename`** : on travaille en **créant l'archive dans un dossier temporaire** avec des noms relatifs (`-C` + basename), sans changer le répertoire courant du script (`cd` global risquant de casser le reste du script). Cela rend aussi l'archive portable.
4. **Si `tar` échoue avec `set -e`** : le script **s'arrête immédiatement**, puis `trap cleanup EXIT` s'exécute et supprime le fichier temporaire partiel. Pas d'archive corrompue, pas de fichier orphelin.
5. **Logs sur stderr** : stderr reste séparé de stdout. un shell ou un pipe qui capture stdout pour traiter des *données* ne reçoit pas les messages de *statut*. C'est la convention qui rend le script utilisable en production.

---

## 📝 Checklist de validation (récapitulatif + conseils)

- [ ] Mon script commence par `set -euo pipefail`.
- [ ] J'ai validé le nombre d'arguments et l'existence du dossier avant d'agir.
- [ ] J'utilise `mktemp` + `trap cleanup EXIT` et je n'ai aucun fichier temporaire orphelin.
- [ ] Je journalise horodaté vers **stderr**.
- [ ] J'ai testé 1 succès + 2 échecs et vérifié les codes de sortie (`0` / `1`).
- [ ] J'ai passé mon script dans `shellcheck backup.sh` (aucune alerte bloquante).

### 💡 Conseils pour la suite

- **Re-teste-toi** dans 2-3 jours en ajoutant une étape de vérification de la taille de l'archive, ou un second argument (dossier de destination).
- **Réflexe pro** : commence **toujours** un script par `set -euo pipefail` + une fonction `log` sur stderr. C'est le squelette minimal de tout script fiable.
- La **Leçon 4** change de sujet : **YAML & JSON** — les formats de données que tu retrouveras dans docker-compose, Kubernetes et les pipelines CI/CD.

---

*Prochaine étape :* Leçon 4 — **Formats de données YAML & JSON** → dossier `04-YAML-et-JSON/`.
