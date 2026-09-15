# Leçon 6 — Automatisation avec Python

> **Bloc 3 · Scripting et programmation** — Leçon 6 sur 7
> Tu sais maintenant les bases de Python (leçon 5). Place à l'automatisation **réelle** : exécuter des commandes système depuis Python (`subprocess`), manipuler des fichiers/dossiers (`os`/`pathlib`), et **appeler des API HTTP** (`requests`) — le cœur du scripting DevOps moderne et le point « API / SDK / REST » de la roadmap. Tu es développeur Spring/NestJS : tu connais déjà les verbes HTTP, ici tu apprends à t'en servir *depuis un script*.
> ⚠️ **Prérequis** : les leçons 5 (bases Python) et 4 (YAML/JSON) sont supposées acquises.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Exécuter** une commande système depuis Python avec `subprocess.run()` et en lire la sortie.
2. **Manipuler** fichiers et dossiers avec `os` et `pathlib` (créer, copier, lister, vérifier).
3. **Faire un appel API HTTP** avec la lib `requests` : `GET`, `POST`, paramètres, en-têtes, gestion du statut.
4. **Lire une réponse JSON** d'API depuis un script.
5. **Assembler** ces briques en un petit script d'automatisation **fiabilisé** (gestion d'erreurs, exit codes).
6. **Expliquer** pourquoi Python + `requests` est l'approche moderne de l'automatisation d'API (vs `curl` seul).

---

## 2. Explication simple

### 2.1 Le « pourquoi » : automatiser plus que de simples commandes

Au bloc 3 leçons 1-3, Bash excellait pour la « colle entre commandes ». Mais dès qu'il faut **lancer une commande système ET récupérer sa sortie**, **tester une API et parser le JSON résultat**, ou **enchaîner plusieurs actions avec des conditions**, Python devient plus lisible et plus maintenable.

L'automatisation DevOps typique combine 3 briques :

```
1. subprocess → lancer une commande, inclure sa sortie
2. os/pathlib → gérer les fichiers/dossiers de l'environnement
3. requests  → dialoguer avec une API (GitHub, Healthcheck, etc.)
```

### 2.2 Le « comment » : trois briques, trois modules

**1. `subprocess`** : exécuter une commande système depuis Python. C'est l'équivalent Python de `$(...)` en Bash, mais avec un contrôle fin (capture de la sortie, code de retour).

```python
import subprocess
r = subprocess.run(["ls", "-l"], capture_output=True, text=True)
print(r.returncode)   # 0 = succès
print(r.stdout)       # sortie standard
print(r.stderr)       # erreur éventuelle
```

**2. `os` / `pathlib`** : gérer le système de fichiers (créer un dossier, lister, copier). `pathlib` est la modern API (objet `Path`), `os` est l'historique.

**3. `requests`** : faire des requêtes HTTP. Le module standard `urllib` existe mais `requests` est **++ la norme** (concise, lisible). Elle n'est pas incluse : `pip install requests`.

```
requête HTTP --> serveur --> réponse (status + JSON)
```

### 2.3 Le « quand » : automatiser une API, pour qui ?

- **S'appeler soi-même** : vérifier qu'un service répond en passant par son endpoint `/health` (santé).
- **Interroger des services externes** : GitHub API (listes de repos), registres, etc.
- **Orchestrer** : déclencher un build/une action, récupérer un artefact.
---

## 📖 Vocabulaire / Abréviations

| Terme | Définition (une ligne) |
|---|---|
| **subprocess** | module Python qui lance des commandes shell et récupère leur résultat |
| **returncode** | code de retour d'une commande lancée (0 = succès) |
| **argparse** | module standard pour gérer les options/arguments d'un script en ligne de commande |
| **Requests** | bibliothèque populaire pour appeler des API HTTP (hors standard, à installer) |
| **Cron** | planificateur Linux : exécute une commande à heure fixe (voir Bloc 2) |
| **Journalisation (logging)** | écrire des événements horodatés pour diagnostiquer ensuite |
| **Gestion d'erreur** | prévoir le cas « ça échoue » (try/except) au lieu de crasher |
| **Refactorisation** | améliorer le code sans changer son comportement |

---

## 3. Exemples concrets

### 3.1 `subprocess` : lancer une commande et lire sa sortie

```python
#!/usr/bin/env python3
import subprocess

# lancer `df -h` (espace disque) et capturer la sortie
r = subprocess.run(["df", "-h", "/"], capture_output=True, text=True)

print("Code de retour :", r.returncode)   # 0 si succès
print(r.stdout)                            # la sortie standard
```

Si la commande échoue (code non nul), testons-le :

```python
r = subprocess.run(["false"], capture_output=True, text=True)
print(r.returncode)     # 1 (commande `false` retourne toujours un échec)
```

> 💡 `capture_output=True` intercepte stdout/stderr ; `text=True` renvoie des **chaînes** (au lieu de bytes). Sans ces options, la sortie est silencieuse et en bytes.

### 3.2 `pathlib` / `os` : gérer fichiers et dossiers

```python
from pathlib import Path
import shutil

src = Path("/tmp/mon-app")
dst = Path("/tmp/backups")

# vérifier / créer des dossiers
print(src.exists())          # True/False
dst.mkdir(parents=True, exist_ok=True)   # crée, sans erreur si présent

# lister les fichiers d'un dossier
for f in src.iterdir():
    print(f.name)            # nom du fichier/dossier

# copier un fichier
shutil.copy(src / "app.log", dst / "app.log.bak")
```

### 3.3 `requests` : un appel GET et le JSON

```python
import requests

# Exemple public : https://api.github.com/repos/octocat/Hello-World
resp = requests.get("https://api.github.com/repos/octocat/Hello-World", timeout=10)

print(resp.status_code)      # 200
donnees = resp.json()        # JSON réponse -> dict Python
print(donnees["full_name"])  # octocat/Hello-World
print(donnees["stargazers_count"])
```

### 3.4 `requests` : GET avec paramètres et en-têtes, gestion du statut

```python
import requests

headers = {"Accept": "application/vnd.github+json"}
params = {"per_page": 3}                    # limitons le résultat

resp = requests.get("https://api.github.com/users/octocat/repos",
                    params=params, headers=headers, timeout=10)

if resp.status_code != 200:
    print(f"API a répondu {resp.status_code}")
    exit(1)

for repo in resp.json():
    print(repo["name"])
```

### 3.5 `requests` : POST (créer une ressource)

```python
import requests

payload = {"title": "Nouveau rapport", "body": "contenu"}
resp = requests.post("https://api.exemple.com/reports",
                     json=payload, timeout=10)

print(resp.status_code)      # 201 Created en cas de succès
```

### 3.6 Vérification d'un endpoint de santé (`/health`)

```python
import requests

url = "http://localhost:8080/health"
try:
    resp = requests.get(url, timeout=5)
    if resp.status_code == 200:
        print("Service sain")
    else:
        print(f"Service indisponible : {resp.status_code}")
        exit(1)
except requests.exceptions.ConnectionError:
    print("Impossible de joindre le service")
    exit(1)
```

> 💡 C'est exactement ce que font les health checks des orchestrateurs (Docker/K8s) et des load balancers.

### 3.7 Installer `requests` (et figer)

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install requests
pip freeze > requirements.txt   # contient désormais requests==...
```
---

## 4. Bonnes pratiques modernes (2025-2026)

- **`subprocess.run()` avec `capture_output=True, text=True`** par défaut : c'est la signature moderne et sûre (pas de shell interposé, pas d'injection).
- **Ne jamais utiliser `shell=True`** sauf cas très précis et contrôlé : ça exécute une commande via le shell (risque d'injection de commande). Passer une **liste** d'arguments (`["ls", "-l"]`) plutôt qu'une chaîne.
- **`requests` avec `timeout=` obligatoire** : sans timeout, un appel bloqué gèle ton script indéfiniment (dès 2025, timeout devient un réflexe de sécurité).
- **Toujours vérifier `resp.status_code`** et gérer les erreurs HTTP avant de parser le JSON.
- **Utiliser `.json()`** de `requests` (plutôt que `json.loads(resp.text)`) : c'est fait pour ça, et ça gère le parsing proprement.
- **`pathlib` (API `Path`)** plutôt que `os.path` + chaînes : plus lisible, plus portable, moins de bugs de concaténation de chemins.
- **Figer les dépendances** (`pip freeze > requirements.txt`) et travailler dans un `venv` : reproductibilité du script sur toute machine.
- **Gérer les erreurs aux bons endroits** : `try/except` ciblé sur `subprocess` (échec commande) et `requests` (connexion/temps). Ne jamais tout avaler.

### 4.1 Vérifier son script : pytest en 10 minutes

> 🧭 **Transition** : ton script tourne (peut-être en cron, vu au Bloc 2) — mais qui vérifie qu'il fonctionne *encore* après chaque modification ? Le **test unitaire** (Bloc 1, Leçon 3) automatisé avec **pytest**, l'outil standard Python.

**Le principe** : mettre la logique dans des **fonctions** (mêmes entrées → même sortie), puis écrire dans un fichier `test_*.py` des vérifications avec **`assert`** (« affirme que… » — si c'est faux, le test échoue). `pytest` découvre et lance tous ces tests en une commande.

```python
# surveillance.py — la logique en fonction testable
def statut_disque(pourcentage, seuil=80):
    if pourcentage < 0:
        return "ERREUR"
    return "ALERTE" if pourcentage >= seuil else "OK"

# test_surveillance.py — un test = un comportement, nommé clairement
from surveillance import statut_disque

def test_statut_seuil_exact():
    assert statut_disque(80, 80) == "ALERTE"    # cas limite : au seuil exact
```

```bash
python3 -m venv .venv && source .venv/bin/activate   # venv (Leçon 5)
pip install pytest                                    # installe l'outil de test
pytest -v            # -v : verbeux ; affiche chaque test et son verdict (passed/failed)
```

> 💡 **Pourquoi c'est utile** : un échec pytest te donne la ligne, la valeur obtenue et l'attendue — et au **Bloc 11 (CI/CD)**, la pipeline lancera ces tests à chaque commit et bloquera le déploiement en cas d'échec. C'est le « détecteur de fumée » branché sur ton code. Garde deux règles : **tests rapides et isolés** (jamais d'appel réseau réel) et **on corrige le code, pas le test** quand un échec révèle un bug.

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi c'est un problème | ✅ Version correcte |
|----------------|------------------------------|---------------------|
| `subprocess.run("ls -l")` en **chaîne** sans `shell` | `FileNotFoundError` : une chaîne n'est pas exécutable telle quelle. | Passer une **liste** `["ls", "-l"]`, ou `shell=True` (à éviter). |
| `subprocess.run(... , shell=True)` avec une entrée utilisateur | **Injection de commande** (sécurité). | Toujours une **liste d'arguments**, jamais shell vrai avec données. |
| `requests.get(url)` **sans timeout** | Appel qui bloque le script indéfiniment. | `requests.get(url, timeout=10)`. |
| Lire `resp.json()` sans vérifier le statut | Sur une erreur (404/500), le corps n'est pas le JSON attendu → `JSONDecodeError`. | Vérifier `if resp.status_code != 200: exit(1)` avant `.json()`. |
| Confondre sortie de `subprocess` et `stdout` | Sans `capture_output`, pas de sortie capturée → `r.stdout` vide. | `capture_output=True, text=True`. |
| Ignorer `returncode` d'une commande | Le script continue alors que la commande a échoué (même piège qu'en Bash sans `set -e`). | Vérifier `r.returncode`, gérer l'échec comme une erreur (exit(1)). |
| `requests` dans le Python global sans venv | Version incohérente selon les machines. | `venv` + `pip freeze > requirements.txt`. |
| Attraper toutes les exceptions avec `except Exception` | Cache l'erreur de programmation en plus de l'erreur d'environnement. | Cibler `requests.exceptions.*` et `FileNotFoundError` précisément. |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`**, la correction commentée dans **`03-correction.md`**. Lis bien cette leçon avant de t'y mettre.

**Énoncé court** : écris un script `verif_api.py` qui (1) lance une commande système avec `subprocess.run([...])` (ex. `df -h /`), (2) fait un appel HTTP GET vers une API de type « health » avec `requests` et affiche son statut, (3) traite la réponse JSON, (4) gère une erreur ciblée (service injoignable ou fichier absent), et lit ses éventuelles valeurs de configuration depuis `os.environ` plutôt qu'en dur.

---

## 7. Correction détaillée de l'exercice

> La correction complète pas-à-pas est dans **`03-correction.md`**. Essentiel du raisonnement :
- `subprocess.run([...])` avec une **liste d'arguments** (pas de `shell=True` sur entrée non sûre) ;
- `requests.get(...)` avec vérification du `status_code`, `.json()` pour parser la réponse ;
- **gérer les erreurs réseau** (`requests.exceptions`) de façon ciblée et **journaliser** ;
- ne **jamais hardcoder** d'URL/secret : on lit via `os.environ.get("URL")` ;
- retourner un **code de sortie** cohérent (0 OK, 1 problème) exploitable.

---

## 8. Checklist de validation

- [ ] Je lance une commande avec `subprocess.run(["cmd", "arg"], capture_output=True, text=True)` et je lis `returncode` / `stdout` / `stderr`.
- [ ] J'évite `shell=True` et je passe des **listes** d'arguments.
- [ ] Je crée/lisie/copie des fichiers et dossiers avec `pathlib` (+ `shutil` si copie).
- [ ] Je fais un `requests.get()` avec `timeout=`, je vérifie `status_code`, et je lis `.json()`.
- [ ] Je fais un `requests.post()` avec `json=payload` pour créer une ressource.
- [ ] J'ai un `try/except` ciblé pour les erreurs de connexion (`requests.exceptions.ConnectionError`).
- [ ] J'ai `requests` dans un `venv` et figé dans `requirements.txt`.

---

🧭 **Pont vers la suite** — Tu maîtrises maintenant Bash **et** Python, les données (YAML/JSON), et l'automatisation (system, fichiers, HTTP). Il est temps de **tout assembler** : c'est la **Leçon 7**, le projet récapitulatif du bloc, qui construit un **outil de diagnostic système complet** (serveur, disque, service, logs, rapport) — le critère « bloc acquis » de la roadmap.

---

*Prochaine étape :* Leçon 7 — **Projet : script de diagnostic & rapport** dans `07-Projet-recapitulatif-scripting/`.

> 💡 **Lien avec tes acquis Java** : un appel `requests.get(...)` = ton `RestTemplate`/`WebClient` côté Java ; le JSON réponse se convertit en dict Python (comme une désérialisation en objet/DTO). Tu connais déjà les verbes et codes HTTP — seule la syntaxe d'appel change.