# Exercice pratique — Leçon 5 : conflits et Pull Request

> **Durée estimée : 50-60 min.** Prérequis : leçon 3 (dépôt GitHub + clone « collègue ») et leçon 4. C'est l'exercice le plus important du bloc : c'est le workflow complet de la roadmap.

## Partie 1 — Conflit en merge (15 min)

1. Avec le clone **collègue** : branche `fix/fin-rapport`, modifie la ligne `=== Fin du diagnostic ===` du script en `--- Rapport termine ---`, commit, push.
2. Dans **ton** dépôt : branche `feature/en-tete`, modifie **la même ligne** autrement (`=== FIN ===`), commit.
3. `git fetch origin` puis `git merge origin/main` : le conflit doit surgir.
4. Résous selon le processus en 5 étapes : identifie (`git status`), comprends (lis les deux intentions), résous (garde la version du **collègue**, plus explicite), **teste** (`bash -n` puis exécution), commit.
5. Vérifie qu'aucun marqueur ne subsiste : `grep -n '<<<<<<<\|>>>>>>>' diagnostic.sh`.

## Partie 2 — Conflit en rebase (10 min)

6. Reproduis le même type de collision sur une autre ligne, entre `feature/test-conflit` (toi) et un push du collègue sur `main`.
7. Depuis ta branche : `git rebase origin/main`. Le rebase se met **en pause** sur le conflit.
8. Résous, puis **continue** le rebase avec `git rebase --continue` (et non `git commit` !). Note la différence de vocabulaire entre merge et rebase en situation de conflit.
9. Si tu préfères tout arrêter : `git rebase --abort`, et constate que rien n'a bougé.

## Partie 3 — Pull Request de bout en bout (25 min)

10. Sur ta branche `feature/en-tete` (conflit résolu, poussée), ouvre une **Pull Request** sur GitHub : base `main`, ta branche, description avec les 3 sections (Quoi / Pourquoi / Comment tester).
11. Joue le réviseur : onglet *Files changed*, ajoute **un commentaire** sur une ligne, puis approuve (*Approve*).
12. Fusionne avec *Merge pull request*, supprime la branche via l'interface, puis synchronise ton local :
    ```bash
    git switch main && git pull --rebase
    git branch -d feature/en-tete     # si encore présente localement
    ```
13. Vérifie sur GitHub que `main` contient bien ta fonctionnalité.

## Livrables attendus

- [ ] Deux conflits résolus (un en merge, un en rebase), sans marqueur résiduel et **avec test d'exécution**.
- [ ] Une PR ouverte, commentée, approuvée, fusionnée — avec une description en 3 sections.
- [ ] `git log --oneline --graph` montre le commit de fusion de la partie 1.
- [ ] Tu sais dire pourquoi on a utilisé `--continue` en rebase et pas un simple `commit`.

## Bonus

Active une **branch protection** sur `main` (GitHub : Settings → Branches → *Require a pull request before merging*). Tente un `git push` direct sur `main` : la plateforme le refuse. C'est la protection standard des équipes — et la raison pour laquelle le workflow PR n'est pas une option.
