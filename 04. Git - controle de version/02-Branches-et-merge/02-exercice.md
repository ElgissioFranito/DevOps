# Exercice pratique — Leçon 2 : deux features en parallèle

> **Durée estimée : 30-40 min.** Prérequis : exercice de la Leçon 1 terminé (dépôt `diagnostic-equipe` avec 4 commits). Si tu as laissé un changement non commité dans le README (état « sale »), commit-le d'abord : `git add README.md && git commit -m "docs: description du projet"`.

## Mise en situation

Tu rejoins une mini-équipe qui maintient l'outil de diagnostic. La règle de la maison : **aucun commit direct sur `main`**. On te confie deux tâches, à faire dans l'ordre.

## Tâche A — `feature/check-services` (merge fast-forward)

1. Vérifie que tu es sur `main` et que le dépôt est propre (`git status`).
2. Crée et bascule sur la branche `feature/check-services`.
3. Ajoute à `diagnostic.sh` un bloc qui affiche l'état du service `nginx` (`systemctl is-active nginx || echo "nginx inactif"`).
4. Commit atomique (`feat:`).
5. Repasse sur `main` et fusionne. **Note ce que Git affiche** : fast-forward ou merge commit ? Pourquoi ?

## Tâche B — `feature/check-memory` (merge à 3 voies)

6. Crée la branche `feature/check-memory` et ajoute un bloc mémoire détaillée (`free -m`). Commit (`feat:`).
7. **Sans toucher à cette branche**, retourne sur `main` et ajoute-y un bloc `echo "=== Fin du diagnostic ==="` à la fin du script. Commit (`feat:`).
8. Fusionne `feature/check-memory` dans `main`. **Note la sortie** : Git crée-t-il un commit de fusion ?
9. Affiche l'historique en arbre : `git log --oneline --graph --all`. Repère le « y » du merge.
10. Supprime les deux branches de feature.

## Tâche C — Le réflexe professionnel

11. Crée la branche `fix/faute-de-frappe`, corrige un commentaire du script, commit.
12. Oh, erreur : tu aurais dû nommer la branche `docs/correction-commentaire`. **Sans perdre le commit**, crée la branche correctement nommée à partir de celle-ci, puis supprime la mauvaise (les deux doivent pointer sur le même commit).
13. Merge la branche renommée dans `main` et nettoie.

## Livrables attendus

- [ ] `main` contient les 3 fonctionnalités, le script reste exécutable (`./diagnostic.sh` ne plante pas).
- [ ] `git log --oneline --graph --all` montre **un** merge commit (la tâche B) et des lignes sinon.
- [ ] `git branch` ne liste plus que `main`.
- [ ] Tu sais dire, pour chaque merge, s'il était fast-forward et pourquoi.
