# Aide-mémoire — Automatisation avec Python

> **Bloc 3 · Leçon 6** — Fiche de référence pour automatiser commandes, fichiers et API en Python.

## 📌 subprocess — lancer une commande

```python
import subprocess
r = subprocess.run(['df', '-h'], capture_output=True, text=True)
print(r.returncode)   # 0 = succes
print(r.stdout)       # sortie standard
print(r.stderr)       # erreur
```

> ⚠️ Toujours une **liste** d'arguments, jamais `shell=True` (risque d'injection).

## 📌 pathlib / shutil — fichiers et dossiers

```python
from pathlib import Path
import shutil, os

d = Path('/tmp/rapports')
d.mkdir(parents=True, exist_ok=True)   # cree sans erreur
print(d.exists())
for f in Path('/tmp').iterdir():        # liste
    print(f.name)
shutil.copy(Path('/a/x'), Path('/b/x.bak'))   # copier
os.listdir('/tmp')                       # equivalent historique
```

## 📌 requests — appels HTTP (à installer)

```bash
python3 -m venv .venv && source .venv/bin/activate
pip install requests
pip freeze > requirements.txt
```

```python
import requests

# GET
resp = requests.get('https://api.github.com/repos/octocat/Hello-World', timeout=10)
print(resp.status_code)     # 200
print(resp.json())          # JSON -> dict

# GET avec params + headers
resp = requests.get(url, params={'per_page': 3}, headers={'Accept': 'application/json'}, timeout=10)

# POST
resp = requests.post('https://api.exemple/reports', json={'title': 'x'}, timeout=10)
```

> ⚠️ Toujours `timeout=`. Toujours vérifier `status_code` avant `.json()`.

## 📌 Gestion d'erreurs réseau

```python
try:
    resp = requests.get(url, timeout=5)
except requests.exceptions.ConnectionError:
    print('Connexion refusee')
    exit(1)
except requests.exceptions.Timeout:
    print('Timeout')
    exit(1)

if resp.status_code != 200:
    print(f'Statut {resp.status_code}')
    exit(1)
```

## 📌 Exceptions requests courantes

| Exception | Sens |
|-----------|------|
| `ConnectionError` | serveur injoignable / connexion refusée |
| `Timeout` | le serveur ne répond pas dans le délai |
| `HTTPError` | erreur HTTP (4xx/5xx) si `raise_for_status()` utilisé |

## 📌 Règles d'or

- `subprocess.run([...], capture_output=True, text=True)` par défaut.
- Jamais de `shell=True` avec des entrées dynamiques.
- `timeout=` sur chaque appel réseau.
- Sortie HTTP : statut d'abord, contenu ensuite.
- `venv` + `requirements.txt` pour toute dépendance.
- `try/except` ciblé (pas `except Exception` nu).
