# Exercice pratique — Leçon 4 : rebase, stash et commits propres

> **Durée estimée : 40-50 min.** Prérequis : dépôt `outil-diagnostic` publié sur GitHub (Leçon 3). Cet exercice manipule l'historique : travaille sur un clone d'entraînement si tu veux être tranquille (`git clone git@github.com:.../outil-diagnostic.git exercice-rebase`).

## Mise en situation

C'est vendredi. Tu développes la vérification de charge système, quand deux urgences tombent. Objectif : gérer tout ça **sans commit bâclé et sans historique pollué**.

## Partie 1 — Stash : l'urgence (10 min)

1. Sur une branche `feature/check-load`, commence un bloc « charge système » dans `diagnostic.sh` (2-3 lignes, incomplètes). Ne commit pas.
2. Vérifie que `git status` montre des modifications.
3. Une urgence : le `.gitignore` oublie `*.tmp`. **Mets ton travail en pause** (stash **nommé**), corrige et commite le `.gitignore` sur `main`.
4. Reviens sur ta feature et **récupère** ton brouillon. Vérifie son contenu.

## Partie 2 — Divergence puis rebase (15 min)

5. Termine le bloc charge système (ajoute `uptime` en dernière ligne), commit : `feat: affichage de la charge systeme`.
6. **Sans toucher à la feature**, va sur `main` et commite un changement ailleurs (par ex. une ligne dans `README.md` : `docs: section installation`). Les branches ont maintenant **divergé**.
7. Reviens sur `feature/check-load` et **rebase** sur `main`. Observe la sortie de Git.
8. Affiche `git log --oneline --graph --all` : l'historique est-il linéaire ? Y a-t-il un commit de fusion ?
9. Ta branche était poussée sur GitHub (Leçon 3) : pousse la version rebasee **correctement** (quel flag ? pourquoi celui-là et pas l'autre ?).

## Partie 3 — Nettoyage avant PR (15 min)

10. Crée une branche `feature/check-disk-usage` et fais **3 commits volontairement sales** sur le même sujet :
    - `feat: check disque` (ajoute `df -h /` au script),
    - `fix: typo` (corrige un commentaire),
    - `wip` (ajoute un commentaire `# TODO seuil d'alerte`).
11. **Fusionne les trois en un seul commit** propre : `feat: verification de l'usage disque avec TODO seuil` (rebase interactif, squash).
12. Vérifie avec `git log --oneline main..HEAD` qu'il ne reste qu'un commit.
13. Pousse la branche.

## Livrables attendus

- [ ] `git stash list` est vide à la fin, et le brouillon a bien été retrouvé puis intégré.
- [ ] L'historique de `feature/check-load` est **linéaire** (aucun nœud de merge avec `main`).
- [ ] Le push de la partie 2 a utilisé `--force-with-lease` (et tu sais justifier).
- [ ] `main..feature/check-disk-usage` contient **un seul** commit au message propre.
- [ ] Aucun rebase sur `main` ni sur une branche d'autrui.

## Bonus

Simule un rebase bloqué : refais diverger deux branches sur **la même ligne** du script, tente `git rebase main` — Git se met en pause avec un conflit (avant-goût de la Leçon 5). Utilise la porte de sortie : `git rebase --abort`, et vérifie que rien n'a changé (`git log --oneline`).
