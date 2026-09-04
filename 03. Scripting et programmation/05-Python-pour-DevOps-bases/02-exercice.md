# Exercice pratique — Introduction à Python pour DevOps

> **Bloc 3 · Leçon 5** — Exercice à faire en autonomie.
> Contexte : tu veux analyser un **fichier de logs** d'un service web et en extraire des informations. Écris ton premier script Python qui lit un fichier, compte des occurrences, manipule un petit dictionnaire JSON et gère une erreur. Aucun prérequis compliqué : tout est dans la leçon.

---

## 🎯 Objectif de l'exercice

Écrire un script Python `analyse_logs.py` qui :
- lit un fichier `app.log` défini,
- compte le nombre de lignes contenant `ERROR`,
- affiche un petit résumé via f-string,
- utilise un dictionnaire et une liste,
- gère proprement le cas où le fichier n'existe pas (`FileNotFoundError`).

---

## 📋 Étape 1 — Préparation fichier

Crée un fichier `~/mon-python/app.log` avec au moins 6 lignes dont **2 mentionnant `ERROR`** (les autres en `INFO`). Exemple :

```
INFO Server started
ERROR Database connection failed
INFO Health check OK
ERROR Timeout after 5s
INFO Server started
INFO Shutting down
```

---

## 📋 Étape 2 — Le squelette

Crée `~/mon-python/analyse_logs.py` avec :

1. le shebang `#!/usr/bin/env python3` ;
2. un docstring de 2 lignes au début (entre `"""` ) ;
3. l'import du module `pathlib` (`from pathlib import Path`).

---

## 📋 Étape 3 — Le corps

Complète ton script pour :

1. définir `chemin = Path("app.log")` ;
2. lire le contenu avec `chemin.read_text()` ;
3. découper en **lignes** : `contenu.splitlines()` ;
4. **compter** les lignes contenant `"ERROR"` avec une boucle `for` (ou une compréhension de liste) ;
5. afficher `f"Nombres d'erreurs : {compte}"` ;
6. mettre le compte dans un **dictionnaire** `resume = {"errors": compte, "total": len(lignes)}` et l'afficher ;
7. envelopper la lecture dans un `try/except FileNotFoundError` qui affiche un message clair.

---

## 📋 Étape 4 — Exécution

```bash
cd ~/mon-python
python3 analyse_logs.py
```

Attendu : un résumé avec `erreurs`, `total`, et aucun plantage.

Teste aussi le cas « fichier manquant » en renommant `app.log` temporairement :

```bash
mv app.log app.log.bak
python3 analyse_logs.py     # doit afficher le message d'erreur propre
mv app.log.bak app.log
```

---

## 📋 Étape 5 — Auto-vérification

1. Quelle est la différence entre une **liste** `[ ]` et un **dictionnaire** `{ }` en Python ?
2. Pourquoi faut-il des **deux-points** après `if` et `for` ?
3. Que fait `splitlines()` ?
4. À quoi sert un `venv` ?
5. Comment gérerait-on une erreur autre que `FileNotFoundError` (ex. fichier non lisible) ?

---

## 🏁 Rendu attendu

Un script `analyse_logs.py` fonctionnel qui affiche un résumé, gère le fichier manquant, et tes 5 réponses écrites.

> Compare ensuite avec `03-correction.md`.
