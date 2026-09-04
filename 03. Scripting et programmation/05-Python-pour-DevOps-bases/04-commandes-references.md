# Aide-mémoire — Introduction à Python pour DevOps

> **Bloc 3 · Leçon 5** — Fiche de référence pour tes premiers scripts Python.

## 📌 Lancer / environnement

```bash
python3 --version
python3 mon_script.py            # exécuter
chmod +x mon_script.py && ./mon_script.py
python3 -m py_compile mon_script.py   # vérifier la syntaxe sans exécuter

python3 -m venv .venv
source .venv/bin/activate
pip install requests
pip freeze > requirements.txt
```

## 📌 Variables / types / print

```python
port = 8080              # entier
nom = "api"              # chaîne
actif = True             # booléen
print(f"port {port}, nom {nom}")
```

## 📌 Listes `[]` et dicts `{}`

```python
envs = ["prod", "staging"]
envs.append("dev")
print(envs[0])           # prod

conf = {"name": "api", "replicas": 3}
print(conf["name"])      # api
conf["region"] = "eu"
```

## 📌 Conditions / boucles

```python
if port < 1024:
    print("privilegie")
elif port < 65536:
    print("user")
else:
    print("invalide")

for env in envs:
    print(env)
```

## 📌 Fichiers avec pathlib

```python
from pathlib import Path
p = Path("app.log")
p.write_text("ligne1\nligne2\n")
texte = p.read_text()
lignes = texte.splitlines()
```

## 📌 JSON

```python
import json
d = json.loads('{"name": "api"}')   # JSON → dict
print(d["name"])
js = json.dumps(d, indent=2)          # dict → JSON
```

## 📌 Gestion d'erreurs

```python
try:
    x = int("abc")
except ValueError as e:
    print("erreur:", e)
```

## 📌 Règles d'or

- Imports **en haut** du fichier.
- **4 espaces** d'indentation, jamais de tabulation.
- Deux-points `:` après `if`/`for`/`while`/`def`.
- `snake_case` pour noms de variables et fichiers.
- F-strings `f"..."` plutôt que concaténation.
- `try/except` **ciblé** (type précis), jamais `except:` nu.
