# Leçon 3 — Travailler avec un remote (clone, push, pull, fetch)

> **Bloc 04 — Git, Leçon 3/6.** Jusqu'ici ton dépôt ne vivait que sur ta machine : si ton disque dur meurt, tout disparaît. Pire : impossible de collaborer. Cette leçon connecte ton dépôt **local** à un dépôt **distant** (*remote*) sur GitHub ou GitLab — le cœur du workflow `commit → push → PR → merge` de la roadmap.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. Expliquer la différence entre dépôt **local** et dépôt **distant** (*remote*), et le rôle d'`origin`.
2. Publier un dépôt local sur GitHub avec `git remote add` + `git push -u`.
3. Récupérer un projet existant avec `git clone`.
4. Distinguer `git fetch` (télécharger sans fusionner) et `git pull` (télécharger **et** fusionner) — et savoir quand utiliser l'un ou l'autre.
5. Configurer l'authentification **SSH** (la norme professionnelle, plutôt que HTTPS + token).
6. Comprendre les branches de suivi (*tracking branches*) et le sens de `git push -u`.

---

## 2. Explication simple

### Pourquoi un remote ?

Ton dépôt local est comme le disque dur de ta console de jeu : tes sauvegardes y sont, mais si la console brûle, tout est perdu. Un **remote** est le « cloud gaming » du code : une copie de l'historique hébergée ailleurs (GitHub, GitLab, un serveur interne), que tu et ton équipe pouvez interroger et enrichir.

En DevOps, c'est encore plus fondamental : le remote est **la source de vérité**. C'est lui qui déclenche les pipelines CI/CD (Bloc 11), sert de source aux déploiements GitOps (Bloc 13), et héberge les revues de code. Un travail non poussé = un travail qui n'existe pas pour l'équipe.

### Comment ? Local et remote sont DEUX historiques

Point essentiel pour tout comprendre : `origin` n'est pas un « dossier partagé en direct ». C'est **une copie** de l'historique, sur laquelle tu ne travailles jamais directement. Le flux est :

```
TON POSTE                                        SERVEUR (GitHub)
┌──────────────┐   git push    ┌──────────────┐
│ Dépôt local  │ ────────────► │  origin/main │   (tu ENVOIES tes commits)
│ main         │ ◄──────────── │              │
└──────────────┘   git fetch   └──────────────┘   (tu TÉLÉCHARGES sans fusionner)
                   git pull = fetch + merge         (tu TÉLÉCHARGES et fusionnes)
```

- **`git push`** : envoie tes commits locaux vers le remote.
- **`git fetch`** : télécharge ce qui a changé côté remote **sans toucher à tes fichiers**. Sûr à 100 % : il ne casse rien. Tu inspectes ensuite (`git log origin/main`) et tu décides.
- **`git pull`** : `fetch` **puis** `merge` en une commande. Rapide, mais fusionne « à l'aveugle » — d'où les mauvaises surprises possibles.
- **`origin`** : le nom par défaut du premier remote ajouté. Une convention, pas une magie : tu peux avoir plusieurs remotes (`origin`, `upstream`…).

### Quand fetcher plutôt que puller ?

Réflexe professionnel : au début de ta journée, `git fetch` puis regarde `git log HEAD..origin/main --oneline` (ce que les autres ont poussé). Si ça te convient, `git merge origin/main` (ou un simple `git pull`). Le `pull` direct reste OK pour un travail solo rapide.

---

## 📖 Vocabulaire / Abréviations

| Terme | Définition (une ligne) |
|---|---|
| **Remote** | un dépôt Git distant (GitHub, GitLab…) relié au tien |
| **origin** | le nom par défaut du remote principal |
| **Clone** | copier un dépôt distant sur ta machine (avec tout son historique) |
| **Push** | envoyer tes commits locaux vers le remote |
| **Pull** | récupérer les commits du remote et les intégrer (= fetch + merge) |
| **Fetch** | récupérer les nouveautés du remote SANS les intégrer |
| **Pull Request (PR)** | proposition de fusion relue par d'autres avant intégration (Leçon 5) |
| **Upstream / tracking** | le lien entre ta branche locale et sa jumelle distante (`-u`) |

---

## 3. Exemples concrets

> 🧪 On publie ton dépôt `outil-diagnostic` des leçons précédentes. Crée d'abord un dépôt **vide** (sans README) nommé `outil-diagnostic` sur github.com.

### 3.1 Authentification SSH (à faire une seule fois)

```bash
# Générer une paire de clés (ed25519 = standard actuel)
ssh-keygen -t ed25519 -C "elgissio@exemple.com"
# → Accepte l'emplacement par défaut ~/.ssh/id_ed25519 et une passphrase

# Afficher la clé PUBLIQUE (celle-là seule va sur GitHub)
cat ~/.ssh/id_ed25519.pub

# Copie-la dans GitHub → Settings → SSH and GPG keys → New SSH key
# Puis teste :
ssh -T git@github.com
# → Hi ElgissioFranito! You've successfully authenticated...
```

> ℹ️ **Pourquoi SSH et pas HTTPS ?** En HTTPS, il faut un *Personal Access Token* (mot de passe d'application) à chaque interaction. En SSH, ta clé authentifie tout, sans effort quotidien. C'est la norme en entreprise (avec des agents SSH et des clés par projet).

### 3.2 Publier un dépôt local existant

```bash
cd ~/projets/outil-diagnostic

# Déclarer le remote (URL en SSH)
git remote add origin git@github.com:ElgissioFranito/outil-diagnostic.git

# Vérifier
git remote -v
# → origin  git@github.com:ElgissioFranito/outil-diagnostic.git (fetch)
# → origin  git@github.com:ElgissioFranito/outil-diagnostic.git (push)

# Premier push : -u crée le lien de suivi main -> origin/main
git push -u origin main
```

Le `-u` (ou `--set-upstream`) enregistre que **ta** branche `main` suit `origin/main`. Après ça, un simple `git push` / `git pull` suffit, Git sait tout seul.

### 3.3 Cloner et cycle fetch/pull

```bash
# Récupérer un projet existant (clone = copie complète + remote origin déjà réglé)
git clone git@github.com:ElgissioFranito/outil-diagnostic.git
cd outil-diagnostic
git branch -vv   # montre les liens de suivi : main...origin/main

# Ce que les autres ont poussé, SANS toucher à ton travail :
git fetch origin
git log HEAD..origin/main --oneline   # les commits distants que tu n'as pas

# Les intégrer :
git merge origin/main   # ou simplement : git pull
```

### 3.4 Le cas « push rejeté » (non-fast-forward)

Scénario quotidien : un collègue a poussé pendant que tu travaillais.

```bash
git push
# → ! [rejected] main -> main (fetch first)
```

Ce n'est **pas une erreur**, c'est une **protection** : Git refuse d'écraser l'historique distant sans que tu intègres le travail des autres.

```bash
git pull                 # intègre origin/main dans ton travail (crée un merge)
# ou, mieux (voir Leçon 4) :
git pull --rebase        # rejoue tes commits par-dessus ceux du remote

git push                 # passe maintenant
```

## 4. Bonnes pratiques modernes (2025-2026)

1. **SSH (ed25519) plutôt que HTTPS+token** pour le quotidien ; les tokens restent utiles pour la CI.
2. **`main` protégée** sur GitHub/GitLab (*branch protection* : pas de push direct, tout passe par PR) — c'est la pratique d'équipe standard, qu'on simule à la Leçon 5.
3. **Pusher ses branches de feature régulièrement** : `git push -u origin feature/check-services`. Une branche non poussée = un travail invisible et non sauvegardé.
4. **`git fetch` avant de décider** : inspecter `git log HEAD..origin/main` évite les merges surprises.
5. **`git pull --rebase`** configuré par défaut pour un historique lisible :
   ```bash
   git config --global pull.rebase true
   ```
6. **Jamais de `--force` sur une branche partagée** ; si tu dois vraiment forcer (après un rebase de TA branche de feature), utilise `--force-with-lease` qui vérifie que personne n'a poussé entre-temps.

## 5. Pièges à éviter

### ❌ Piège 1 : croire que `git push` « synchronise »

```bash
# MAUVAIS réflexe : "je push donc on est à jour"
git push          # n'envoie QUE tes commits, ne ramène RIEN du remote
```

```bash
# BON : je ramène d'abord, puis j'envoie
git fetch origin && git log HEAD..origin/main --oneline   # qu'y a-t-il côté remote ?
git pull --rebase
git push
```

### ❌ Piège 2 : `git push --force` sur `main`

```bash
git push --force   # MAUVAIS : écrase l'historique distant, les commits des autres disparaissent
```

```bash
# BON : jamais de force sur une branche partagée ; si nécessaire sur TA feature :
git push --force-with-lease origin feature/ma-branche
```

### ❌ Piège 3 : cloner dans un dépôt déjà Git (dépôt imbriqué)

```bash
cd ~/projets/outil-diagnostic   # déjà un dépôt !
git clone git@github.com:.../autre-projet.git   # MAUVAIS : crée un sous-dépôt imbriqué
```

Clone toujours **hors** d'un dépôt existant, dans un dossier neutre.

### ❌ Piège 4 : committer sa clé privée

`~/.ssh/id_ed25519` (la clé **privée**) ne quitte **jamais** ta machine. Seule la clé `.pub` va sur GitHub. Une clé privée poussée sur un repo = clé compromise, à révoquer immédiatement.

## 6. Exercice pratique

👉 Voir **`02-exercice.md`** : publier ton dépôt sur GitHub, cloner une « copie collègue », simuler un travail à deux et résoudre un push rejeté.

## 7. Correction de l'exercice

👉 Voir **`03-correction.md`**.

## 8. Checklist de validation

- [ ] Je sais expliquer que local et remote sont **deux historiques** et ce que fait `origin`.
- [ ] Je sais publier un dépôt local (`remote add` + `push -u`) et cloner un projet.
- [ ] Je sais configurer l'authentification **SSH** et la tester (`ssh -T`).
- [ ] Je connais la différence **fetch** (télécharger) / **pull** (télécharger + fusionner) et je sais inspecter avant de fusionner.
- [ ] Je sais réagir à un **push rejeté** sans forcer.
- [ ] Je sais pourquoi `--force` est interdit sur une branche partagée (et ce que fait `--force-with-lease`).

---

*Prochaine étape :* Leçon 4 — **Rebase, stash et historique propre** dans `04-Rebase-stash-et-historique-propre/` : garder un historique lisible et changer de branche sans committer du travail à moitié fini.

