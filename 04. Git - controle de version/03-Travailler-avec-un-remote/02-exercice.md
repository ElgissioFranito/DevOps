# Exercice pratique — Leçon 3 : publier et collaborer via un remote

> **Durée estimée : 45 min.** Prérequis : dépôt `outil-diagnostic` des leçons 1-2, un compte **GitHub** (gratuit). Si tu préfères GitLab, les commandes sont identiques, seule l'URL change.

## Mise en situation

Ton outil de diagnostic intéresse un « collègue ». Tu vas publier ton dépôt sur GitHub, puis simuler le travail à deux avec un **second clone** qui jouera le rôle du collègue — technique standard pour s'entraîner sans second ordinateur.

## Partie 1 — Authentification SSH (10 min)

1. Génère une clé `ed25519` si tu n'en as pas (`ls ~/.ssh/id_ed25519.pub` pour vérifier).
2. Ajoute la clé **publique** sur GitHub (Settings → SSH and GPG keys).
3. Teste : `ssh -T git@github.com`. Note le message de succès.

## Partie 2 — Publication (10 min)

4. Sur github.com, crée un dépôt **vide** `outil-diagnostic` (⚠️ sans README ni .gitignore, sinon les historiques divergeront dès le départ).
5. Depuis ton dépôt local, déclare le remote en SSH et vérifie avec `git remote -v`.
6. Pousse `main` avec le lien de suivi. Vérifie sur GitHub que les fichiers et les commits des leçons 1-2 sont là.
7. Crée et pousse aussi ta branche `feature/ajout-check-disque` (crée-la avec un petit commit si tu ne l'as plus) — à savoir faire : `git push -u origin feature/...`.

## Partie 3 — Le « collègue » (15 min)

8. Dans un autre dossier (⚠️ pas dans ton dépôt existant) : `git clone git@github.com:<ton-user>/outil-diagnostic.git collegue-diagnostic`.
9. Dans ce clone (le collègue) : crée une branche `feature/check-load`, ajoute au script un bloc `uptime` + charge moyenne, commit, push.
10. Sur GitHub, vérifie que la branche existe (menu des branches).

## Partie 4 — Le push rejeté (10 min)

11. Retourne dans TON dépôt. **Avant de fetcher**, ajoute un commit local sur `main` (par exemple `docs: note sur l'usage du script` dans le README).
12. Tente `git push` : il est **rejeté**. Lis le message : pourquoi ?
13. Intègre le travail du collègue **sans merge commit** (`git pull --rebase` — si tu ne l'as pas configuré en global : `git pull --rebase origin main`), puis pousse.
14. Affiche `git log --oneline --graph --all` : tes commits sont-ils « au-dessus » de ceux du collègue ? Y a-t-il un nœud de merge ?

## Livrables attendus

- [ ] `git remote -v` affiche `origin` en SSH (git@..., pas https://).
- [ ] `git branch -vv` montre le lien de suivi de `main`.
- [ ] Le dépôt GitHub contient le commit du collègue ET le tien.
- [ ] Tu peux expliquer, dans tes mots, la différence entre `fetch` et `pull`.
- [ ] Aucun `--force` n'a été utilisé.

## Bonus

- Crée un **fork** d'un vrai projet open source (bouton *Fork*), clone-le, puis ajoute le dépôt source comme second remote nommé `upstream` : `git remote add upstream git@github.com:<auteur-original>/<projet>.git`. Montre `git remote -v` avec les deux remotes.
