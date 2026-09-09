# Leçon 1 — Git : les bases (init, add, commit, historique)

> **Bloc 04 — Git, Leçon 1/6.** Dans le **Bloc 03**, tu as écrit des scripts Bash et Python : un vrai outil de diagnostic. Aujourd'hui, on commence à le traiter comme le ferait un professionnel : **avec Git**, l'outil de contrôle de version utilisé par quasiment 100 % de l'industrie. Ce fil rouge (versionner ton outil de diagnostic) te suivra jusqu'à la Leçon 6.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. Expliquer **ce qu'est un système de contrôle de version** et pourquoi Git a gagné face aux sauvegardes manuelles (`projet-final.zip`, `projet-final-v2-VRAI.zip`…).
2. **Initialiser un dépôt** (`git init`) et **configurer ton identité** (`git config user.name / user.email`).
3. Expliquer et utiliser les **3 zones de Git** : répertoire de travail, index (staging), dépôt local.
4. Faire des **commits atomiques** avec des messages **lisibles** (convention *Conventional Commits*).
5. **Inspecter** l'historique et les modifications avec `git log`, `git show`, `git diff`.
6. Protéger ton dépôt avec un **`.gitignore`** dès le premier commit.

---

## 2. Explication simple

### Pourquoi Git ?

Imagine un jeu vidéo où tu ne peux sauvegarder **qu'à un seul endroit**, en **écrasant** la partie précédente. Si tu arrives bloqué à un boss, impossible de revenir en arrière. C'est exactement ce que font la plupart des débutants avec leur code : ils écrasent sans cesse les mêmes fichiers, sans filet de sécurité.

Git résout ce problème : chaque **commit** est une **sauvegarde nommée, datée et signée** de l'état complet du projet. Tu peux :

- **revenir** à n'importe quel état antérieur (fini le « ça marchait hier ! ») ;
- **voir qui a changé quoi, quand et pourquoi** ;
- **travailler à plusieurs** sans s'écraser mutuellement ;
- et en DevOps : **tracer** chaque changement de script, de configuration, de pipeline. Un `deploy.sh` modifié sans commit, c'est un changement **invisible et irréversible** — l'exact opposé de la philosophie DevOps.

> ℹ️ Au passage : SVN et CVS, deux anciens systèmes de contrôle de version **centralisés** (un seul serveur garde l'historique), se font rares. Git est **distribué** : chaque clone contient l'historique complet. Retiens juste les noms, au cas où tu les croises sur un vieux projet.

### Comment ? Les 3 zones de Git

C'est LE schéma mental à retenir. Entre ton fichier modifié et son enregistrement définitif, il passe par **trois endroits** :

```
 ┌──────────────────────┐  git add   ┌──────────────────┐  git commit  ┌──────────────┐
 │ Répertoire de travail │ ────────► │ Index (staging)  │ ───────────► │ Dépôt local  │
 │ (tes fichiers réels)  │           │ (« panier »)     │              │ (historique) │
 └──────────────────────┘           └──────────────────┘              └──────────────┘
```

- **Répertoire de travail** : tes fichiers tels que tu les vois dans l'éditeur.
- **Index (ou *staging area*)** : le « panier de courses ». Tu y mets **uniquement ce que tu veux inclure dans le prochain commit**. C'est ce qui permet des commits propres, même si tu as touché 5 fichiers.
- **Dépôt local** : l'historique, stocké dans le dossier caché `.git/` à la racine. **Ne supprime jamais ce dossier** : il contient toute la mémoire du projet.

Deux notions à connaître dès maintenant :

- **HEAD** : un pointeur qui indique « où tu es » dans l'historique (en général, le dernier commit de la branche courante).
- **Commit** : une **photo complète** du projet, avec un identifiant unique (le *hash*, ex. `a1b2c3d`), un auteur, une date et un message.

### Quand committer ?

Règle simple : **à chaque fois que tu atteins un état qui fonctionne et qui a du sens**. Un commit = **une idée, une modification cohérente** (« j'ajoute la vérification du disque »), ni « j'ai bossé 3 jours », ni « typo + refonte complète » mélangés.

---

## 3. Exemples concrets

> 🧪 Toutes les commandes ci-dessous sont **testables telles quelles**. On démarre le fil rouge : versionner ton outil de diagnostic du Bloc 03.

### 3.1 Configurer Git (une seule fois par machine)

```bash
# Ton nom et ton email apparaîtront dans chaque commit
git config --global user.name "Elgissio Franito"
git config --global user.email "elgissio@exemple.com"

# Branche par défaut : "main" (standard actuel, au lieu de "master")
git config --global init.defaultBranch main

# Vérifier ta configuration
git config --global --list
```

### 3.2 Créer le dépôt et le premier commit

```bash
# Créer et entrer dans le dossier du projet
mkdir -p ~/projets/outil-diagnostic && cd ~/projets/outil-diagnostic

# Transformer ce dossier en dépôt Git
git init
# → Initialized empty Git repository in /home/elgissio/projets/outil-diagnostic/.git/

# Le fichier le plus important du projet : ce que Git doit IGNORER
cat > .gitignore << 'EOF'
# Rapports générés par le script (ne versionne jamais des sorties)
rapport-*.txt
*.log
# Environnements virtuels Python
.venv/
__pycache__/
# Secrets locaux (clés, mots de passe)
*.env
EOF
```

```bash
# État du dépôt : qu'est-ce qui a changé ? qu'est-ce qui est en attente ?
git status
# → "Untracked files:" liste .gitignore (Git le voit mais ne le sauvegarde pas encore)

# Préparer (mettre dans le panier), puis enregistrer
git add .gitignore
git commit -m "chore: initialisation du depot avec gitignore"

git status
# → "nothing to commit, working tree clean" : tout est enregistré ✔
```

> ℹ️ En 2025-2026, la branche par défaut est **`main`**. Si tu es sur un dépôt créé avec `master`, renomme-le : `git branch -m master main`.

### 3.3 Le cycle de travail quotidien

```bash
# Créer le premier script (version simplifiée de ton outil du Bloc 03)
cat > diagnostic.sh << 'EOF'
#!/usr/bin/env bash
set -euo pipefail
# Outil de diagnostic systeme (Bloc 03, versionne au Bloc 04)
echo "=== Diagnostic systeme : $(date) ==="
echo "Disque :"
df -h /
echo "Memoire :"
free -h
EOF
chmod +x diagnostic.sh

# 1) Vérifier ce que Git voit
git status

# 2) Regarder précisément les changements (avant de mettre en panier)
git diff
# → lignes rouges (supprimées) / vertes (ajoutées) par rapport au dernier commit

# 3) Mettre en panier puis enregistrer
git add diagnostic.sh
git commit -m "feat: ajout du script de diagnostic disque et memoire"

# 4) Historique
git log --oneline
# → b3f9a21 (HEAD -> main) feat: ajout du script de diagnostic disque et memoire
# → 7d2e4c8 chore: initialisation du depot avec gitignore
```

### 3.4 Inspecter l'historique

```bash
# Historique compact
git log --oneline --graph --all

# Le détail d'un commit précis (qui, quand, quelles lignes)
git show b3f9a21

# Comparer la version actuelle à celle d'avant le dernier commit
git diff HEAD~1
```

## 4. Bonnes pratiques modernes (2025-2026)

1. **Commits atomiques** : un commit = une intention. Facilite les retours en arrière et la *code review*.
2. **Conventional Commits** : `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`, `test:`. Exemple : `feat: ajout de la verification du service nginx`.
3. **Message au présent, en impératif, avec le « pourquoi »** quand c'est utile : `fix: empecher le crash si df est absent` vaut mille fois `fix`.
4. **`.gitignore` avant tout le reste**, avec des gabarits prêts à l'emploi ([github.com/github/gitignore](https://github.com/github/gitignore)).
5. **`main` comme branche par défaut** (convention GitHub/GitLab depuis 2020).
6. **Ne commit jamais de secrets** : clés API, mots de passe, `.env`. Utilise un `.gitignore` + des variables d'environnement. Les scanners (gitleaks, truffleHog) sont standard en entreprise et sont branchés dans les pipelines CI (Bloc 11).
7. **Écris des commandes au fil de l'eau** : un commit toutes les 20-30 minutes de travail effectif vaut mieux qu'un « méga-commit » de fin de journée.

## 5. Pièges à éviter

### ❌ Piège 1 : `git add .` à l'aveugle

```bash
# MAUVAIS : tu ne sais pas ce que tu mets dans le panier
git add .
git commit -m "update"
```

Si ton dossier contient un `rapport-test.txt` oublié ou un fichier `secret.env`, il part dans l'histoire du projet **pour toujours** (le retirer demande de réécrire l'historique).

```bash
# BON : on regarde d'abord, puis on ajoute de façon ciblée
git status          # je lis ce qui a changé
git diff            # je vérifie le contenu
git add diagnostic.sh .gitignore
git diff --staged   # je vérifie ce que j'ai mis dans le panier
git commit -m "feat: ..."
```

### ❌ Piège 2 : `git commit -am` qui « mange » des fichiers

```bash
# MAUVAIS : -a commit les fichiers DÉJÀ suivis, mais JAMAIS les nouveaux
git commit -am "feat: nouvelle fonctionnalite"   # ton nouveau fichier reste hors Git !
```

```bash
# BON : add explicite, commit ensuite
git add mon-nouveau-fichier.sh
git commit -m "feat: ..."
```

### ❌ Piège 3 : messages inutiles

```bash
git commit -m "fix"              # MAUVAIS : aucun sens dans 6 mois
git commit -m "changement"       # MAUVAIS : idem
```

```bash
# BON : le message raconte ce qui a changé ET pourquoi
git commit -m "fix: ignorer le code de sortie 1 de df quand le disque est sature"
```

### ❌ Piège 4 : supprimer ou bidouiller le dossier `.git/`

Supprimer `.git/` = perdre **tout l'historique** du projet. Si Git te semble cassé, la solution n'est **jamais** de supprimer ce dossier : on apprendra à réparer proprement à la Leçon 6.

## 6. Exercice pratique

👉 Voir le fichier **`02-exercice.md`** : versionner une mini-version de ton outil de diagnostic en 4 commits atomiques, avec `.gitignore` et inspection de l'historique.

## 7. Correction de l'exercice

👉 Voir le fichier **`03-correction.md`** : correction pas à pas, explications des choix, checklist et conseils.

## 8. Checklist de validation

Coche chaque item **en le démontrant dans le terminal** :

- [ ] Je sais expliquer les 3 zones de Git et où va un fichier à chaque étape.
- [ ] Je sais initialiser un dépôt et configurer mon identité (global ET local).
- [ ] Je sais faire un commit atomique avec un message `type: description`.
- [ ] Je sais écrire et vérifier un `.gitignore` avant le premier commit.
- [ ] Je sais lire `git status`, `git diff`, `git log --oneline` et `git show <hash>`.
- [ ] Je sais corriger le message du dernier commit avec `--amend`.
- [ ] Je sais expliquer pourquoi on ne commit jamais de secrets ni de fichiers générés.

---

*Prochaine étape :* Leçon 2 — **Branches et merge** dans `02-Branches-et-merge/`, où l'on travaille sans risquer de casser `main`.

