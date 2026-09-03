# Exercice pratique — Structures de contrôle et fonctions Bash

> **Bloc 3 · Leçon 2** — Exercice à faire en autonomie.
> Contexte : tu dois écrire un script utilitaire **`check-service.sh`** qui vérifie l'état d'un service systemd, et utilise **fonctions**, **conditions**, **`case`** et **boucles** pour énumérer plusieurs services. C'est exactement le genre d'outil qu'on garde en production.

---

## 🎯 Objectif de l'exercice

Écrire un script qui combine : une fonction de log, une fonction de vérification d'un service systemd, une boucle sur une liste de services, et un `case` pour gérer le compte-rendu. Aucune action destructive — tout se contente de lire l'état (`systemctl is-active`).

---

## 📋 Étape 1 — Le squelette et la fonction `log`

1. Crée `~/mon-check/check-service.sh`.
2. Ajoute le shebang `#!/usr/bin/env bash`.
3. Définis une fonction `log() { ...; }` qui affiche `[horodatage] message`. Utilise la commande `date` (tu as l'exemple dans la leçon).
4. Rends le script exécutable `chmod +x`.

---

## 📋 Étape 2 — La fonction `check_service`

Écris une fonction `check_service() {
local service="$1"
if systemctl is-active --quiet "$service"; then
log "OK : $service";      return 0
else
log "ECHEC : $service";    return 1
fi
}`

**Important** : `systemctl is-active --quiet` retourne `0` si le service est actif, non-nul sinon. N'appelle pas `systemctl status` (trop verbeux).

---

## 📋 Étape 3 — La boucle et le `case`

1. Définis une liste de services dans une variable : `SERVICES="nginx mysql ssh sshd"`.
2. Effectue une boucle `for service in $SERVICES; do check_service "$service"; done`.
3. Puis un `case "$1"` dans `main` qui appelle la boucle si l'argument vaut `all` ; sinon affiche l'usage. Pour cet exercice, on gère simplement :

   ```bash
   case "$1" in
       all)  check_all ;;
       *)  echo "Usage : $0 all" >&2; exit 1;;
   esac
   ```

4. Prends l'argument en compte : le script doit accepter `all` en argument obligatoire.

---

## 📋 Étape 4 — Test manuel

Exécute et note ce que tu obtiens :

```bash
./check-service.sh all
./check-service.sh (sans argument)
echo "code sortie = $?"
```

- `all` doit lister chaque service avec OK/ECHEC.
- Sans argument doit afficher l'usage sur **stderr** et retourner `1`.

---

## 📋 Étape 5 — Auto-vérification

1. Pourquoi utiliser `[[ ]]` de préférence à `[ ]` ?
2. Quelle différence y a-t-il entre comparer des **chaînes** (`=`) et des **entiers** (`-eq`) ?
3. Pourquoi `local service="$1"` au début d'une fonction ?
4. Que fait exactement `systemctl is-active --quiet` par rapport à `systemctl status` ?
5. Que renvoie une fonction si on oublie le `return` ?

---

## 🏁 Rendu attendu

Un script `~/mon-check/check-service.sh` exécutable, ta session de test avec `all` et sans argument, et tes réponses écrites aux 5 questions.

> Compare ensuite avec `03-correction.md`.
