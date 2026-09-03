# Leçon 1 — Les bases du scripting Bash

> **Bloc 3 · Scripting et programmation** — Leçon 1 sur 7
> Tu es déjà développeur (Java/Spring, variante NestJS) : tu sais programmer. Cette leçon te montre comment **transposer tes réflexes** (variables, arguments, codes de sortie…) vers **Bash**, le langage de scripting n°1 des serveurs Linux. À la fin, tu sauras transformer une série de commandes répétitives en script réutilisable.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Expliquer** ce qu'est un script Bash et **quand** l'utiliser plutôt que des commandes tapées à la main.
2. **Créer** un script exécutable avec le shebang `#!/usr/bin/env bash` et `chmod +x`.
3. **Utiliser** les variables, les **arguments** (`$1`, `$#`, `$@`) et leurs règles de citation.
4. **Comprendre et utiliser** les **codes de sortie** (`exit 0`, `exit 1`) et `$?`.
5. **Écrire** des sorties propres (`echo`, `printf`) et des commentaires utiles.

---

## 2. Explication simple

### 2.1 Le « pourquoi » : automatiser ce que tu répètes

Quand tu traites un serveur à la main, tu tapes les mêmes commandes encore et encore :

```bash
cd /opt/mon-app && git pull
systemctl restart mon-app
curl -s http://localhost:8080/health
```

Dès que tu reproduis cette suite **3 fois ou plus**, tu dois l'écrire **une fois** dans un script. Le script est une **recette de cuisine** : tu écris la recette une fois, et tu la rejoues à l'identique autant de fois que nécessaire, sans oublier une étape, sans rien transcrire de travers.

En DevOps, les scripts Bash servent à **embarquer une exécution** : déploiement, sauvegarde, diagnostic, préparation d'un serveur. La roadmap le résume d'une phrase : « Pourquoi je fais ça 50 fois à la main ? Je vais l'automatiser. »

### 2.2 Le « comment » : l'anatomie d'un script Bash

Un script Bash est un **simple fichier texte** dont le système sait qu'il faut le lancer avec un interpréteur Bash. Trois ingrédients obligatoires :

1. **Le shebang** (première ligne) : `#!/usr/bin/env bash` indique au système *quel programme* exécute ce fichier.
2. **Les permissions d'exécution** : `chmod +x script.sh` rend le fichier lançable en tapant `./script.sh`.
3. **Les commandes Bash**, ligne par ligne.

```bash
#!/usr/bin/env bash
# Deploy : prepare et relance l'application
cd /opt/mon-app
systemctl restart mon-app
echo "Application relancee"
```

### 2.3 Le « quand » : script Bash ou pas ?

| Situation | Automatiser ? |
|-----------|---------------|
| Suite de commandes système + redirections + pipes | ✅ **Bash** (parfait)
| Boucles/conditions légères sur des fichiers | ✅ Bash
| Logique métier complexe, JSON/API, data | 🔵 Python (leçons 5-6)
| Orchestration cloud multi-étapes | 🔵 Terraform / Ansible (blocs 8)

> 💡 **Lien avec tes acquis** : Bash fait **la colle** entre les commandes Linux. Ton expérience Java/JS te donne la logique (conditions, boucles, fonctions) ; ici tu apprends la **syntaxe shell** et ses particularités (espaces, citations).

---

## 3. Exemples concrets

### 3.1 Le premier script

```bash
#!/usr/bin/env bash
# mon-script.sh
echo "Bonjour depuis un script Bash"
```

```bash
chmod +x mon-script.sh
./mon-script.sh
# → Bonjour depuis un script Bash
```

### 3.2 Variables

```bash
#!/usr/bin/env bash
NOM="projet-api"    # PAS d'espace autour du =
PORT=8080

echo "Application: $NOM sur le port $PORT"
echo "Chemin complet: ${NOM}/logs/app.log"   # {} pour delimiter
```

### 3.3 Arguments et paramètres positionnels

```bash
#!/usr/bin/env bash
# deploy.sh production staging
echo "Le 1er argument  : $1"   # → production
echo "Le 2e argument   : $2"   # → staging
echo "Nombre d'args    : $#"   # → 2
echo "Tous les args    : $@"   # → production staging
```

### 3.4 Codes de sortie (exit codes)

```bash
#!/usr/bin/env bash
ENV="$1"

if [ "$ENV" = "production" ]; then
    echo "Deploiement en production..."
    exit 0            # 0 = succes (convention)
else
    echo "Environnement inconnu: $ENV" >&2
    exit 1            # != 0 = erreur
fi
```

Récupérer le code de sortie de la **dernière** commande :

```bash
grep "ERROR" app.log
echo "Le code de retour de grep : $?"
```

### 3.5 `echo` vs `printf`

```bash
echo "texte simple"                                       # le plus courant
printf "%s - %s\n" "2026-09-03" "Application demarree"    # format precis
```

---

## 4. Bonnes pratiques modernes (2025-2026)

- **Shebang `#!/usr/bin/env bash`** plutôt que `#!/bin/bash` : il respecte le `PATH` et fonctionne sur plus de machines (macOS, Ubuntu, distributions non standard).
- **Toujours mettre `set -euo pipefail` en tête de script** (détail en Leçon 3) : la base pour des scripts qui échouent tôt plutôt que continuer en silence.
- **Citer systématiquement les variables** : `"$1"`, `"$var"`. Un chemin sans guillemets + espace = plantage.
- **Pas d'espace autour du `=`** pour l'affectation : `PORT=8080` et non `PORT = 8080`.
- **Terminer par un `exit` explicite** et retourner `0` (succès) ou un code non nul (échec) : c'est ce que la CI/CD lira.
- **Commenter le « pourquoi »**, pas le « quoi » (`# on relance apres un pull pour charger la config` plutôt que `# relance`).
- **Valider avec `shellcheck`** : `shellcheck mon-script.sh` détecte les bugs classiques avant exécution (indispensable, standard 2025-2026).
- **`2>&1` pour fusionner erreurs** quand le script lève des erreurs utiles (détail en Leçon 2-3).

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi c'est dangereux / inefficace | ✅ Version correcte |
|----------------|----------------------------------------|---------------------|
| `NOM = "api"` (espaces autour du =) | Bash interprète `NOM` comme une commande → erreur `command not found`. | `NOM="api"` |
| `echo $PORT` sans guillemets | Si `$PORT` contient des espaces ou est vide, le comportement devient imprévisible. | `echo "$PORT"` |
| `echo "le code est $?"` après d'autres lignes | `$?` reflète la **dernière** commande, pas celle que tu crois. | Lire `$?` **immédiatement** après la commande concernée. |
| Oublier le shebang puis `./script.sh` | Permission refusée ou interprété par le mauvais shell. | Première ligne `#!/usr/bin/env bash` + `chmod +x`. |
| `rm -rf /opt/${APP}/` avec `APP` vide | Devient `rm -rf /opt/` → drame silencieux. | Garde-fou : `[ -n "$APP" ]` avant, ou `set -u`. |
| Mettre la logique métier en Bash | Script illisible et fragile dès que ça devient complexe (JSON, APIs). | Passer à **Python** (leçons 5-6). |

---

## 8. Checklist de validation

- [ ] Je peux **expliquer** ce qu'est un script Bash et donner 3 situations où l'automatiser.
- [ ] Je sais écrire un script avec `#!/usr/bin/env bash`, le rendre exécutable par `chmod +x`.
- [ ] Je sais créer des **variables** sans espace autour du `=` et les **citer** correctement.
- [ ] Je sais lire les **arguments** `$1`, `$#`, `$@` dans un script.
- [ ] Je sais utiliser **`exit 0` / `exit 1`** et récupérer `$?` au bon moment.
- [ ] Je sais choisir `echo` ou `printf` et commenter utilement.

---

> 📖 Prochaine étape : fais l'**exercice pratique** dans `02-exercice.md`, puis compare avec `03-correction.md`.
