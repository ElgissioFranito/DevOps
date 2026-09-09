# Correction détaillée — Leçon 3 : publier et collaborer via un remote

## Partie 1 — SSH

```bash
ls ~/.ssh/id_ed25519.pub          # si absent :
ssh-keygen -t ed25519 -C "elgissio@exemple.com"

cat ~/.ssh/id_ed25519.pub         # à copier dans GitHub → Settings → SSH keys
ssh -T git@github.com
# → Hi <ton-user>! You've successfully authenticated, but GitHub does not provide shell access.
```

Le message « does not provide shell access » est **normal** : on ne se connecte pas en terminal, seulement pour Git. Si l'authentification échoue, vérifie que tu as copié le contenu **complet** du `.pub` (une seule ligne `ssh-ed25519 AAAA... commentaire`).

## Partie 2 — Publication

⚠️ Le piège du dépôt GitHub avec README : si tu avais coché « Add a README », GitHub crée un commit que ton local n'a pas → premier `push` rejeté. Deux écoles : créer le dépôt **vide** (choix de l'énoncé, le plus simple), ou tirer d'abord (`git pull --rebase origin main`) avant de pousser.

```bash
cd ~/projets/outil-diagnostic
git remote add origin git@github.com:ElgissioFranito/outil-diagnostic.git
git remote -v                     # les 2 lignes (fetch/push) doivent être en git@github.com
git push -u origin main
```

**Pourquoi `-u` ?** Il crée la branche de suivi : ta `main` locale « suit » `origin/main`. Ensuite `git status` te dira `Your branch is ahead of 'origin/main' by 1 commit` — une boussole précieuse. `git branch -vv` doit afficher :

```
* main a1b2c3d [origin/main] docs: note sur l'usage du script
```

Branche de feature :

```bash
git switch -c feature/ajout-check-disque
echo "echo 'Disque /home :'" >> diagnostic.sh
echo "df -h /home"            >> diagnostic.sh
git add diagnostic.sh && git commit -m "feat: verification du disque /home"
git push -u origin feature/ajout-check-disque
```

## Partie 3 — Le « collègue »

```bash
cd ~/projets                       # dossier NEUTRE : jamais cloner dans un dépôt existant
git clone git@github.com:ElgissioFranito/outil-diagnostic.git collegue-diagnostic
cd collegue-diagnostic

git switch -c feature/check-load
echo "echo 'Charge systeme :'" >> diagnostic.sh
echo "uptime"                  >> diagnostic.sh
git add diagnostic.sh && git commit -m "feat: affichage de la charge systeme"
git push -u origin feature/check-load      # le collègue pousse SA branche
```

Le clone a déjà `origin` configuré et les branches en suivi — c'est l'intérêt de `clone` par rapport à `init` + `remote add`.

## Partie 4 — Le push rejeté

```bash
cd ~/projets/outil-diagnostic
echo "Usage : ./diagnostic.sh [nom-de-machine]" >> README.md
git add README.md && git commit -m "docs: note sur l'usage du script"

git push
# → ! [rejected] main -> main (fetch first)
# → hint: Updates were rejected because the remote contains work that you do not have locally.
```

**Pourquoi rejeté ?** Le remote contient un commit que tu n'as pas (le push du collègue a fait avancer `origin/main`). Git refuse d'écraser un historique que tu n'as pas vu — c'est la protection *non-fast-forward*.

```bash
git fetch origin
git log HEAD..origin/main --oneline
# → f3e8d21 feat: affichage de la charge systeme    (le commit du collègue, visible SANS toucher à ton travail)

git pull --rebase origin main
git push
git log --oneline --graph --all
```

**Résultat attendu** : pas de nœud de merge. `--rebase` a **rejoué** ton commit `docs:` **par-dessus** celui du collègue — historique linéaire et lisible. C'est exactement ce qu'on approfondit à la Leçon 4.

---

## ✅ Checklist de validation (réécrite)

- [ ] Je sais générer une clé SSH `ed25519`, la déclarer sur GitHub et tester avec `ssh -T`.
- [ ] Je sais publier un dépôt local (`remote add origin` + `push -u`) et vérifier les liens de suivi (`branch -vv`).
- [ ] Je sais cloner un projet dans un dossier **neutre** et savoir ce que `clone` configure automatiquement.
- [ ] Je sais expliquer **fetch** (télécharge, ne touche à rien) vs **pull** (fetch + fusion).
- [ ] Je sais inspecter les commits distants avec `git log HEAD..origin/main --oneline` **avant** de fusionner.
- [ ] Je sais réagir à un push rejeté : fetch → intégrer (`pull --rebase`) → push, sans jamais `--force`.
- [ ] Je sais pousser une branche de feature et créer un second remote (`upstream`) sur un fork.

## 🧭 Conseils

- **Réflexe quotidien** : `git fetch` en arrivant, `git status` (il te dira si tu es en avance/en retard), puis `git pull --rebase` si besoin. Tu ne seras plus jamais surpris par un push rejeté.
- **Munition de vocabulaire** : remote, origin, upstream, tracking branch, non-fast-forward, force-with-lease, fork.
- Ton dépôt est maintenant **sur GitHub avec une branche de feature poussée** : c'est le terrain parfait pour la Leçon 5 (Pull Request, review, conflits) — la plateforme transforme ta branche poussée en « demande d'intégration ».

---

*Prochaine étape :* Leçon 4 — **Rebase, stash et historique propre** dans `04-Rebase-stash-et-historique-propre/`.
