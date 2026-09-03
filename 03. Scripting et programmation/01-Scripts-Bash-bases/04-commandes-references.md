# Aide-mémoire — Les bases du scripting Bash

> **Bloc 3 · Leçon 1** — Fiche de référence pour écrire tes premiers scripts.

## 📌 Le squelette d'un script

```bash
#!/usr/bin/env bash
set -euo pipefail   # (détaille en Lecon 3, a adopter des maintenant)

# Variables
NOM="api"

# Code...
exit 0
```

## 📌 Variables et arguments

| Syntaxe | Effet |
|---------|-------|
| `NOM="api"` | Affectation (sans espace autour du `=`)
| `$NOM` / `${NOM}` | Lecture d'une variable (`{}` utile si suivie de caractères)
| `"$NOM"` | Lecture en citant (protège les espaces)
| `$1`, `$2`... | 1er, 2e argument positionnel
| `$#` | Nombre d'arguments
| `$@` | Tous les arguments (liste)
| `$0` | Nom du script lui-même
| `$?` | Code de sortie de la *dernière* commande

## 📌 Codes de sortie

| Instruction | Effet |
|-------------|-------|
| `exit 0` | Succès (convention)
| `exit 1` | Erreur générique
| `exit 2` | Erreur d'usage (argument invalide) — convention
| `exit 127` | Commande introuvable (généré par le shell)
| `echo "..." >&2` | Écrit sur **stderr** (parent)

## 📌 Affichage

| Commande | Effet |
|----------|-------|
| `echo "texte"` | Affiche + retour à la ligne
| `printf "%s\n" "val"` | Formatage précis (le plus fiable)
| `echo "..." >&2` | Affichage sur stderr

## 📌 Création / exécution

```bash
chmod +x mon-script.sh   # rend exécutable
./mon-script.sh          # execute
bash mon-script.sh       # execute sans droits d'exec (depannage)
```

## 📌 Check pratique

```bash
# 1. presence d'argument
if [ "$#" -eq 0 ]; then
    echo "Usage : $0 mon-argument" >&2
    exit 1
fi

# 2. tester une valeur
if [ "$1" = "production" ]; then
    echo "production"
fi
```
