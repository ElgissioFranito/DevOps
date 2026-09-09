# Correction détaillée — Leçon 1 : tes premiers commits propres

## Partie 1 — Préparation

```bash
mkdir -p ~/projets/diagnostic-equipe && cd ~/projets/diagnostic-equipe
git init
```

Configuration **locale** au dépôt (pas `--global`, c'était la subtilité de l'énoncé — utile quand tu as plusieurs identités : perso / pro) :

```bash
git config user.name  "Elgissio Franito"
git config user.email "elgissio@exemple.com"

git config user.name   # → Elgissio Franito (la config locale est prioritaire)
```

> 💡 **Pourquoi local ici ?** Sur un vrai projet d'équipe, tu utilises souvent l'email professionnel pour le boulot et l'email perso pour tes projets. La config locale te permet de séparer les deux sans toucher à ta config globale.

## Partie 2 — Le `.gitignore` d'abord

```bash
cat > .gitignore << 'EOF'
rapport-*.txt
*.log
.venv/
*.env
__pycache__/
EOF

git add .gitignore
git commit -m "chore: initialisation du depot avec gitignore"
```

**Pourquoi le `.gitignore` AVANT tout le reste ?** Parce que Git ignore les fichiers **dès le premier commit**… mais seulement ceux qui ne sont pas encore suivis. Si tu commit des logs puis ajoutes le `.gitignore` après, les logs restent **dans l'historique** et continuent d'être suivis. C'est une des erreurs les plus coûteuses à réparer (il faut réécrire l'historique). Le `.gitignore` se pose toujours en premier.

## Partie 3 — Commits atomiques

```bash
cat > diagnostic.sh << 'EOF'
#!/usr/bin/env bash
set -euo pipefail
# Outil de diagnostic systeme
echo "=== Diagnostic : $(date) ==="
echo "Disque :"
df -h /
echo "Memoire :"
free -h
EOF
chmod +x diagnostic.sh

git add diagnostic.sh
git commit -m "feat: ajout du script de diagnostic disque et memoire"
```

Modification du script (ajoute la ligne uptime avant la dernière ligne) :

```bash
echo "Uptime :" >> diagnostic.sh
echo "uptime -p" >> diagnostic.sh

git diff                # tu vois la/les lignes ajoutées en vert
git add diagnostic.sh
git commit -m "feat: affichage de l'uptime systeme"
```

README et 4e commit :

```bash
echo "# Outil de diagnostic d'equipe" > README.md
git add README.md
git commit -m "docs: ajout du README"
```

**Pourquoi des préfixes `feat:`, `chore:`, `docs:` ?** C'est la convention **Conventional Commits**, standard de fait en 2025-2026. Elle rend l'historique **scannable** (on repère d'un coup d'œil les fonctionnalités vs la maintenance) et permet aux outils de CI de générer des changelogs automatiquement. Tes futurs pipelines (Bloc 11) pourront s'en servir.

## Partie 4 — Inspection

```bash
git log --oneline
# → a4c7e12 (HEAD -> main) docs: ajout du README
# → 9b1f3d5 feat: affichage de l'uptime systeme
# → 2e8a6c4 feat: ajout du script de diagnostic disque et memoire
# → 5d0b9a1 chore: initialisation du depot avec gitignore
```

```bash
git show 9b1f3d5
# → author, date, puis le diff : la ligne "+uptime -p" doit apparaître
```

Changement non commité (état « sale », exactement ce qu'on attend en fin d'exercice) :

```bash
echo "Projet d'entrainement Git (Bloc 04)." >> README.md
git diff
# → diff --git a/README.md b/README.md
#   +Projet d'entrainement Git (Bloc 04).
git status
# → "Changes not staged for commit: modified: README.md"
```

**Test du `.gitignore`** :

```bash
touch rapport-test.txt
git status
# → rapport-test.txt N'APPARAIT PAS : il est ignoré ✔
rm rapport-test.txt
```

## Bonus

Corriger le message du dernier commit (le modifier, pas le répéter) :

```bash
git commit --amend -m "docs: ajout du README du projet"
```

> ⚠️ `--amend` **remplace** le dernier commit (nouveau hash). C'est parfait avant un `push`, à éviter sur des commits déjà poussés et partagés (voir Leçon 6).

---

## ✅ Checklist de validation (réécrite)

Tu peux cocher chaque ligne **en le démontrant dans le terminal**, pas de mémoire :

- [ ] Je sais expliquer les **3 zones** (répertoire de travail → index → dépôt local) et à quoi sert chacune.
- [ ] Je sais créer un dépôt (`git init`) et configurer mon identité **locale ou globale**.
- [ ] Je sais faire un commit **atomique** avec un message `type: description` (conventional commits).
- [ ] Je sais écrire un `.gitignore` **avant** le premier commit et vérifier qu'il fonctionne.
- [ ] Je sais lire `git status`, `git diff`, `git log --oneline` et `git show <hash>`.
- [ ] Je sais corriger le message du **dernier** commit avec `--amend` (et je sais quand ne pas le faire).
- [ ] Je sais expliquer pourquoi on ne commit **jamais** de secrets ni de fichiers générés.

## 🧭 Conseils

- **Réflexe terminal** : avant chaque `commit`, tape `git status` puis `git diff --staged` après le `add`. 10 secondes qui évitent 90 % des commits bâclés.
- **Munition de vocabulaire** : commit, hash, HEAD, index/staging, untracked, working tree clean, atomique — sers-t'en à voix haute.
- Ces bases seront **directement réutilisées** dans chaque leçon suivante ; au Bloc 11 (CI/CD), tes pipelines se déclencheront sur ces mêmes commits et tu utiliseras les tags de version.

---

*Prochaine étape :* Leçon 2 — **Branches et merge** dans `02-Branches-et-merge/`.
