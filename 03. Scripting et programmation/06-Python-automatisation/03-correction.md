# Correction détaillée — Automatisation avec Python

> **Bloc 3 · Leçon 6** — Correction pas-à-pas de `02-exercice.md`. Suis chaque étape et compare à ton script.

---

## ✅ Étape 1 — venv + requests

```bash
mkdir -p ~/mon-check && cd ~/mon-check
python3 -m venv .venv
source .venv/bin/activate
pip install requests
pip freeze > requirements.txt
```

**Explication** : on isole `requests` dans un `venv` pour ne pas installer de dépendance dans le Python système (propre, reproductible) et on fige la version (lockfile) pour que le script tourne partout à l'identique.

---

## ✅ Étape 2 — Le squelette

```python
#!/usr/bin/env python3
"""Verifie la sante d'un service et ecrit un rapport."""

from pathlib import Path
import subprocess
import requests
import datetime
```

> ℹ️ Les imports sont regroupés en haut (bonne pratique vue en leçon 5).

---

## ✅ Étape 3 — Le corps complet

```python
rapports = Path('/tmp/rapports')
rapports.mkdir(parents=True, exist_ok=True)

# 1. uptime via subprocess
uptime = subprocess.run(['uptime'], capture_output=True, text=True)

# 2. appel /health avec gestion de connexion
url = 'http://localhost:8080/health'
try:
    resp = requests.get(url, timeout=5)
    if resp.status_code == 200:
        statut = 'sain'
    else:
        statut = f'indisponible (code {resp.status_code})'
except requests.exceptions.ConnectionError:
    statut = 'injoignable (connexion refusee)'

# 3. rapport horodate
horodatage = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')
contenu = f"{horodatage}\nUptime : {uptime.stdout.strip()}\nService : {statut}\n"
rapports.joinpath('rapport.txt').write_text(contenu)

print(f"Service {statut}")
```

**Explication des choix** :
- `mkdir(parents=True, exist_ok=True)` : crée le dossier (et ses parents) sans erreur s'il existe déjà.
- `subprocess.run(['uptime'], ...)` : **liste** d'arguments, `capture_output=True` capture la sortie, `text=True` la renvoie en chaîne. `uptime.stdout` contient «  up 2 days, ... ».
- `try/except requests.exceptions.ConnectionError` : quand aucun serveur n'écoute, `requests.get` lève une `ConnectionError` (et pas un statut HTTP). On la capture proprement au lieu d'un crash.
- Le statut gère 3 cas : 200 (sain), 200≠ (dispo mais erreur), connexion refusée.
- On écrit le rapport dans `/tmp/rapports/rapport.txt` via `joinpath().write_text()` (API `pathlib`).

> ⚠️ **Ordre des vérifications** : on traite l'exception de **connexion** d'abord, puis on teste `status_code` — l'inverse (`if ... else`) serait faux : une `ConnectionError` n'a pas de `status_code` accessible.

---

## ✅ Étape 4 — Résultat attendu

**Sans service sur le port 8080** :

```
Service injoignable (connexion refusee)
```

Et `cat /tmp/rapports/rapport.txt` :

```
2026-09-03 15:30:00
Uptime :  15:30:00 up 2 days, 1:20, 2 users, load average: 0.10, 0.12, 0.15
Service : injoignable (connexion refusee)
```

**Avec un serveur qui répond 200** sur `/health` :

```
Service sain
```

---

## ✅ Étape 5 — Réponses de l'auto-vérification

1. **Liste plutôt que chaîne** : en passant une **liste**, Python exécute le programme directement sans interpréter par un shell → pas de problème d'échappement, pas d'**injection** de commande, et plus sûr. Une chaîne exigerait `shell=True` (à éviter).
2. **`capture_output=True`** : redirige stdout/stderr du sous-processus vers des attributs `r.stdout` / `r.stderr` au lieu de les afficher. **`text=True`** : renvoie des `str` (au lieu de bytes bruts) — plus lisible.
3. **`timeout=5`** : sans lui, si le serveur ne répond pas (hors connexion refusée, ex. réseau silencieux), `requests` **bloque indéfiniment** et le script se fige. `timeout` force un abandon après 5 s.
4. **`resp.json()` sans vérifier le statut** : sur une erreur (404/500), le corps n'est pas du JSON → `requests.exceptions.JSONDecodeError` (plus rare) ou un dict inattendu ; le script plante ou traite de la mauvaise donnée.
5. **`venv` + `requests`** : on ne contamine pas le Python système (versions, dépendances), on fige dans `requirements.txt`, et on estime le script reproductible sur n'importe quelle machine.

---

## 📝 Checklist de validation (récapitulatif + conseils)

- [ ] J'ai un `venv` actif avec `requests` installé et `requirements.txt` créé.
- [ ] J'utilise `subprocess.run(['uptime'], capture_output=True, text=True)` et je lis `stdout`.
- [ ] J'ai créé `/tmp/rapports` avec `pathlib` `mkdir(parents=True, exist_ok=True)`.
- [ ] Je fais `requests.get(url, timeout=5)` dans un `try/except requests.exceptions.ConnectionError`.
- [ ] Je vérifie `resp.status_code == 200` AVANT d'utiliser la réponse.
- [ ] J'écris le rapport horodaté avec `joinpath(...).write_text(...)`.
- [ ] J'ai testé le cas « service absent » (connexion refusée) sans crash.

### 💡 Conseils pour la suite

- **Re-teste-toi** dans 2-3 jours en remplaçant l'endpoint par une autre API (ex. GitHub) et en encodant le JSON dans le rapport.
- **Réflexe pro** : un script d'automatisation = toujours les mêmes habitudes — `venv` + haut de fichier (imports) + `try/except` ciblé + exits codes + logs horodatés. Tu as maintenant la base.
- La **Leçon 7 (projet récapitulatif)** combine le tout : vérifier un serveur, l'espace disque, un service, analyser les logs, et envoyer un rapport — le critère « bloc acquis » de la roadmap.

---

*Prochaine étape :* Leçon 7 — **Projet : script de diagnostic & rapport** → dossier `07-Projet-recapitulatif-scripting/`.
