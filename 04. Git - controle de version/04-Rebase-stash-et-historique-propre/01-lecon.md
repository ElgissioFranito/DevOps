# Leçon 4 — Rebase, stash et historique propre

> **Bloc 04 — Git, Leçon 4/6.** Prérequis : Leçons 1 à 3. Tu sais brancher, merger, pousser. Reste deux compétences qui séparent le pro du débutant : **réécrire proprement un historique** (`rebase`) et **mettre son travail en pause** sans commit bâclé (`stash`). La roadmap exige aussi de savoir **récupérer une mauvaise manipulation** — c'est l'objet de la Leçon 6, mais cette leçon te donne déjà les réflexes.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. Expliquer ce que fait `git rebase` : **rejouer** tes commits sur une nouvelle base.
2. Choisir entre **merge** et **rebase** en connaissance de cause (et connaître la « règle d'or » du rebase).
3. Synchroniser une branche de feature avec `main` sans commit de fusion parasite.
4. Utiliser `git stash` pour mettre de côté un travail en cours et le retrouver.
5. Utiliser `git pull --rebase` pour un historique **linéaire** au quotidien.
6. Nettoyer ses propres commits avant partage (`rebase -i`, squash) — sur SES branches uniquement.

---

## 2. Explication simple

### Pourquoi le rebase ?

Retour au scénario de la Leçon 3 : tu bosses sur `feature/check-load`, et pendant ce temps `main` avance. Deux façons de rattraper :

- **Merge** : `git merge main` crée un commit de fusion. Résultat : un historique en **arbre**, avec des « Merge branch 'main' into feature/... » qui polluent le graphe à chaque synchronisation.
- **Rebase** : Git **détache** tes commits, les **copie** et les **rejoue** un par un au sommet de `main`. Résultat : un historique **linéaire**, comme si tu avais commencé ton travail après les derniers changements.

```
Avant :                A ── B ── C   (main)
                        \    \
                         D ── E         (feature)

Merge   :  A ── B ── C ──── M ──►   M a 2 parents (un "nœud")
            \         \  /
             D ── E ───

Rebase  :  A ── B ── C ── D' ── E'   (D' et E' sont des COPIES : nouveaux hash)
```

### Comment ça marche (le point qui change tout) ?

Le rebase **crée de nouveaux commits** : mêmes contenus, mêmes messages, mais **nouveaux hash**. Les anciens `D`/`E` ne sont plus référencés. Conséquence — la **règle d'or** :

> ⚠️ **Ne rebase jamais des commits déjà poussés et partagés** avec l'équipe. Si tes commits ont été copiés, les copies distantes deviennent des « fantômes » et tout le monde casse. Tu rebase **tes** commits, **pas encore poussés** (ou sur **ta** branche de feature dont tu es le seul auteur).

### Et le stash ?

Tu es à moitié sur une modification, il faut urgemment corriger un bug sur une autre branche. Committer un travail à moitié fini = commit inutile. Abandonner = perte. **`git stash`** met tes modifications dans une « pile de mise de côté » et te rend un répertoire de travail **propre**. `git stash pop` les ressort plus tard.

### Quand utiliser quoi ?

| Situation | Outil |
|---|---|
| Intégrer le travail des autres dans ma feature | `git rebase main` (ou `merge` si l'équipe préfère les merges) |
| Ramener le remote dans ma branche | `git pull --rebase` |
| Urgence pendant un travail en cours | `git stash` → gérer l'urgence → `stash pop` |
| Fusionner une feature terminée dans `main` | `git merge` (le merge reste la façon d'intégrer une feature) |
| Nettoyer mes commits avant la PR | `git rebase -i` (squash, reword) |

---

## 3. Exemples concrets

> 🧪 Sur `outil-diagnostic`. Assure-toi que tout est commité avant de commencer (`git status` doit être clean).

### 3.1 Rebase d'une feature sur main

```bash
# Situation : main a avancé, ma feature aussi
git switch feature/check-load
git rebase main
# → Successfully rebased and updated refs/heads/feature/check-load.

git log --oneline          # mes commits sont maintenant APRÈS ceux de main
git push --force-with-lease   # la branche était déjà poussée → il faut "remplacer" (voir pièges)
```

### 3.2 Le stash

```bash
git switch main
echo "# brouillon en cours" >> notes.md     # travail à moitié fini

git stash push -m "brouillon de notes.md"
# → Saved working directory ... On main
git status
# → working tree clean : parfait, tu peux changer de branche sans risque

# ... autre urgence ...

git stash list
# → stash@{0}: On main: brouillon de notes.md
git stash pop
# → tes modifications reviennent, le stash est retiré de la pile
```

### 3.3 Pull --rebase au quotidien

```bash
git config --global pull.rebase true    # une fois pour toutes

# Désormais, chaque "git pull" rejoue tes commits locaux sur le remote :
git pull
# → First, rewinding head to replay your work on top of it...
```

### 3.4 Nettoyer ses commits avant la PR (rebase interactif)

```bash
# 3 commits de travail sur ma feature, dont 2 corrections de détail :
git log --oneline main..HEAD
# → e5f6a1b fix: typo dans le message de charge
# → d4c3b2a fix: valeur de load absente
# → c2b1a0f feat: affichage de la charge systeme

# Les regrouper en UN commit propre :
git rebase -i main
# Un éditeur s'ouvre. Remplace "fix" par "squash" (ou juste "s") :
#   pick  c2b1a0f feat: affichage de la charge systeme
#   s     d4c3b2a fix: valeur de load absente
#   s     e5f6a1b fix: typo dans le message de charge
# Sauvegarde → Git te demande le message final du commit fusionné.

git log --oneline main..HEAD
# → a9b8c7d feat: affichage de la charge systeme   (un seul commit, prêt pour la PR)
```

## 4. Bonnes pratiques modernes (2025-2026)

1. **Historique linéaire en équipe** : `pull.rebase true` en config globale ; beaucoup d'équipes activent aussi le *squash merge* côté GitHub/GitLab (chaque PR devient **un** commit sur `main`).
2. **Rebase uniquement tes commits non partagés** — règle d'or gravée. Pour tes branches de feature poussées, `--force-with-lease` est acceptable car **tu en es le seul auteur**.
3. **`rebase -i` avant chaque PR** : un commit propre et descriptif vaut mille fois 7 commits « fix typo » pour le réviseur.
4. **Stash nommé** (`git stash push -m "..."`) : un stash anonyme devient incompréhensible trois jours plus tard.
5. **Le stash n'est pas une poubelle à long terme** : il sert à quelques heures, pas à des semaines. Pour du travail à reprendre plus tard, commit sur une branche `wip/...` et pousse.
6. **Communique** : si l'équipe utilise des merges systématiques (certaines entreprises le font), respecte la convention locale — merge et rebase sont deux choix d'équipe valides, le tout est d'être cohérent.

## 5. Pièges à éviter

### ❌ Piège 1 : rebase des commits partagés

```bash
# MAUVAIS : rebase d'une branche sur laquelle TOUTE l'équipe travaille
git switch develop        # branche partagée
git rebase main           # réécrit l'historique de develop → chaos pour tous
```

```bash
# BON : rebase limité à TA branche de feature, dont tu es le seul auteur
git switch feature/check-load
git rebase main
```

### ❌ Piège 2 : `--force` classique au lieu de `--force-with-lease`

```bash
git push --force           # MAUVAIS : écrase sans vérifier — si un collègue a poussé entre-temps, son travail est perdu
```

```bash
git push --force-with-lease   # BON : n'écrase que si le remote est toujours dans l'état que tu as vu
```

### ❌ Piège 3 : le stash comme fourre-tout

```bash
git stash            # MAUVAIS : x5, sans message, depuis 3 semaines
git stash list       # → 5 entrées anonymes : impossible de savoir ce que c'est
```

```bash
# BON : stash ciblé, nommé, et vidé vite
git stash push -m "refonte du check disque, en attente du retour de Léa"
```

### ❌ Piège 4 : paniquer pendant un rebase

Un conflit pendant un `rebase` n'est pas une erreur : Git fait **pause** et te laisse trancher. On apprend la résolution complète à la Leçon 5 ; en attendant, sache qu'il existe toujours une porte de sortie : `git rebase --abort` remet tout exactement comme avant.

## 6. Exercice pratique

👉 Voir **`02-exercice.md`** : diverger, rebase, stash sous pression, nettoyer ses commits avant PR.

## 7. Correction de l'exercice

👉 Voir **`03-correction.md`**.

## 8. Checklist de validation

- [ ] Je sais expliquer le rebase (rejouer des commits sur une nouvelle base) et ses conséquences (nouveaux hash).
- [ ] Je connais la **règle d'or** : jamais de rebase de commits partagés.
- [ ] Je sais synchroniser ma feature avec `main` par rebase et pousser avec `--force-with-lease`.
- [ ] Je sais mettre un travail en pause avec `git stash push -m` et le récupérer avec `pop`.
- [ ] J'utilise `git pull --rebase` pour un historique linéaire.
- [ ] Je sais regrouper mes commits avec `git rebase -i` (squash) avant une PR.
- [ ] Je connais la porte de sortie d'un rebase bloqué : `git rebase --abort`.

---

*Prochaine étape :* Leçon 5 — **Conflits et Pull Requests** dans `05-Conflits-et-pull-requests/` : trancher les collisions et faire réviser ton code.

