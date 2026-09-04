# Exercice pratique — Automatisation avec Python

> **Bloc 3 · Leçon 6** — Exercice à faire en autonomie.
> Contexte : tu dois écrire un script d'automatisation qui **vérifie la santé d'un service** et génère un rapport. Il combine les 3 briques de la leçon : lancer une commande (`subprocess`), gérer un dossier (`pathlib`), et interroger un endpoint `/health` (`requests`). C'est le prélude direct au projet récapitulatif de la leçon 7.

---

## 🎯 Objectif de l'exercice

Écrire `check_service.py` qui :
- vérifie que le dossier `/tmp/rapports` existe (le crée si besoin) ;
- exécute `uptime` avec `subprocess` et le consigne ;
- appelle un endpoint `/health` avec `requests` (GET) ;
- selon le statut HTTP, affiche « service sain » ou « service indisponible » ;
- écrit un rapport texte horodaté dans `/tmp/rapports/`.

---

## 📋 Étape 1 — Préparation (venv)

Crée un dossier `~/mon-check` avec un `venv`, active-le, installe `requests`, fige dans `requirements.txt`.

```bash
mkdir -p ~/mon-check && cd ~/mon-check
python3 -m venv .venv && source .venv/bin/activate
pip install requests
pip freeze > requirements.txt
```

---

## 📋 Étape 2 — Le squelette

Crée `check_service.py` avec, en haut :

```python
#!/usr/bin/env python3
from pathlib import Path
import subprocess, requests, datetime
```

---

## 📋 Étape 3 — Le corps

1. Crée le dossier `/tmp/rapports` avec `Path('/tmp/rapports').mkdir(parents=True, exist_ok=True)`.
2. Exécute `subprocess.run(['uptime'], capture_output=True, text=True)` et garde `r.stdout`.
3. Fais un `requests.get('http://localhost:8080/health', timeout=5)` dans un `try/except requests.exceptions.ConnectionError`.
4. Si `status_code == 200` : `statut = 'sain'`, sinon `statut = 'indisponible (code X)'`.
5. Écris dans `/tmp/rapports/rapport.txt` un texte horodaté contenant l'uptime et le statut.
6. Affiche `f'Service {statut}'` en console.

---

## 📋 Étape 4 — Double test

Exécute une fois **sans** service sur le port 8080, puis (optionnel) avec un petit serveur local si tu en as un. Note les deux comportements (le `except ConnectionError` doit s'activer quand rien n'écoute).

```bash
cd ~/mon-check && source .venv/bin/activate
python3 check_service.py
cat /tmp/rapports/rapport.txt
```

---

## 📋 Étape 5 — Auto-vérification

1. Pourquoi `subprocess` demande une **liste** d'arguments plutôt qu'une chaîne ?
2. Que fait `capture_output=True` ? et `text=True` ?
3. Pourquoi `timeout=5` est obligatoire sur `requests.get` ?
4. Quel est le risque si on appelle `resp.json()` sans vérifier le statut ?
5. Pourquoi garder `requests` dans un `venv` et non dans le Python global ?

---

## 🏁 Rendu attendu

Un script `check_service.py` fonctionnel, un `rapport.txt` généré dans `/tmp/rapports/`, et tes 5 réponses écrites.

> Compare ensuite avec `03-correction.md`.
