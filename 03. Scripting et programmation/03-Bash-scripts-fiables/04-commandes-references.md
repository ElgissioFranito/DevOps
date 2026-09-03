# Aide-mémoire — Écrire des scripts Bash fiables

> **Bloc 3 · Leçon 3** — Fiche de référence pour rendre tes scripts sûrs en production.

## 📌 Le squelette fiable

```bash
#!/usr/bin/env bash
set -euo pipefail

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >&2
}

cleanup() {
    rm -f "$TMP"
}
trap cleanup EXIT

exit 0
```

## 📌 `set -euo pipefail`

| Option | Effet |
|--------|-------|
| `set -e` | Arrête à la première commande en échec (fail fast) |
| `set -u` | Erreur sur variable non définie |
| `set -o pipefail` | Échec d'un pipe si n'importe quelle étape échoue |

> 💡 Ne s'applique **pas** dans `if`, `while`, `until`, ni après `||` / `&&`. C'est là qu'on place les cas tolérés.

## 📌 Validation d'arguments

```bash
if [ "$#" -ne 1 ]; then
    echo "Usage : $0 argument_requis" >&2
    exit 1
fi
VAR="$1"
```

## 📌 Tests de prérequis

| Test | Vrai si |
|------|---------|
| `command -v stress` | la commande est dans le PATH |
| `[ -d "$x" ]` / `[ -f "$x" ]` | dossier / fichier existe |
| `[ -r "$f" ]` / `[ -w "$f" ]` | lisible / inscriptible |

## 📌 Fichier temporaire sûr

```bash
TMP=$(mktemp)          # fichier
TMPDIR2=$(mktemp -d)   # dossier
```

## 📌 Journalisation

```bash
log "msg" >&2                  # sur stderr
echo "..." | tee -a "$LOG"    # ecran + fichier
```

## 📌 Débogage

```bash
bash -x script.sh   # trace de chaque commande
set -x                    # activer la trace dans le script
set +x                    # desactiver
```

## 📌 Codes d'erreur recommandés

| Code | Signification |
|------|---------------|
| `0` | Succès |
| `1` | Erreur générique |
| `2` | Mauvaise utilisation (arguments) |
| `3` | Prérequis manquant |

> 📖 Après validation, lancer `shellcheck mon-script.sh`.
