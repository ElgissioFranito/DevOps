# Correction détaillée — Structures de contrôle et fonctions Bash

> **Bloc 3 · Leçon 2** — Correction pas-à-pas de `02-exercice.md`. Suis chaque étape et compare à ton script.

---

## ✅ Étape 1 — Le squelette et la fonction `log`

```bash
mkdir -p ~/mon-check
cd ~/mon-check
nano check-service.sh
chmod +x check-service.sh
```

Contenu initial :

```bash
#!/usr/bin/env bash

log() {
    local msg="$1"
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $msg"
}
```

**Explication** : `$(date ...)` est une **substitution de commande** : on exécute `date` et on insère sa sortie dans la chaîne. On date chaque ligne de log, indispensable pour comparer des états dans le temps.

---

## ✅ Étape 2 — La fonction `check_service`

```bash
check_service() {
    local service="$1"
    if systemctl is-active --quiet "$service"; then
        log "OK : $service"
        return 0
    else
        log "ECHEC : $service"
        return 1
    fi
}
```

**Pourquoi `systemctl is-active --quiet`** :
- `is-active` n'affiche **que** `active` / `inactive` / `failed` et retourne un code de sortie (0 si actif). C'est idéal pour une condition.
- `--quiet` supprime la sortie, on n'a donc **que le code de retour** à tester.
- `systemctl status` serait beaucoup trop verbeux pour un script (sortie paginée, couleur…).
- On protège `"$service"` par des guillemets — jamais une valeur utilisateur sans citation.

---

## ✅ Étape 3 — La boucle et le `case`

```bash
SERVICES="nginx mysql ssh sshd"

check_all() {
    for service in $SERVICES; do
        check_service "$service"
    done
}

case "$1" in
    all)  check_all ;;
    *)  echo "Usage : $0 all" >&2; exit 1;;
esac
```

**Explication** :
- La boucle `for service in $SERVICES` itère sur **chaque mot** de la variable (séparé par des espaces). Chaque itération appelle `check_service`.
- Le `case` gère le **routage** : un seul argument accepté, sinon usage sur stderr + `exit 1`
- On a bien découpé en **trois fonctions** propres (`log`, `check_service`, `check_all`) : chaque bloc a une responsabilité unique — le réflexe DRY venant de tes acquis Java/JS.

`case` vs `if` ici : le `case` est plus lisible pour une **liste courte de valeurs exclusives**. Pour un choix binaire simple, `if` suffirait.

---

## ✅ Étape 4 — Exemples de sortie (sur un vrai serveur)

```
[2026-09-03 15:45:10] OK : nginx
[2026-09-03 15:45:11] OK : mysql
[2026-09-03 15:45:12] ECHEC : ssh
[2026-09-03 15:45:13] OK : sshd
```

Sans argument :

```
Usage : ./check-service.sh all
```

(affiché sur stderr, et `echo $?` → `1`.)

> 💡 Les services `ssh` et `sshd` sont volontairement les deux testés (parfois seul l'un est actif selon la config. Tu apprends ainsi à ne pas présumer que tous tes services seront actifs.

---

## ✅ Étape 5 — Réponses de l'auto-vérification

1. **`[[ ]]` vs `[ ]`** : `[[ ]]` est une construction **interne** de Bash (pas une commande externe), supporte `&&`, `||`, `=~`, et ne casse pas avec les valeurs vides. `[ ]` est plus ancien et moins tolérant. On préfère `[[ ]]` en Bash pur.
2. **Chaînes vs entiers** : `=` compare deux **chaînes** (la chose elle-même), `-eq` compare deux **nombres** (valeur numérique). `if [ "3" -eq 3 ]` est vrai, `if [ "3" = 3 ]` est **faux** (chaînes différentes car les caractères diffèrent. En `[ ]`, les opérateurs `<` et `>` sont des redirections, pas des comparaisons.
3. **`local service="$1"`** : déclare la variable **locale à la fonction** (pas persistante globalement) et la **nomme** (on sait ce que représente `$1`). Pour `log`, `check_service` et `check_all` qui sont appelées plusieurs fois, `local` évite que `service` fuit et écrase une variable du script.
4. **`systemctl is-active --quiet` vs `status`** : `is-active` → un mot (`active`) + un code de retour adapté à une condition ; `status` → un texte long et coloré, pour humains. En script on veut la **forme programmatique**.
5. **Sans `return`** : la fonction renvoie le code de sortie de sa **dernière commande exécutée** — pas nécessairement le résultat logique qu'on voulait. Toujours retourner `0`/`1` explicitement pour une fonction booléenne.

---

## 📝 Checklist de validation (récapitulatif + conseils)

- [ ] J'ai une fonction `log` qui horodate ses sorties.
- [ ] Ma fonction `check_service` utilise `systemctl is-active --quiet` et retourne `0`/`1`
- [ ] Je boucle avec `for` sur une liste de services (`nginx mysql ssh sshd`).
- [ ] Je route avec `case` et gère l'usage sur stderr + `exit 1`
- [ ] J'utilise `local` et je nomme les paramètres de chaque fonction.
- [ ] J'ai testé `all` et sans argument et j'ai vérifié les codes de sortie.

### 💡 Conseils pour la suite

- **Re-teste-toi** dans 2-3 jours avec une liste de services différente (ex. `nginx redis postgresql`).
- **Réflexe pro** : faces à un diagnostic, écris une petite fonction dédiée par vérification (`check_disk`, `check_ram`, `check_service`) — c'est le prélude au script « tout-en-un » de la Leçon 3.
- La **Leçon 3** te montre comment rendre ces scripts **fiables** avec `set -euo pipefail`, `trap` et la gestion d'erreurs.

---

*Prochaine étape :* Leçon 3 — **Écrire des scripts Bash fiables** → dossier `03-Bash-scripts-fiables/`.
