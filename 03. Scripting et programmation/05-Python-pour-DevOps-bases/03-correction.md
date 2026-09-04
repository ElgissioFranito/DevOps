# Correction détaillée — Introduction à Python pour DevOps

> **Bloc 3 · Leçon 5** — Correction pas-à-pas de `02-exercice.md`. Suis chaque étape et compare à ton script.

---

## ✅ Étape 1 — Le fichier de logs

```bash
mkdir -p ~/mon-python
cd ~/mon-python
nano app.log
```

```
INFO Server started
ERROR Database connection failed
INFO Health check OK
ERROR Timeout after 5s
INFO Server started
INFO Shutting down
```

---

## ✅ Étape 2 — Le squelette

```bash
nano analyse_logs.py
```

```python
#!/usr/bin/env python3
"""Analyse un fichier de logs et affiche un petit resume."""

from pathlib import Path
```

**Explication** :
- Le shebang `#!/usr/bin/env python3` permet de lancer `./analyse_logs.py` directement (comme en Bash).
- Le **docstring** (`""" ... """`) documente le script — bonne habitude Python.
- L'import `from pathlib import Path` en **haut** : tout import se place en début de fichier.

---

## ✅ Étape 3 — Le corps

```python
chemin = Path("app.log")

try:
    contenu = chemin.read_text()
except FileNotFoundError:
    print("Erreur : fichier introuvable", chemin)
    exit(1)

lignes = contenu.splitlines()

erreurs = 0
for ligne in lignes:
    if "ERROR" in ligne:
        erreurs += 1

resume = {
    "errors": erreurs,
    "total": len(lignes),
}

print(f"Nombres d'erreurs : {erreurs}")
print("Resume:", resume)
```

**Explication des choix** :
- `Path("app.log")` crée un objet chemin portable (au lieu d'un simple string). `read_text()` lit tout le fichier.
- `splitlines()` découpe le contenu en une **liste de lignes** (enlève les `\n`)
- Boucle `for ligne in lignes:` + test `if "ERROR" in ligne:` compte les occurrences. (Équivalent d'un `for...of` JS.)
- Le **dictionnaire** `resume` regroupe les infos — c'est le pont vers JSON.
- Le `try/except FileNotFoundError` interrompt proprement (avec `exit(1)`) si le fichier est absent, au lieu d'un traceback.

> 💡 Variante plus « pythonique » : `erreurs = sum(1 for l in lignes if "ERROR" in l)` (compréhension de liste). La boucle explicite est plus pédagogique ici.

---

## ✅ Étape 4 — Résultat attendu

```
Nombres d'erreurs : 2
Resume: {'errors': 2, 'total': 6}
```

Pour le cas « fichier manquant » :

```
Erreur : fichier introuvable app.log
```

(et `echo $?` → `1`)

---

## ✅ Étape 5 — Réponses de l'auto-vérification

1. **Liste `[]` vs dictionnaire `{}`** : une **liste** est une suite **ordonnée** de valeurs accessibles par **index** (`l[0]`) ; un **dictionnaire** associe des **clés** à des valeurs (`d["nom"]`), non ordonné logiquement, comme un objet JS.
2. **Deux-points** : en Python, l'indentation **et les deux-points** délimitent le bloc. Sans `:`, le parseur attend un bloc qui n'arrive jamais → `SyntaxError`. C'est la syntaxe qui remplace les accolades Java/JS.
3. **`splitlines()`** : découpe une chaîne multi-lignes en **liste de lignes** (les `\n` sont retirés). Idéal pour traiter logs ou CSV.
4. **`venv`** : crée un **environnement Python isolé** pour un projet, où `pip` installe ses dépendances sans polluer le Python système ni entrer en conflit avec d'autres projets.
5. **Autre erreur (fichier non lisible)** : on capture l'exception `PermissionError` dans un `except PermissionError:` séparé, ou utilise `except (FileNotFoundError, PermissionError)`. On loggue et on `exit(1)` — jamais `except:` nu qui avalerait tout.

---

## 📝 Checklist de validation (récapitulatif + conseils)

- [ ] J'ai écrit un script Python avec shebang, docstring et imports en haut.
- [ ] Je lis un fichier avec `pathlib` (`Path(...).read_text()`).
- [ ] J'ai utilisé une **liste** (`splitlines`) et un **dictionnaire** `{}`.
- [ ] J'ai compter les `ERROR` avec une boucle `for` + `if`.
- [ ] J'ai affiché un résumé avec une **f-string**.
- [ ] J'ai géré `FileNotFoundError` avec `try/except` + `exit(1)`.
- [ ] J'ai testé le cas « fichier manquant » sans traceback.

### 💡 Conseils pour la suite

- **Re-teste-toi** dans 2-3 jours en modifiant le critère de comptage (ex. compter les `INFO`).
- **Réflexe pro** : pour un vrai script d'analyse de logs, enchaîne en **pipeline** (« commande `grep` + Python ») plutôt que tout en Python quand c'est du filtrage.
- La **Leçon 6** te montre comment **automatiser** avec Python : `subprocess`, `os`/`pathlib`, et les **appels API HTTP** avec `requests`.

---

*Prochaine étape :* Leçon 6 — **Automatisation avec Python** → dossier `06-Python-automatisation/`.
