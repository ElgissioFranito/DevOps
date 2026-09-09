# Exercice pratique — Leçon 6 : réparer, puis le projet récapitulatif

> **Durée estimée : 60-75 min.** Prérequis : dépôt `outil-diagnostic` sur GitHub (leçons 3-5). **Important** : fais cet exercice sur un clone d'entraînement (`git clone git@github.com:<user>/outil-diagnostic.git exercice-reparation`) — on va volontairement casser des choses.

## Partie 1 — Les 3 scénarios de crash (25 min)

### Scénario A : le commit sur la mauvaise branche

1. Sur `main`, commite **par erreur** un début de fonctionnalité : ajoute un commentaire `# TODO charge reseau` dans le script, commit `feat: debut check reseau`.
2. Répare **sans perdre le commit** : il doit finir sur une nouvelle branche `feature/check-reseau`, et `main` doit revenir à son état d'avant. (Deux commandes suffisent — Leçon 2, piège 1.)

### Scénario B : le reset de trop

3. Sur `feature/check-reseau`, fais 2 commits (le bloc de charge réseau complet : `uptime` + moyenne de charge).
4. Puis commets l'erreur : `git reset --hard HEAD~2`. Tes 2 commits ont disparu de la branche.
5. **Résous avec `git reflog`** : retrouve les hash, restaure la branche exactement comme avant le reset, vérifie avec `git log --oneline`.

### Scénario C : le commit partagé à annuler

6. Pousse `feature/check-reseau`, puis commite un « bug » volontaire : une ligne qui fait échouer le script (ex. `exit 1` en plein milieu), commit, push.
7. **Annule proprement ce commit poussé** (sans réécrire l'histoire), pousse, et vérifie que le script refonctionne (`./diagnostic.sh`).

## Partie 2 — Projet récapitulatif : le workflow complet (35 min)

Mission : ajouter la **vérification réseau** (ping d'une machine) à l'outil de diagnostic, en appliquant **tout le bloc**. À enchaîner seul, dans l'ordre :

1. **Branche** `feature/check-reseau-final` créée depuis `main` à jour (fetch + pull --rebase d'abord).
2. **Commits atomiques** (minimum 3, conventionnels) : ajout du bloc ping, puis du `.gitignore`/README si pertinent, puis correction d'un détail.
3. **Nettoyage** : regroupe ou renomme si nécessaire avant l'ouverture de la PR (`rebase -i`).
4. **Push** de la branche.
5. **Pull Request** avec la description en 3 sections (Quoi / Pourquoi / Comment tester).
6. **Self-review** : commente une ligne, approuve, puis **merge**.
7. **Resynchronisation** locale (`main` + suppression de la branche locale).
8. **Vérification finale** : `git log --oneline --graph -10` propre, script exécutable.

## Livrables attendus

- [ ] Scénario A : `main` n'a plus le commit erroné, la branche `feature/check-reseau` l'a.
- [ ] Scénario B : les 2 commits sont restaurés (même hash qu'avant le reset visible dans le reflog).
- [ ] Scénario C : l'historique contient le commit bug **et** son `Revert`, le script tourne.
- [ ] Partie 2 : une PR fusionnée, description complète, `main` à jour, `git branch` sans branche résiduelle.
- [ ] Tu peux raconter **chaque scénario de mémoire** (quelle commande, pourquoi celle-là).

## Bonus

Crée un fichier `REPARATIONS.md` dans le dépôt (via une PR, évidemment) documentant les 3 scénarios : symptôme → commande de réparation → pourquoi. Ce fichier devient ta « fiche de secours » Git.
