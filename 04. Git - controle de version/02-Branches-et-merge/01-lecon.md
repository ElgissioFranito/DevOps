# Leçon 2 — Branches et merge

> **Bloc 04 — Git, Leçon 2/6.** Prérequis : savoir faire des commits propres (Leçon 1). Ici, on apprend la compétence qui distingue un utilisateur de Git d'un professionnel : **travailler sur plusieurs lignes de développement en parallèle** sans jamais mettre en danger la branche principale.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. Expliquer **ce qu'est une branche** (une simple étiquette mobile sur un commit) et à quoi elle sert.
2. Créer, changer et lister des branches avec `git branch` et `git switch`.
3. Comprendre la différence entre **fast-forward** et **merge à 3 voies (3-way)**.
4. Fusionner une branche de fonctionnalité dans `main` avec `git merge` **sans conflit**.
5. Lire l'historique **en arbre** avec `git log --oneline --graph`.
6. Appliquer la convention de nommage `feature/*`, `fix/*` exigée par la roadmap.

---

## 2. Explication simple

### Pourquoi des branches ?

Imagine que tu écris un roman. Ton manuscrit « publié » est bon, mais tu veux tenter un chapitre expérimental. Deux options :

- **Option kamikaze** : modifier le manuscrit publié directement. Si l'essai rate, ton texte est abîmé.
- **Option photocopie** : tu photocopies le manuscrit, tu expérimentes sur la copie, et si le résultat est bon, tu reportes les changements dans l'original.

En Git, la photocopie s'appelle une **branche**. Elle coûte zéro octet (c'est un simple pointeur), se crée en une milliseconde, et permet d'expérimenter sans risque.

### Comment ça marche sous le capot ?

Une branche n'est **pas une copie des fichiers** : c'est une **étiquette** posée sur un commit, qui avance automatiquement à chaque nouveau commit. **HEAD** est le curseur qui dit « sur quelle étiquette je me trouve » :

```
        a1b2c3d          e4f5g6h          i7j8k9l
main ──────────────────●────────────────●  ← HEAD est ici, main pointe sur le dernier commit
                        \
feature/login           ●─── m0n1o2p      ← feature/login avance indépendamment
```

Quand tu committes, **la branche courante** (celle sur laquelle pointe HEAD) avance. L'autre ne bouge pas. D'où l'importance de bien savoir **sur quelle branche tu es** (regarde `git status` ou le prompt).

### Le merge : deux façons de fusionner

Repartir d'une branche `feature/login` terminée et la fusionner dans `main` :

```bash
git switch main        # on se place sur la branche DESTINATAIRE
git merge feature/login
```

Deux cas possibles, à savoir distinguer :

**Cas 1 — Fast-forward** : `main` n'a pas bougé depuis la création de la branche. Git fait juste « avancer » l'étiquette `main` jusqu'au bout de la branche. Historique **linéaire**, pas de commit supplémentaire :

```
Avant :  a ── b ── c ── d      (main sur a-b ; feature sur c-d)
              main ────────►  (fast-forward : main rattrape)
Après :  a ── b ── c ── d   (main et feature sur le même commit)
```

**Cas 2 — Merge à 3 voies (3-way)** : les deux branches ont avancé chacune de leur côté. Git crée un **commit de fusion** avec **deux parents**, qui « noue » les deux lignes de développement :

```
      ┌── c ── d ──┐          ← feature/login
a ── b               m ──►    ← main, après le merge
      └── e ── f ──┘          ← main avait avancé de son côté
```

### Quand créer une branche ?

**Dès que tu commences un travail** qui ne doit pas polluer `main` : une fonctionnalité (`feature/export`), une correction (`fix/auth`), un test d'idée. En 2025-2026, la règle d'or des équipes est : **on ne commit jamais directement sur `main`** — tout passe par une branche + une Pull Request (on y vient à la Leçon 5).

---

## 3. Exemples concrets

> 🧪 Toujours sur ton dépôt `outil-diagnostic` (ou refais le setup de la Leçon 1 en 1 minute).

### 3.1 Créer et naviguer entre branches

```bash
# Lister les branches (l'étoile * indique la branche courante)
git branch
# → * main

# Créer une branche ET y basculer (deux commandes modernes, Git >= 2.23)
git switch -c feature/check-services
# → Switched to a new branch 'feature/check-services'

# Équivalent avec l'ancienne commande (à savoir LIRE, mais préfère switch)
# git checkout -b feature/check-services

# Où suis-je ?
git status
# → On branch feature/check-services
```

### 3.2 Travailler sur la branche

```bash
cat >> diagnostic.sh << 'EOF'
echo "Services :"
systemctl is-active nginx || echo "nginx inactif"
EOF

git add diagnostic.sh
git commit -m "feat: verification du service nginx"
```

> 💡 **Ton `main` est intact** : supprime même le bloc que tu viens d'ajouter, `main` n'a jamais vu ce commit. C'est ça, la sécurité des branches.

### 3.3 Merger : cas fast-forward

```bash
# Retour sur la branche destinataire (JAMAIS l'inverse !)
git switch main

# Fusionner le travail de la feature
git merge feature/check-services
# → Fast-forward ... main is now at <hash>

# Historique linéaire, comme si tu avais commité directement sur main
git log --oneline
```

### 3.4 Merger : cas 3-way (avec un commit de fusion)

```bash
# Pendant que feature/check-memory existe, main avance aussi :
git switch -c feature/check-memory
echo "echo 'Memoire detaillee :'" >> diagnostic.sh
echo "free -m"                        >> diagnostic.sh
git add diagnostic.sh && git commit -m "feat: memoire detaillee en Mo"

git switch main
echo "echo '=== Fin du diagnostic ==='" >> diagnostic.sh
git add diagnostic.sh && git commit -m "feat: ligne de fin de rapport"

# Les deux branches ont divergé → merge à 3 voies
git merge feature/check-memory
# → Merge made by the 'ort' strategy.   (un commit de fusion est créé)

# L'historique montre maintenant un "y" dans le temps
git log --oneline --graph --all
```

### 3.5 Nettoyer après le merge

```bash
# Une branche fusionnée ne sert plus à rien : on la supprime
git branch -d feature/check-memory
# → Deleted branch feature/check-memory (was e4f5g6h).

# -d refuse si la branche n'est PAS fusionnée (protection) ; -D force (dangereux)
git branch
```

## 4. Bonnes pratiques modernes (2025-2026)

1. **`main` est sacrée** : elle doit toujours être dans un état déployable. Tout travail passe par une branche (`feature/*`, `fix/*`, `docs/*`…). C'est exactement le schéma de la roadmap :
   ```
   main
    │
    ├── feature/login
    ├── feature/export
    └── fix/auth
   ```
2. **Branches courtes et ciblées** : une branche = une fonctionnalité, fusionnée en quelques heures/jours. Une branche qui vit 3 semaines devient impossible à fusionner.
3. **Nommage explicite** : `feature/ajout-check-disque`, `fix/crash-df-absent`, pas `ma-branche-v2`.
4. **`git switch` plutôt que `git checkout`** : `checkout` (1995) fait trop de choses ; `switch` (2019) change de branche, `restore` annule des modifications. Plus clair, moins d'erreurs.
5. **Synchronise régulièrement** : rebascule souvent `main` dans ta branche (on verra `rebase` à la Leçon 4) pour éviter les gros écarts.
6. **Supprime les branches fusionnées** : un dépôt avec 40 branches zombies ne rassure personne.

## 5. Pièges à éviter

### ❌ Piège 1 : travailler directement sur `main`

```bash
# MAUVAIS : je modifie et je commite sur main pour un gros chantier
git switch main
# ... 2 semaines de commits en vrac sur main ...
# → main est cassée, personne ne peut déployer, impossible de tester à part.
```

```bash
# BON : je crée une branche AVANT de toucher au code
git switch -c feature/ajout-check-disque
```

> 💡 Et si tu as déjà commité sur `main` par erreur ? Pas de panique : tes commits **suivent** avec toi. `git switch -c feature/mon-chantier` les emporte sur une nouvelle branche, et `main` peut revenir en arrière (on le fera proprement à la Leçon 6).

### ❌ Piège 2 : merger dans le mauvais sens

```bash
# MAUVAIS : je suis sur feature/... et je "merge main" en pensant finir le travail
git switch feature/check-services
git merge feature/check-services   # merge d'une branche dans elle-même, ou sens inversé
```

```bash
# BON : le merge se fait TOUJOURS depuis la branche destinataire
git switch main
git merge feature/check-services
```

> ℹ️ Nuance : faire `git switch feature/x && git merge main` (synchroniser sa feature avec main) est une pratique **valide et courante**. L'erreur est de croire que ça fusionne la feature dans main. Le sens du merge dépend **toujours de la branche sur laquelle tu es**.

### ❌ Piège 3 : la branche zombie

Une branche ouverte pour « un petit test » et jamais fermée. Trois semaines plus tard, elle diverge tellement qu'elle génère conflits sur conflits. Règle : **si ça ne sert pas, supprime** (`git branch -d` refuse les branches non fusionnées — c'est une protection, pas un bug).

## 6. Exercice pratique

👉 Voir **`02-exercice.md`** : deux fonctionnalités en parallèle sur l'outil de diagnostic, un fast-forward et un merge 3-way, puis nettoyage.

## 7. Correction de l'exercice

👉 Voir **`03-correction.md`** : correction pas à pas, explications, checklist et conseils.

## 8. Checklist de validation

- [ ] Je sais expliquer qu'une branche est une **étiquette** sur un commit, pas une copie des fichiers.
- [ ] Je sais créer/basculer/supprimer une branche (`switch -c`, `branch -d`) et savoir **sur quelle branche je suis**.
- [ ] Je sais distinguer **fast-forward** et **merge 3-way** et expliquer le commit de fusion.
- [ ] Je sais faire un merge dans le bon sens (depuis la branche destinataire).
- [ ] Je sais lire un historique en arbre (`git log --oneline --graph --all`).
- [ ] J'applique le modèle `main` + `feature/*` / `fix/*` et je ne commit plus jamais un chantier directement sur `main`.

---

*Prochaine étape :* Leçon 3 — **Travailler avec un remote** (`push`, `pull`, `fetch`) dans `03-Travailler-avec-un-remote/` : on emmène ton dépôt sur GitHub.

