# Leçon 3 — Écrire des scripts Bash fiables

> **Bloc 3 · Scripting et programmation** — Leçon 3 sur 7
> Un script qu'on écrit en une soirée, c'est facile. Un script **fiable** qu'on garde en production pendant des années, c'est autre chose. Cette leçon te donne la **discipline** (et les commandes) pour que tes scripts échouent vite, proprement et avec des messages utiles : `set -euo pipefail`, `trap`, gestion d'erreurs, validation d'arguments et journalisation.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Expliquer et utiliser** `set -euo pipefail` pour faire échouer un script tôt (« fail fast »).
2. **Nettoyer automatiquement** avec `trap ... EXIT` (fichiers temporaires, processus).
3. **Valider** les arguments et les prérequis d'environnement avant d'agir.
4. **Gérer les erreurs** en distinguant cas réellement fatals et cas récupérables.
5. **Journaliser** proprement (horodatage, stderr, fichiers de log) et **déboguer** avec `set -x`.

---

## 2. Explication simple

### 2.1 Le « pourquoi » : un échec silencieux est pire que rien

Le pire bug d'un script, ce n'est pas qu'il crashe : c'est qu'il **continue** alors qu'une commande a échoué, et te laisse croire que tout va bien. Exemple classique : le script de backup qui n'arrive plus à monter le disque, mais qui « réussit » quand même, parce que la dernière commande (le `echo` de fin) a retourné 0.

L'analogie : un pilote reçoit une **alarme « moteur en feu »**... le contraire serait de continuer à voler en souriant. `set -e` c'est l'alarme du pilote : dès qu'une étape échoue, on **s'arrête immédiatement** et on regarde l'erreur.

### 2.2 Le « comment » : `set -euo pipefail` en pratique

```bash
#!/usr/bin/env bash
set -euo pipefail
```

| Option | Effet |
|--------|-------|
| `set -e` | Arrête le script dès qu'une commande échoue (code non nul). |
| `set -u` | Erreur si on utilise une **variable non définie** (protège des typos). |
| `set -o pipefail` | Dans un pipe `a \| b`, échoue si **n'importe quelle** étape échoue (pas seulement la dernière). |

### 2.3 Le « quand » : tous mes scripts ?

Oui, **tous tes scripts**, dès la première version. Ces trois options sont le standard de l'industrie 2025-2026 pour des scripts Bash sérieux. Il faut juste connaître les **exceptions** (commandes dont on attend un échec) et les gérer explicitement avec `|| true` ou une condition `if`.

---

## 3. Exemples concrets

### 3.1 Jeu complet en tête de script

```bash
#!/usr/bin/env bash
set -euo pipefail

LOG_DIR="/var/log/mon-app"
mkdir -p "$LOG_DIR"
```

### 3.2 Gérer une commande « attendue à échouer »

```bash
# Si on tolère l'echec d'une commande :
grep -q "pattern" fichier.txt || echo "pattern absent (normal)"

# Ou dans une condition, set -e ne s'applique pas dans if :
if ssh -o BatchMode=yes server hostname > /dev/null 2>&1; then
    echo "ssh ok"
else
    echo "ssh indisponible"
fi
```

> 💡 **Astuce essentielle** : `set -e` ne s'applique **pas** à l'intérieur d'un `if`, d'un `while`, d'un `until`, ni aux commandes précédées de `||` ou `&&`. C'est là qu'on place les vérifications « tolérantes ».

### 3.3 `trap` pour le nettoyage

```bash
#!/usr/bin/env bash
set -euo pipefail

TMPFILE=$(mktemp)              # fichier temporaire sûr
cleanup() {
    rm -f "$TMPFILE"
    echo "Nettoyage effectue" >&2
}
trap cleanup EXIT              # execute cleanup a la sortie (meme en cas d'echec)

# ... corps du script
```

### 3.4 Valider arguments et prérequis

```bash
#!/usr/bin/env bash
set -euo pipefail

if [ "$#" -ne 1 ]; then
    echo "Usage : $0 environnement" >&2
    exit 1
fi
ENV="$1"

command -v curl >/dev/null 2>&1 || { echo "curl manquant" >&2; exit 1; }
[[ "$ENV" =~ ^(production|staging)$ ]] || { echo "env invalide : $ENV" >&2; exit 1; }
```

### 3.5 Fichiers de log avec horodatage

```bash
#!/usr/bin/env bash
set -euo pipefail

LOG="/tmp/mon-script.log"
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG"
}

log "Demarrage"
# ...
log "Fin"
```

### 3.6 Débogage `set -x`

```bash
bash -x mon-script.sh      # affiche chaque commande avant de l'executer
# ou, dans le script :
set -x   # active
# ...
set +x   # desactive
```

---

## 4. Bonnes pratiques modernes (2025-2026)

- **`set -euo pipefail` en tête de chaque script** : c'est la norme. Seul `set -u` peut être omis si un script doit rester compatible POSIX (`sh`), ce qui n'est pas notre cas.
- **`trap ... EXIT` systématique** dès qu'on crée des ressources temporaires (`mktemp`) : le nettoyage doit avoir lieu même sur crash.
- **`mktemp -d` plutôt que d'inventer un chemin** : `/tmp/mon-app-$$` est prévisible et devinable. `mktemp` crée un nom aléatoire et sûr.
- **Journaliser proprement et horodaté** : `tee -a` pour voir à l'écran **et** écrire dans le fichier.
- **Valider en amont** (arguments, commande présente, droits) plutôt que de planter au milieu.
- **Codes d'erreur distincts** : `exit 1` générique, `exit 2` mauvaise utilisation (arguments), `exit 3` prérequis manquant… documentés en tête de script.
- **Linter avec `shellcheck`** : `shellcheck script.sh` — il détecte 90% des bugs avant même le premier run.

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi c'est dangereux / inefficace | ✅ Version correcte |
|----------------|----------------------------------------|---------------------|
| Script sans `set -e` qui « réussit » malgré une erreur | La dernière commande retourne 0 → le script paraît OK alors qu'une étape a échoué. | `set -euo pipefail` en tête. |
| `cd /important` puis continuer sans vérifier | Si le `cd` échoue, le reste tourne dans le mauvais répertoire (drame). | `cd /important \|\| exit 1` (blindé). |
| Écrire dans `/tmp/fichier-fixe` | Chemin prévisible devinable par un autre processus → risque sécurité / collision. | `mktemp` ou `mktemp -d`. |
| Nettoyer « à la fin » sans `trap` | En cas de crash on ne fait pas le dernier bloc → fichier temp resté, processus orphelin. | `trap cleanup EXIT`. |
| Ignorer les erreurs avec `cmd 2>/dev/null` | On cache les vraies erreurs qu'on ne verra plus jamais. | Vérifier `$?` / utiliser `if` pour ce qu'on tolère ; `2>/dev/null` seulement pour le bruit délibéré. |
| Déboguer avec des `echo` partout puis commenter | Le code se pollue, et on oublie des lignes mortes. | `set -x` / `bash -x` pour tracer, retirer à la fin. |
| Sans gérer `&&` / `\|\|` quand une commande peut échouer | `set -e` stoppe brutalement tout le script sans message. | Employer `if` / `\|\| { ...; exit 1; }` avec message utile. |

---

## 8. Checklist de validation

- [ ] J'ai `set -euo pipefail` en tête de chaque script.
- [ ] J'utilise `trap ... EXIT` pour nettoyer mes fichiers temporaires (`mktemp`).
- [ ] Je valide **arguments et prérequis** avant le travail (usage, `command -v`, regex).
- [ ] Je distingue les erreurs fatales (exit) des cas tolérés (`if` / `\|\| true`).
- [ ] Je journalise avec horodatage vers **stderr** ou un fichier (`tee -a`).
- [ ] Je débogue avec `set -x` / `bash -x` plutôt que des `echo` jetables.
- [ ] J'ai passé mon script dans **`shellcheck`**.

---

> 📖 Prochaine étape : fais l'**exercice pratique** dans `02-exercice.md`, puis compare avec `03-correction.md`.
