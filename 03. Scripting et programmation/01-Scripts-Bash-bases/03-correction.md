# Correction détaillée — Les bases du scripting Bash

> **Bloc 3 · Leçon 1** — Correction pas-à-pas de `02-exercice.md`. Suis chaque ligne et compare à ton script.

---

## ✅ Étape 1 — Créer la structure

```bash
mkdir -p ~/mon-deploy
cd ~/mon-deploy
nano deploy.sh
```

Contenu initial, **trois lignes obligatoires** :

```bash
#!/usr/bin/env bash
# deploy.sh — Prepare le deploiement de l'application api-demo
# Usage : ./deploy.sh production|staging
```

Rendre exécutable :

```bash
chmod +x deploy.sh
```

**Explication des choix** :
- Le **shebang** `#!/usr/bin/env bash` indique *quel interpréteur* exécute le fichier. `env` cherche `bash` dans le `PATH`, ce qui est plus portable.
- `chmod +x` rend le fichier lançable directement (`./deploy.sh`). Sans lui, tu obtiens `Permission denied` même si ton code est parfait.

---

## ✅ Étape 2 — Le script complet

```bash
#!/usr/bin/env bash
# deploy.sh — Prepare le deploiement de l'application api-demo
# Usage : ./deploy.sh production|staging

APP="api-demo"

# 1. Verifie qu'un argument est present
if [ "$#" -eq 0 ]; then
    echo "Usage : ./deploy.sh production|staging" >&2
    exit 1
fi

ENV="$1"

# 2. Traitement principal
if [ "$ENV" = "production" ]; then
    printf "Deploiement de %s en production...\n" "$APP"
    exit 0
else
    echo "Environnement inconnu : $ENV" >&2
    exit 1
fi
```

**Explication des choix techniques** :
- `[ "$#" -eq 0 ]` : `$#` est le **nombre d'arguments**. S'il vaut 0, on s'arrête.
- `>&2` : la **redirection de stderr**. Un message d'erreur doit partir sur stderr, pas stdout, pour que le script soit exploitable par un autre programme (un pipe ne récupérera pas l'erreur, un logger oui).
- Les **guillemets partout** (`"$#"`, `"$ENV"`, `"$APP"`) : protège contre les espaces et valeurs vides.
- `exit 0` en succès, `exit 1` en erreur : convention universelle, lue aussi par la CI/CD.

> ⚠️ **Piège vu dans l'énoncé** : ne fais pas `if [ $ENV = production ]` sans guillemets — si `$ENV` est vide, la ligne devient `[ = production ]` et Bash explose en erreur de syntaxe.

---

## ✅ Étape 3 — Résultats des tests

| Commande | Sortie | Code de sortie |
|----------|--------|----------------|
| `./deploy.sh` | `Usage : ./deploy.sh production|staging` (stderr) | `1` |
| `./deploy.sh production` | `Deploiement de api-demo en production...` | `0` |
| `./deploy.sh staging` | `Environnement inconnu : staging` (stderr) | `1` |
| `./deploy.sh qa` | `Environnement inconnu : qa` (stderr) | `1` |

Pour vérifier le code de sortie d'un test :

```bash
./deploy.sh ; echo "code sortie = $?"
```

> 💡 `staging` passe dans `else` car seuls `production` est accepté — c'est voulu : on *fail-fast* plutôt que de déployer vers un environnement non reconnu.

---

## ✅ Étape 4 — Réponses de l'auto-vérification

1. **`#!/usr/bin/env bash` vs `#!/bin/bash`** : `env` résout `bash` depuis le `PATH` → plus portable (macOS, Linux sans bash dans `/bin`).
2. **Sans `chmod +x`** : `./deploy.sh` renvoie `Permission denied`. On peut encore lancer `bash deploy.sh`, mais le réflexe pro est de le rendre exécutable.
3. **`exit 1` sur erreur** : sans lui le script se termine avec le code de la *dernière* commande, qui peut être `0` même en cas d'échec → le pipeline/CI croirait que tout va bien. Un `exit` explicite est une **contracte** clair sur la réussite du script.
4. **`$@` vs `$#`** : `$@` = **tous les arguments** (la liste), `$#` = **le nombre d'arguments**.

---

## 📝 Checklist de validation (récapitulatif + conseils)

- [ ] J'ai écrit un script avec `#!/usr/bin/env bash` + `chmod +x` qui s'exécute avec `./deploy.sh`.
- [ ] J'utilise des **variables** (`APP`, `ENV`) sans espace autour du `=` et **citées** partout.
- [ ] Je lis **`$#`**, **`$1`** et je gère le cas « aucun argument ».
- [ ] Je distingue **stdout** (succès) et **stderr** (`>&2` pour les erreurs).
- [ ] Je retourne **`exit 0` / `exit 1`** selon le résultat.
- [ ] Je peux récupérer `$?` **immédiatement** après une commande.

### 💡 Conseils pour la suite

- **Re-teste-toi** dans 2-3 jours sans regarder la correction.
- **Réflexe pro** : dès qu'un script reçoit des arguments d'environnement, *fail-fast* au début (vérifier `$#`, valeurs autorisées) plutôt que continuer puis planter au milieu.
- La **Leçon 2** ajoute les **structures de contrôle** (`if`, `for`, `while`) et les **fonctions** — le socle pour des scripts plus intelligents.

---

*Prochaine étape :* Leçon 2 — **Structures de contrôle et fonctions Bash** → dossier `02-Bash-structures-de-controle/`.
