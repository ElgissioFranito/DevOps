# Exercice pratique — Leçon 1 : tes premiers commits propres

> **Durée estimée : 30-45 min.** Prérequis : Git installé (`git --version`) et les Leçons 1 à 3 du Bloc 03. Tu vas versionner une version mini de ton outil de diagnostic, comme si c'était un vrai projet professionnel.

## Mise en situation

Tu récupères le mini-script de diagnostic d'un collègue. Il l'a développé en vrac, sans Git. Ta mission : créer le dépôt **proprement** et poser les bases d'un historique lisible, comme le ferait un DevOps qui intègre un nouveau projet.

## Partie 1 — Préparation (5 min)

1. Crée un dossier `~/projets/diagnostic-equipe` et rends-en un dépôt Git.
2. Configure Git **pour ce dépôt uniquement** (pas en global) avec :
   - `user.name` : ton prénom et nom ;
   - `user.email` : une adresse email.
   > 💡 Indice : la config *locale* (au dépôt) écrase la config globale. Vérifie avec `git config user.name` une fois dans le dossier.

## Partie 2 — Le `.gitignore` d'abord (5 min)

3. Crée un `.gitignore` qui ignore :
   - les rapports générés : `rapport-*.txt` et tout `*.log` ;
   - l'environnement virtuel Python : `.venv/` ;
   - les fichiers de secrets : `*.env` ;
   - le cache Python : `__pycache__/`.
4. Fais de ce `.gitignore` le **premier commit** du dépôt, avec un message de type `chore:`.

## Partie 3 — Le script et des commits atomiques (15 min)

5. Crée `diagnostic.sh` (script Bash qui affiche la date, l'usage disque de `/` et la mémoire) et rends-le exécutable.
6. **Commit 2** : ajoute le script (`feat:`).
7. Modifie le script pour **ajouter** l'affichage de l'uptime (`uptime -p`).
8. **Commit 3** : ce changement (`feat:`).
9. Crée le fichier vide `README.md` contenant une ligne : `# Outil de diagnostic d'equipe`.
10. **Commit 4** : (`docs:`).

## Partie 4 — Inspection (10 min)

11. Affiche l'historique **compact** du dépôt.
12. Affiche le **détail du commit 3** (auteur, date, lignes modifiées).
13. Modifie le `README.md` (ajoute une 2e ligne) et affiche la **différence non commitée** avec `git diff`. Ne commit pas ce changement : laisse le dépôt dans cet état « sale » et note ce que `git status` affiche.

## Livrables attendus

- [ ] `git log --oneline` montre **exactement 4 commits**, dans l'ordre chore → feat → feat → docs.
- [ ] `git show <commit-3>` affiche bien l'ajout de la ligne uptime.
- [ ] Le `.gitignore` empêche un éventuel `rapport-test.txt` d'apparaître dans `git status` (teste-le !).
- [ ] Tu sais expliquer pourquoi le commit 2 contient **un seul** fichier.

## Bonus (si tu vas vite)

- Tu as fait une faute de frappe dans le message du commit 4 ? Corrige le **dernier** message sans créer de nouveau commit (indice : `git commit --amend`).
- Ajoute `gitleaks` à ta boîte à outils (`sudo apt install gitleaks`) et lance `gitleaks detect` dans ton dépôt : 0 secret détecté ? Parfait.
