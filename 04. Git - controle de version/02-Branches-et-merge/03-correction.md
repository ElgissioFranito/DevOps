# Correction détaillée — Leçon 2 : deux features en parallèle

## Tâche A — `feature/check-services` (fast-forward)

```bash
git status                      # On branch main, working tree clean
git switch -c feature/check-services

# Ajout du bloc services à diagnostic.sh, puis :
git add diagnostic.sh
git commit -m "feat: verification du service nginx"

git switch main
git merge feature/check-services
# → Updating <hash>..<hash>
# → Fast-forward
```

**Pourquoi fast-forward ?** Entre la création de la branche et le merge, `main` n'a reçu **aucun** commit. Git se contente de faire avancer l'étiquette `main` jusqu'au dernier commit de la feature : historique linéaire, pas de commit de fusion.

## Tâche B — `feature/check-memory` (merge 3-way)

```bash
git switch -c feature/check-memory
echo "echo 'Memoire detaillee :'" >> diagnostic.sh
echo "free -m"                    >> diagnostic.sh
git add diagnostic.sh && git commit -m "feat: memoire detaillee en Mo"

git switch main
echo "echo '=== Fin du diagnostic ==='" >> diagnostic.sh
git add diagnostic.sh && git commit -m "feat: ligne de fin de rapport"

git merge feature/check-memory
# → Merge made by the 'ort' strategy.
```

**Pourquoi un commit de fusion cette fois ?** Les deux branches ont **divergé** : chacune a un commit que l'autre n'a pas. Git ne peut plus simplement « avancer » : il crée un commit avec **deux parents** qui réconcilie les deux lignes. Pas de conflit ici car les deux commits touchent des **zones différentes** du fichier (le merge 3-way insère chaque bloc là où il a été écrit — la fin de fichier est ajoutée avant la ligne « Fin du diagnostic » ? Non : chaque commit a appendé à la fin, mais les ajouts sont des lignes distinctes que Git arrive à placer sans collision).

```bash
git log --oneline --graph --all
# → *   c4f8a21     Merge branch 'feature/check-memory'
#   |\
#   | * e2d9b34   feat: memoire detaillee en Mo
#   * | a1c3f77   feat: ligne de fin de rapport
#   |/
#   * b7e5d20     feat: verification du service nginx
#   ...
```

## Nettoyage

```bash
git branch -d feature/check-services feature/check-memory
# → Deleted branch ... (les deux étaient fusionnées, -d accepte)

git branch
# → * main
```

## Tâche C — Renommer une branche sans perdre le commit

```bash
git switch -c fix/faute-de-frappe
# ... correction du commentaire ...
git add diagnostic.sh
git commit -m "docs: correction d'un commentaire"
```

Le commit est fait sur la branche mal nommée. Solution : créer la bonne branche **au même endroit** puis supprimer la mauvaise — un commit n'appartient jamais à une branche, il est juste **pointé** par elle :

```bash
git branch docs/correction-commentaire   # nouvelle étiquette sur le MÊME commit
git switch docs/correction-commentaire
git branch -d fix/faute-de-frappe        # -d accepte : la branche est "fusionnée" au sens pointé

# Fusion dans main et nettoyage
git switch main
git merge docs/correction-commentaire    # fast-forward
git branch -d docs/correction-commentaire
```

**Vérification finale** : `./diagnostic.sh` s'exécute sans erreur, `git log --graph` montre un seul nœud de merge, `git branch` ne montre que `main`.

---

## ✅ Checklist de validation (réécrite)

- [ ] Je sais expliquer qu'une branche est une **étiquette mobile sur un commit**, et non une copie des fichiers.
- [ ] Je sais vérifier en permanence **sur quelle branche je suis** (`git status`, `git branch`) avant de committer ou merger.
- [ ] Je sais créer/basculer avec `git switch -c` et supprimer avec `git branch -d`.
- [ ] Je sais expliquer **fast-forward** (main n'a pas bougé) vs **3-way** (divergence → commit de fusion à 2 parents).
- [ ] Je fais mes merges **depuis la branche destinataire**, et je sais dans quel sens je fusionne.
- [ ] Je sais lire `git log --oneline --graph --all` et identifier un merge commit.
- [ ] Je sais « déplacer » le travail d'une branche mal nommée (étiquette au même commit) sans perdre de commit.
- [ ] Je respecte le modèle `main` + `feature/*` / `fix/*` : plus aucun chantier commité directement sur `main`.

## 🧭 Conseils

- **Réflexe** : un alias qui te fait gagner du temps — `git config --global alias.lg "log --oneline --graph --all"`, puis `git lg`. Tu l'utiliseras dans chaque leçon suivante.
- **Munition de vocabulaire** : branche, HEAD, étiquette/pointeur, fast-forward, 3-way, commit de fusion, divergence, branche zombie.
- Le prochain maillon du workflow `feature → commit → push → PR → merge` est le **remote** : Leçon 3. Et note que le réflexe « jamais rien sur main sans branche + review » prendra tout son sens à la Leçon 5 (Pull Requests) et au Bloc 11 (CI/CD déclenché sur `main`).

---

*Prochaine étape :* Leçon 3 — **Travailler avec un remote** dans `03-Travailler-avec-un-remote/`.
