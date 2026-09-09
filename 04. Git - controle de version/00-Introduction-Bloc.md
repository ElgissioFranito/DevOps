# Introduction au Bloc 4 — Git : contrôle de version

> **À lire en premier**, avant la Leçon 1. Ce fichier te dit :
> - de quoi parle ce bloc et **pourquoi il est la compétence n° 1 attendue** d'un DevOps,
> - ce qu'il te faut **préparer** avant de commencer (Git + un compte GitHub),
> - les **6 leçons** du bloc et le **fil rouge** qui les relie,
> - le vocabulaire que tu vas croiser.

---

## 🎯 De quoi parle ce bloc ?

Dans le **Bloc 03**, tu as écrit un outil de diagnostic en Bash et Python. Mais un code qui n'est pas **versionné** est un code fragile : pas d'historique, pas de retour en arrière, pas de collaboration, pas de traçabilité. Ce bloc te donne le socle de **tout le travail en équipe** : **Git**, le système de contrôle de version utilisé par quasiment 100 % de l'industrie, et son écosystème (GitHub/GitLab, branches, Pull Requests).

Objectif de la roadmap : *« une équipe te donne un repository et tu peux travailler dessus proprement, sans casser l'historique ou avoir peur des conflits »*. C'est un critère **bloquant** : les blocs suivants (CI/CD, GitOps) supposent que tes commits et tes branches sont maîtrisés.

> 💡 **Bloc très concret** : chaque leçon se pratique dans le terminal, et les leçons 3 à 6 sur un vrai dépôt GitHub. Prépare ton compte GitHub dès maintenant.

---

## ✅ Prérequis et préparation

- **Les Blocs 01-03** : le schéma « du Git à la production » (Bloc 01) et des scripts à versionner (Bloc 03 — ils servent de fil rouge).
- **Git** : `git --version`. Sous Ubuntu : `sudo apt install git`.
- **Configuration initiale** (à faire une fois, Leçon 1) :
  ```bash
  git config --global user.name "Ton Nom"
  git config --global user.email "ton@email.com"
  git config --global init.defaultBranch main
  ```
- **Un compte GitHub** (gratuit) et une **clé SSH** configurée (géré à la Leçon 3).
- Un éditeur de texte (VS Code, nano…) — aucune autre installation.

---

## 🗺️ Les 6 leçons du bloc (et le fil rouge)

Le fil rouge : *« versionner et faire évoluer l'outil de diagnostic du Bloc 03 comme le ferait une équipe professionnelle »*.

| # | Leçon | Compétence |
|---|-------|------------|
| 1 | Bases : commit et historique | `init`, 3 zones, `add`/`commit`, `log`/`diff`, `.gitignore` |
| 2 | Branches et merge | `branch`/`switch`, fast-forward vs 3-way, modèle `main + feature/*` |
| 3 | Travailler avec un remote | SSH, `clone`, `push`, `fetch` vs `pull`, push rejeté |
| 4 | Rebase, stash et historique propre | `rebase`, règle d'or, `stash`, `pull --rebase`, squash avant PR |
| 5 | Conflits et Pull Requests | Résoudre un conflit (5 étapes), PR/MR, review, GitHub vs GitLab |
| 6 | Réparation et projet récapitulatif | `reset`/`revert`/`reflog`, workflow complet de bout en bout |

Chaque dossier contient 3 fichiers : `01-lecon.md`, `02-exercice.md`, `03-correction.md`.

---

## 🧠 Vocabulaire que tu vas croiser

| Terme | C'est quoi ? (1 phrase) | Tu l'apprendras vraiment |
|-------|--------------------------|--------------------------|
| **Commit** | Une photo nommée et signée de l'état du projet | Leçon 1 |
| **Index / staging** | Le « panier » qui décide de ce que contiendra le prochain commit | Leçon 1 |
| **HEAD** | Le pointeur « où suis-je » dans l'historique | Leçons 1-2 |
| **Branche** | Une étiquette mobile sur un commit (pas une copie !) | Leçon 2 |
| **Remote / origin** | La copie de l'historique hébergée sur GitHub/GitLab | Leçon 3 |
| **fetch vs pull** | Télécharger sans fusionner / télécharger et fusionner | Leçon 3 |
| **Rebase** | Rejouer mes commits sur une nouvelle base (historique linéaire) | Leçon 4 |
| **stash** | Mettre un travail en cours de côté sans le committer | Leçon 4 |
| **Conflit** | Deux modifications concurrentes sur les mêmes lignes | Leçon 5 |
| **PR / MR** | « Vérifiez mon code avant de l'intégrer » (GitHub / GitLab) | Leçon 5 |
| **revert / reset / reflog** | Les trois outils d'annulation et la boîte noire de Git | Leçon 6 |
| **SVN / CVS** | Anciens systèmes **centralisés** — à savoir nommer, pas maîtriser | Leçons 1 et 5 |

---

## ✅ Bloc acquis si

Tu peux, **de mémoire** :

- faire le cycle `add → commit` avec des messages lisibles (conventional commits) ;
- créer une branche, la synchroniser (rebase) et la fusionner sans peur ;
- pousser, fetcher, et réagir à un push rejeté sans `--force` ;
- résoudre un conflit en suivant **identifier → comprendre → résoudre → tester → commit** ;
- ouvrir et faire merger une Pull Request proprement ;
- réparer une mauvaise manipulation avec `reset`/`revert`/`reflog`.

Si tu coches tout, la suite logique de la roadmap t'attend : **Bloc 05 — Réseautage et sécurité**, et tes commits alimenteront bientôt les pipelines du **Bloc 11 (CI/CD)** et du **Bloc 13 (GitOps)**.

---

*Démarre maintenant avec la **Leçon 1** dans `01-Git-bases-commit-et-historique/`.*
