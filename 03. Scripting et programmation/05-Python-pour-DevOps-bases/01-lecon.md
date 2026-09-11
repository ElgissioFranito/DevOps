# Leçon 5 — Introduction à Python pour DevOps

> **Bloc 3 · Scripting et programmation** — Leçon 5 sur 7
> Tu n'as **jamais fait de Python** — et c'est justement là qu'on commence. Python est devenu **le** langage de scripting DevOps moderne (Ansible, outils d'automatisation, scripts d'admin). Tu sais déjà coder en Java/JS : cette leçon te montre comment **transposer tes réflexes** vers une syntaxe plus concise et des concepts typiquement Python (indentation, dictionnaires, listes).
> ⚠️ **Prérequis** : les blocs 2 et 3 (leçons 1-4) sont supposés acquis — shell, scripts Bash, variables, et surtout YAML/JSON.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Expliquer** ce qu'est Python, pourquoi le DevOps moderne l'utilise, et configurer un environnement avec `venv` + `pip`.
2. **Écrire** un script Python simple : variables, types, `print()`, commentaires.
3. **Utiliser** les **listes** et **dictionnaires** (l'équivalent des arrays/objets que tu connais déjà).
4. **Écrire** des **boucles** (`for`, `while`) et des **conditions** (`if`/`elif`/`else`) en Python.
5. **Lire/écrire** des fichiers texte et du **JSON** avec les modules `json` et `pathlib`.
6. **Gérer** les premières erreurs avec `try/except`.

---

## 2. Explication simple

### 2.1 Le « pourquoi » : Python, le couteau suisse du DevOps

En DevOps on écrit beaucoup de **petits scripts d'automatisation** : lire un fichier de logs, transformer du JSON, appeler une API, déplacer des fichiers, nettoyer un serveur. Bash est parfait pour « coller des commandes », mais dès qu'il faut **manipuler des données** (JSON, YAML, CSV), des **conditions complexes**, des **fichiers**, ou du **réseau** (HTTP, API), Python est **beaucoup** plus lisible et sûr.

L'analogie : si Bash est la **boîte à outils manuelle** du plombier, Python est le **compagnon polyvalent** qu'on emmène sur tous les chantiers.

Les raisons de ce choix en 2025-2026 :
- **Écosystème DevOps** : Ansible (configuration), Prometheus exporters, scripts cloud, outils d'automatisation — la majorité tourne en Python.
- **Lisible** : Python « se lit comme l'anglais » (`for x in items:`) — idéal pour des scripts que tu reliras dans 6 mois.
- **Batteries incluses** : modules standard pour JSON, fichiers, réseau, sans rien installer.

### 2.2 Le « comment » : la syntaxe Python, ses 3 différences clés

Par rapport à Java/JS, trois choses déroutent au début :

1. **L'indentation EST la structure** : pas d'accolades `{}`. Les blocs `if`/`for`/`function` sont délimités par **l'indentation** (espaces) et un **deux-points**.

```python
# Java :  if (age > 18) { ... }
# Python :
if age > 18:
    print("majeur")      # indenté = dans le if
print("fin")             # non indenté = hors du if
```

2. **Les types sont dynamiques mais typés** : pas de déclaration `int x = 5`. On écrit juste `x = 5`. Le type est inféré.

3. **Les blocs « fonction »** se déclarent avec `def`, et le **retour** avec `return` (comme en JS).

### 2.3 Le « quand » : Bash ou Python ?

| Situation | Outil |
|-----------|-------|
| Enchaîner des commandes système, pipes | Bash (leçons 1-3) |
| Manipuler JSON/YAML/fichiers/API | **Python** (les 3 prochaines leçons incl. celle-ci) |
| Logique complexe, gestion d'erreurs | **Python** |
| Script lancé très souvent / ultra-léger | Bash |
| Script volumineux, maintenable, testable | **Python** |
---

## 📖 Vocabulaire / Abréviations

| Terme | Définition (une ligne) |
|---|---|
| **Interpréteur** | le programme qui exécute ton code Python ligne à ligne (`python3`) |
| **Module / import** | fichier de fonctions réutilisables, chargé avec `import` |
| **pip** | le gestionnaire de paquets Python (installe des bibliothèques) |
| **venv** | environnement virtuel : un « bac à sable » isolant les dépendances d'un projet |
| **f-string** | chaîne de caractères avec variables intégrées : `f"Bonjour {nom}"` |
| **REPL** | mode interactif (tu tapes, Python répond) lancé par `python3` sans argument |
| **Shebang** (`#!/usr/bin/env python3`) | première ligne qui dit au système d'utiliser Python |
| **Type** | le genre d'une valeur (texte `str`, nombre `int`, liste `list`, booléen `bool`) |

---

## 3. Exemples concrets

### 3.1 Premier script : variables et `print()`

```python
#!/usr/bin/env python3
# mon_script.py
app = "api-demo"          # variable (type inféré)
port = 8080
actif = True              # booléen

print(app)                # api-demo
print(f"Port : {port}")   # f-string : injection de variable
print(f"{app} actif = {actif}")
```

```bash
chmod +x mon_script.py
./mon_script.py
# ou :
python3 mon_script.py
```

### 3.2 Listes et dictionnaires

```python
# Liste (= like un tableau JS)
environnements = ["production", "staging", "dev"]
print(environnements[0])        # production
environnements.append("test")   # ajoute à la fin
for env in environnements:
    print(env)

# Dictionnaire (= like un objet JS / un JSON)
config = {
    "name": "api",
    "replicas": 3,
    "ports": [80, 443],
}
print(config["name"])       # api
print(config["ports"][0])   # 80
config["region"] = "eu-west-1"   # ajout d'une clé
```

### 3.3 Conditions et boucles

```python
port = 8080
if port < 1024:
    print("port privilégié")
elif port < 65536:
    print("port utilisateur")
else:
    print("port invalide")

compteur = 0
while compteur < 3:
    print(f"essai {compteur}")
    compteur += 1
```

### 3.4 Lire/écrire des fichiers avec `pathlib`

```python
from pathlib import Path

chemin = Path("/tmp/mon-app.log")

# écrire
chemin.write_text("Ligne 1\nLigne 2\n")

# lire
print(chemin.read_text())
# → Ligne 1
#   Ligne 2

# vérifier l'existence
print(chemin.exists())   # True
```

### 3.5 Travailler avec du JSON (`json` module)

```python
import json

# d'un fichier JSON vers un dictionnaire Python
donnees = json.loads('{"name": "api", "replicas": 3}')
print(donnees["replicas"])          # 3

# d'un dictionnaire vers une chaîne JSON
texte = json.dumps({"name": "api", "replicas": 3}, indent=2)
print(texte)
```

### 3.6 Gestion d'erreurs : `try/except`

```python
try:
    nombre = int("pas-un-nombre")
except ValueError as e:
    print("Conversion impossible :", e)
# Programme continue sans planter
```

### 3.7 Environnement virtuel : `venv` + `pip` (bonne pratique)

```bash
python3 -m venv .venv              # crée un environnement isolé
source .venv/bin/activate          # l'active (sur Linux/macOS)
pip install requests               # installe une lib dans l'environnement
pip freeze > requirements.txt      # fige les dépendances
# ... plus tard :
deactivate                          # quitte l'environnement
```

> 💡 Vérifie avec `which python3` avant/après `activate` : le chemin pointe vers `.venv/bin/python3`, signe qu'on travaille dans l'environnement isolé.
### 3.8 🛡️ Aperçu DevSecOps — appeler des commandes système avec `subprocess`

En DevOps, Python sert souvent à **lancer des commandes système** (`df`, `systemctl`, `docker`…). L'outil standard est le module `subprocess` :

```python
import subprocess

result = subprocess.run(
    ["df", "-h", "/"],          # ✅ une LISTE d'arguments
    capture_output=True,
    text=True,
    check=True,                  # lève une erreur si la commande échoue
)
print(result.stdout)
```

> ⚠️ **Règle de sécurité (DevSecOps)** : passe toujours une **liste** d'arguments, et **jamais `shell=True`** avec une donnée qui vient de l'utilisateur ou d'un fichier externe — sinon tu ouvres la porte à une **injection de commande** (quelqu'un fait exécuter `; rm -rf /` à ton script). Exemple interdit : `subprocess.run(f"ls {dossier}", shell=True)` si `dossier` n'est pas contrôlé.
>
> Ce sujet est approfondi dans la **leçon 6** (Python automatisation) — ici, retiens juste le réflexe : *liste d'arguments, pas de `shell=True` sur des entrées externes*.

---
---

## 4. Bonnes pratiques modernes (2025-2026)

- **Toujours utiliser `venv`** pour un projet (même petit) : isoler les dépendances, éviter de polluer le Python système. C'est l'équivalent d'un `package.json`/`node_modules` en JS.
- **Figer les dépendances** dans `requirements.txt` (`pip freeze > requirements.txt`) pour la reproductibilité — comme un lockfile.
- **`f-strings` plutôt que la concaténation** : `f"port {port}"` au lieu de `"port " + str(port)`. Plus lisible et performant.
- **`pathlib` plutôt que `os.path`** : la nouvelle API moderne (2020+) pour les chemins ; plus propre et portable.
- **Utiliser `json.load`/`json.dumps`** plutôt que de parser à la main — le standard pour le JSON en Python.
- **`try/except` ciblé** : attraper **des exceptions précises** (`ValueError`) plutôt qu'une exception générique. Ne jamais cache silencieusement.
- **Nommer ses variables en `snake_case`** (`mon_script.py`, `app_port`) — c'est la convention Python.
- **Passer son code dans un linter** : `ruff check .` ou `python3 -m py_compile` (vérifie la syntaxe sans exécuter).

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi c'est un problème | ✅ Version correcte |
|----------------|------------------------------|---------------------|
| Mélanger espaces et **tabulations** dans l'indentation | L'indentation est la structure en Python → `IndentationError`. | Toujours **4 espaces** (standard), jamais de tab. |
| Oublier les **deux-points** après `if`/`for`/`while`/`def` | `SyntaxError: invalid syntax`. | `if x > 5:` (les deux-points obligatoires). |
| Tableau/objet JS mélangé | Python liste `[]` ≠ dict `{}` ; confusion de syntaxe. | Liste = `[...]`, dict = `{...}` avec `:` entre clé/valeur. |
| `print("x = " + x)` avec un nombre | Concaténation chaîne+nombre → `TypeError`. | `print(f"x = {x}")` (f-string). |
| `import` en bas du script | Peu lisible ; les imports sont toujours en **haut**. | Tous les `import` au début du fichier. |
| `except:` (sans type) qui avale tout | Cache toutes les erreurs (programmation, système…) → bugs invisibles. | Attraper `except ValueError:` et loguer. |
| Installer des libs dans le Python global | Pollue le système, conflits de versions. | Toujours `venv` + `pip`. |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`**, la correction commentée dans **`03-correction.md`**. Lis bien cette leçon avant de t'y mettre.

**Énoncé court** : dans un `venv`, écris un petit script Python `mon_inventaire.py` : crée un **dictionnaire** décrivant un service (nom, port, tags), une **liste** de 3 services, boucle dessus avec `for`, écrit la structure en **JSON** dans un fichier (module `json`), recharge-le avec `pathlib`, et gère une erreur ciblée avec `try/except`.

---

## 7. Correction détaillée de l'exercice

> La correction complète pas-à-pas est dans **`03-correction.md`**. Essentiel du raisonnement :
- on travaille dans un **`venv`** (`python3 -m venv .venv && source .venv/bin/activate`) pour isoler ;
- **listes** `[]` et **dictionnaires** `{}` (`for service in services:` pour itérer) ;
- **`json.dump`** pour écrire et **`json.load`** pour relire — jamais de parsing à la main ;
- **`pathlib`** (`Path`) moderne pour les chemins ;
- **`try/except ValueError`** ciblé, jamais d'exception générique qui cache tout.

---

## 8. Checklist de validation

- [ ] Je sais installer/configurer un environnement `venv` + `pip` et l'activer (`source .venv/bin/activate`).
- [ ] J'écris un script Python simple (variables, `print`, f-strings, commentaires) exécutable avec `./mon_script.py`.
- [ ] J'utilise correctement les **listes** `[]` et **dictionnaires** `{}` et je sais boucler dessus.
- [ ] J'ai une **indentation** correcte (`if`, `for`, `def`) sans mélange espace/tab.
- [ ] Je lis/écris un **fichier** avec `pathlib` et du **JSON** avec le module `json`.
- [ ] Je gère une erreur avec `try/except` ciblé.
- [ ] J'ai figé mes dépendances dans `requirements.txt`.

---

🧭 **Pont vers la suite** — Tu sais maintenant écrire du **Python** (variables, listes/dicts, fichiers, JSON). Mais un script DevOps ne se limite pas au disque : il **interagit** — avec des fichiers, des commandes système et surtout des **APIs HTTP**. La **Leçon 6** te fait passer à l'automatisation réelle : `subprocess`, `requests` (HTTP), et manipulation de données.

---

*Prochaine étape :* Leçon 6 — **Automatisation avec Python** dans `06-Python-automatisation/`.

> 💡 **Lien avec tes acquis** : une **liste** Python ≈ un tableau JS ; un **dictionnaire** Python ≈ un objet JS ; un `for` Python ≈ un `for...of` JS. Tu retrouves tes concepts, juste une autre syntaxe.