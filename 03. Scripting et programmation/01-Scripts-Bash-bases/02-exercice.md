# Exercice pratique — Les bases du scripting Bash

> **Bloc 3 · Leçon 1** — Exercice à faire en autonomie.
> Contexte : tu es DevOps sur un serveur. Tu manipules déjà `deploy.sh production` à la main. On te demande maintenant d'écrire **ton premier vrai script réutilisable** qui prépare le déploiement d'une application, avec arguments et codes de sortie propres.

---

## 🎯 Objectif de l'exercice

Écrire un script `deploy.sh` qui prend un environnement en argument (`production`, `staging`), vérifie la présence de l'argument, affiche un message et retourne un code de sortie *correct* selon le cas.

---

## 📋 Étape 1 — Créer la structure

1. Dans un dossier `~/mon-deploy`, crée le fichier `deploy.sh`.
2. Ajoute en **première ligne** le shebang `#!/usr/bin/env bash`.
3. Ajoute un commentaire de 2 lignes décrivant le rôle du script.
4. Rends-le exécutable avec `chmod +x`.

---

## 📋 Étape 2 — Le script

Écris un script qui :

1. Déclare une variable `APP="api-demo"`.
2. Vérifie qu'un **argument** est bien passé : si aucun argument (`$#` vaut 0), affiche « Usage : ./deploy.sh production|staging » sur **stderr** et `exit 1`.
3. Stocke l'argument dans une variable `ENV`.
4. Si `ENV` vaut `production` : affiche « Deploiement de $APP en production... » et `exit 0`.
5. Sinon : affiche « Environnement inconnu : $ENV » sur stderr et `exit 1`.
6. Teste les 3 cas ci-dessous et note les sorties.

> 💡 Astuce `printf` : tu peux utiliser `printf "%s\n" "$APP"` pour un affichage propre. `>&2` redirige vers **stderr**.

---

## 📋 Étape 3 — Test manuel

Copie-colle et note le résultat de chaque commande :

```bash
./deploy.sh
./deploy.sh production
./deploy.sh staging
./deploy.sh qa
```

Pour chaque cas, note **ce qui s'affiche** et **le code de sortie**. Récupère le code de sortie immédiatement avec :

```bash
./deploy.sh ; echo "code sortie = $?"
```

---

## 📋 Étape 4 — Auto-vérification

1. Pourquoi mettre `#!/usr/bin/env bash` et pas seulement `#!/bin/bash` ?
2. Que se passe-t-il si tu oublies `chmod +x` ?
3. Pourquoi `exit 1` sur une erreur plutôt que de laisser le script se terminer « tout seul » ?
4. Quelle est la différence entre `$@` et `$#` ?

---

## 🏁 Rendu attendu

Un fichier `~/mon-deploy/deploy.sh` exécutable, et un petit tableau par écrit de tes 4 tests avec sortie + code de sortie.

> Compare ensuite avec `03-correction.md`.
