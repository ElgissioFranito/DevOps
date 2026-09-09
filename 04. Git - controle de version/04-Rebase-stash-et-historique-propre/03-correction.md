# Correction détaillée — Leçon 4 : rebase, stash et commits propres

## Partie 1 — Stash

```bash
git switch -c feature/check-load
echo "echo 'Charge systeme :'" >> diagnostic.sh      # brouillon incomplet, volontairement

git stash push -m "brouillon check-load en cours"
git status          # → nothing to commit, working tree clean ✔

git switch main
echo "*.tmp" >> .gitignore
git add .gitignore && git commit -m "chore: ignorer les fichiers temporaires"
git switch feature/check-load

git stash pop
cat diagnostic.sh   # le brouillon est revenu
git stash list      # vide : le stash a été consommé par pop ✔
```

**Choix technique** : `pop` (récupère + supprime) plutôt que `apply` (récupère sans supprimer). Si tu veux garder le stash en réserve après récupération (rare), c'est `apply`. Et le `-m` nommé : sans lui, `stash@{0}` ne dit rien dans trois jours.

## Partie 2 — Divergence puis rebase

```bash
echo "uptime" >> diagnostic.sh
git add diagnostic.sh && git commit -m "feat: affichage de la charge systeme"

git switch main
echo "## Installation" >> README.md
echo "git clone git@github.com:..." >> README.md
git add README.md && git commit -m "docs: section installation"

git switch feature/check-load
git rebase main
# → Successfully rebased and updated refs/heads/feature/check-load.

git log --oneline --graph --all
# → PAS de nœud de merge : docs: ... puis feat: ..., en ligne droite ✔
```

**Ce qui s'est passé** : Git a copié tes commits de feature et les a rejoués au sommet de `main`. Tes commits ont **changé de hash** (ce sont des copies) — d'où l'étape suivante :

```bash
git push --force-with-lease origin feature/check-load
```

**Pourquoi `--force-with-lease` et pas `--force` ?** La branche distante contient les **anciens** commits (avant copie) : un push normal est rejeté (non-fast-forward). Il faut remplacer l'historique distant — mais uniquement si **personne d'autre n'a poussé entre-temps**, ce que `--force-with-lease` vérifie. Sur une branche d'équipe, même ce push serait interdit : on rebase avant le premier push, ou on merge.

## Partie 3 — Nettoyage avant PR

```bash
git switch -c feature/check-disk-usage
echo "echo 'Usage disque :'" >> diagnostic.sh
echo "df -h /"                >> diagnostic.sh
git add diagnostic.sh && git commit -m "feat: check disque"
# ... 2 autres commits (fix: typo / wip) ...

git log --oneline main..HEAD
# → 8f7e6d5 (HEAD -> feature/check-disk-usage) wip
# → 7e6d5c4 fix: typo
# → 6d5c4b3 feat: check disque

git rebase -i main
```

Dans l'éditeur, on **garde** le premier en `pick` et on écrase les deux autres dans lui :

```
pick   6d5c4b3 feat: check disque
squash 7e6d5c4 fix: typo
squash 8f7e6d5 wip
```

Git demande le message final ; on écrit :

```
feat: verification de l'usage disque avec TODO seuil d'alerte
```

```bash
git log --oneline main..HEAD
# → a9b8c7d (HEAD -> feature/check-disk-usage) feat: verification de l'usage disque avec TODO seuil d'alerte
git push -u origin feature/check-disk-usage
```

**Pourquoi squash avant PR ?** Le réviseur lit **une** modification cohérente au lieu de trois allers-retours. C'est le standard « PR = une intention » des équipes modernes (souvent automatisé via *squash merge* côté plateforme).

## Bonus — la porte de sortie

Après le conflit pendant `git rebase main` (Git en pause, `git status` affiche *interactive rebase in progress*), `git rebase --abort` annule tout et te rend l'état d'avant, au commit près. La résolution du conflit elle-même est la Leçon 5.

---

## ✅ Checklist de validation (réécrite)

- [ ] Je sais mettre un travail en cours de côté avec `git stash push -m "..."` et le retrouver avec `git stash pop`.
- [ ] Je sais expliquer le rebase : copie + rejeu de mes commits sur une nouvelle base, avec **nouveaux hash**.
- [ ] Je connais la règle d'or : rebase uniquement **mes** commits non partagés (ou ma feature dont je suis seul auteur).
- [ ] Je sais synchroniser une feature avec `main` par rebase et obtenir un historique **linéaire** (`log --graph` sans nœud de merge).
- [ ] Je sais pousser une branche rebasee avec `--force-with-lease` et justifier pourquoi pas `--force`.
- [ ] Je sais regrouper plusieurs commits en un avec `git rebase -i` + `squash` avant une PR.
- [ ] Je connais `git rebase --abort` comme porte de sortie et j'ai vérifié qu'il ne casse rien.

## 🧭 Conseils

- **Réflexe avant chaque PR** : `git fetch && git rebase origin/main` puis `rebase -i` pour squash — ta PR sera fusionnée d'un coup, sans conflit surprise.
- **Munition de vocabulaire** : rebase, rejouer, hash, squash, reword, stash, pop/apply, force-with-lease, historique linéaire.
- **Attention** : un rebase est aussi la première source de « j'ai perdu mes commits ! » chez les débutants — en réalité rien n'est perdu, et la Leçon 6 (`reflog`) te montrera comment tout retrouver.

---

*Prochaine étape :* Leçon 5 — **Conflits et Pull Requests** dans `05-Conflits-et-pull-requests/`.
