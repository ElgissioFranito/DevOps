# Leçon 2 — Structures de contrôle et fonctions Bash

> **Bloc 3 · Scripting et programmation** — Leçon 2 sur 7
> Tu sais déjà `if`, `for`, `while` et les fonctions en Java/JS. Cette leçon te montre **leur équivalent Bash** — avec les subtilités spécifiques au shell : les conditions `[ ]` et `[[ ]]`, les boucles sur des fichiers, et comment découper un script en **fonctions** propres.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Écrire** des conditions `if` / `elif` / `else` avec les tests `test`, `[ ]` et modernes `[[ ]]`.
2. **Utiliser** `for`, `while` et `case` pour traiter des listes de valeurs et de fichiers.
3. **Comparer** nombres (entiers) et **tester** fichier/existence avec les opérateurs bash.
4. **Définir et appeler** une **fonction** en Bash, avec arguments et valeur de retour (`return`).
5. **Choisir** la bonne structure selon le cas (la même transposition logique que tu connais déjà).

---

## 2. Explication simple

### 2.1 Le « pourquoi » : rendre les scripts intelligents

Au bloc précédent tu as vu l'anatomie d'un script et les variables. Mais un script utile **prend des décisions** : « si le service tourne, ne le relance pas ; sinon, relance-le. » C'est exactement ce que tu manipules déjà en Java/JS (`if`, `for`, `while`), mais le **shell Bash a une syntaxe déroutante** : beaucoup d'espaces, deux sortes de comparaison (valeurs vs chaînes), un cas particulier `[[ ]]`.

L'analogie : en cuisine, tu as la **recette** (les commandes, Leçon 1) et maintenant tu ajoutes les **branches logiques** (« si les oignons sont restés, faites 5 minutes de plus ») et les **boucles** (« pour chaque garniture, ajoute-en une »). C'est ce qui transforme une suite de commandes en **programme**.

### 2.2 Le « comment » : les fondations de la syntaxe

**Condition `if` :** la syntaxe exige des **espaces** et un `;` avant `then` :

```bash
if [ "$PORT" -eq 8080 ]; then
    echo "port ok"
fi
```

La commande test est `[ ... ]` (équivalent ancien de la commande `test`) ou [[ ... ]] (moderne, plus sûr, recommandé 2025-2026). C'est en fait une **commande** qui retourne 0 (vrai) ou non-nul (faux) — c'est pour ça qu'il y a des espaces de chaque côté des parenthèses.

### 2.3 Le « quand » : quelle structure ?

| Besoin | Structure |
|--------|-----------|
| Test une condition / fichier | `if [ ]` ou `[[ ]]` |
| Plusieurs choix exclusifs | `case` |
| Itérer sur une liste | `for` |
| Répéter tant que … | `while` |
| Réutiliser un bloc | **fonction** |

> 💡 **Lien avec tes acquis** : une `for` Bash équivaut à un `for (String x : liste)` Java (enhanced for) : tu itères sur une liste, pas sur un index. Réflexe : transposer la **logique**, pas la **syntaxe**.

---

## 3. Exemples concrets

### 3.1 Conditions avec `[ ]` et `[[ ]]`

```bash
#!/usr/bin/env bash
PORT=8080

# comparaison d'entiers : -eq -ne -lt -gt (pas de <, >)
if [ "$PORT" -eq 8080 ]; then
    echo "port standard"
fi

# comparaison de chaînes
if [ "$ENV" = "production" ]; then
    echo "environnement prod"
fi

# moderne : [[ ]], supporte =~ (regex), && , ||
if [[ "$ENV" =~ ^prod ]]; then
    echo "environnement qui commence par prod"
fi

# combiner
if [[ "$PORT" -gt 1024 && "$PORT" -lt 65536 ]]; then
    echo "port valide"
fi
```

### 3.2 Tester un fichier

```bash
if [ -f "/etc/nginx/nginx.conf" ]; then
    echo "fichier existe"
fi
if [ -d "$HOME/apps" ]; then
    echo "dossier existe"
fi
if [ -z "$VAR" ]; then        # -z : chaîne vide
    echo "VAR est vide"
fi
```

### 3.3 Boucle `for` sur une liste / *

```bash
# sur une liste de valeurs
for env in production staging dev; do
    echo "Prepare environnement : $env"
done

# sur des fichiers (glob)
for conf in /etc/app/*.conf; do
    echo "Chargement : $conf"
done

# plage d'entiers (style ancien, à éviter)
# for i in $(seq 1 5); do ... done
```

### 3.4 Boucle `while`

```bash
# tant que le fichier a des lignes
while IFS= read -r ligne; do
    echo "Ligne : $ligne"
done < /etc/app/hosts.txt

# attente de disponibilité (pattern de polling DevOps)
while ! curl -sf "http://localhost:8080/health"; do
    echo "L'API n'est pas prête, nouvel essai dans 2s..."
    sleep 2
done
echo "API prête !"
```

### 3.5 `case` (choix multiples)

```bash
case "$ENV" in
    production)  echo "Mode prod";;
    staging)     echo "Mode staging";;
    dev)         echo "Mode dev";;
    *)           echo "inconnu"; exit 1;;
esac
```

### 3.6 Fonctions

```bash
#!/usr/bin/env bash

# Définition : log() { ... ; }
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"
}

# Appel
log "Demarrage du deploiement"

# Retour et capture
check_env() {
    [[ "$1" = "production" ]] && return 0 || return 1
}
if check_env "production"; then
    log "Ok production"
fi
```

---

## 4. Bonnes pratiques modernes (2025-2026)

- **Privilégier `[[ ]]` à `[ ]`** : interprété par Bash (pas une commande externe), plus sûr avec les valeurs vides, et supporte `&&`, `||`, `=~`, les parenthèses. C'est le standard actuel pour des scripts Bash purs.
- **Citer toujours les variables** : `[ -z "$VAR" ]`, `[[ "$PORT" ... ]]`. Une citation manquante sur une valeur vide fait planter le test.
- **Comparer les entiers avec `-eq`, `-ne`, `-gt`, `-lt`**, jamais avec `<`/`>` (qui sont des redirections en shell).
- **Itérer sur la liste réelle plutôt que `$(seq n)`** : `for *.conf` travaille sur les vrais fichiers. `$(seq 1 10000)` fabrique 10 000 arguments en mémoire à chaque tour — lourd et fragile.
- **Utiliser `read -r`**, pas `read` seul : `-r` empêche l'interprétation des backslashes dans les lignes (noms de fichiers bizarres, logs).
- **Déclarer les fonctions avec `nom() { ...; }`** (sans le mot-clé `function`) et nommer leurs paramètres dès le départ : `local x="$1"` — tu ne confonds plus l'argument de fonction et `$1` du script.
- **Sortir d'une fonction par `return 0` / `return 1` explicite**, pas par absence : sinon elle renvoie le code de la dernière commande, inattendu.
- **Un bloc procédural répété ≥ 2 fois doit devenir une fonction** (même logique que le DRY en Java/JS).

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi c'est dangereux / inefficace | ✅ Version correcte |
|----------------|----------------------------------------|---------------------|
| `if [ $ENV = production ]` (sans guillemets) | Si `$ENV` est vide → `[ = production ]` → erreur de syntaxe. | `if [ "$ENV" = production ]` |
| `if [ "$a" > 5 ]` | `>` est une **redirection** vers un fichier `5`, pas une comparaison. | `if [ "$a" -gt 5 ]` ou `[[ "$a" -gt 5 ]]` |
| `if [ $a ]` sans espaces | `[$a]` n'est pas une commande valide → `command not found`. | `[ "$a" ]` avec espaces |
| `for i in $(seq 1 10000)` sur de gros compteurs | Fabrique des dizaines de milliers d'arguments en mémoire. | Itérer la liste réelle (`for *.log`) ou utiliser une boucle comptée proprement. |
| Boucler en lisant un fichier sans `-r` | Les backslashes dans les lignes sont interprétés → lignes corrompues. | `while IFS= read -r ligne; do ...; done < fichier` |
| Utiliser `function nom {}` | Syntaxe obsolète, moins portable (POSIX). | `nom() { ...; }` |
| Ne pas mettre `local` dans une fonction | La variable fuit dans la portée globale du script → effets de bord. | `local x="$1"` en début de fonction |
| Oublier `return` dans une fonction booléenne | Retourne le code de la dernière commande, aléatoire. | `return 0` / `return 1` explicite |

---

## 8. Checklist de validation

- [ ] Je sais écrire une condition `if/elif/else` avec `[[ ]]` et les opérateurs de comparaison (`-eq`, `-ne`, `-gt`, `-lt`).
- [ ] Je sais tester l'existence d'un fichier / dossier / variable vide (`-f`, `-d`, `-z`).
- [ ] Je sais écrire une boucle `for` sur une liste de valeurs et de fichiers.
- [ ] Je sais écrire une boucle `while` (lecture de fichier, polling HTTP).
- [ ] Je sais utiliser `case` pour un choix multiple.
- [ ] Je sais définir et appeler une **fonction** avec paramètres et `return`.

---

> 📖 Prochaine étape : fais l'**exercice pratique** dans `02-exercice.md`, puis compare avec `03-correction.md`.
