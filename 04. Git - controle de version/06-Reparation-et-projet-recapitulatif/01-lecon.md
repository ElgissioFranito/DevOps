# Leçon 6 — Réparation (reset, revert, reflog) et projet récapitulatif

> **Bloc 04 — Git, Leçon 6/6.** La roadmap l'exige explicitement : « récupérer un projet après une mauvaise manipulation Git ». Cette leçon te donne les outils de réparation, puis le **projet récapitulatif** enchaîne le workflow complet des leçons 1 à 5. À la fin, tu coches le critère du bloc : *« une équipe te donne un repository et tu peux travailler dessus proprement, sans casser l'historique ni avoir peur des conflits »*.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. Distinguer les **trois niveaux d'annulation** : modifier le dernier commit (`--amend`), annuler des commits partagés (`revert`), réécrire l'historique local (`reset`).
2. Choisir entre `reset --soft`, `--mixed` et `--hard` en connaissant exactement ce que chacun conserve.
3. Utiliser `git reflog` pour **tout retrouver** — y compris des commits « perdus » — et comprendre pourquoi rien ne se perd vraiment.
4. Annuler un commit **déjà poussé** sans réécrire l'historique (`git revert`).
5. Enchaîner seul le **workflow complet** : branche → commits → push → PR → review → merge.

---

## 2. Explication simple

### Pourquoi savoir réparer ?

À force de manipuler, tu vas un jour committer sur la mauvaise branche, committer un fichier qu'il ne fallait pas, ou faire un `reset --hard` de trop. Un DevOps qui **panique** devant Git est dangereux ; un DevOps qui sait **réparer** est précieux. La règle rassurante à retenir : tant que tu as **commité**, Git a **presque tout retenu** pendant des semaines (reflog).

### Comment ? Trois outils, trois situations

**1. `--amend` — corriger le dernier commit** (Leçon 1) : faute dans le message ou fichier oublié **avant** tout push. Il remplace le dernier commit par une version corrigée.

**2. `git revert` — annuler un commit partagé** : il ne supprime rien ; il crée un **nouveau commit** qui fait **l'inverse** du commit visé. L'historique avance, rien n'est réécrit → **sans danger** sur une branche poussée et partagée.

```
A ── B ── C ── D      D introduit un bug
            └─ D' ──►  git revert D  : D' annule exactement D (nouveau commit !)
```

**3. `git reset` — reculer le curseur local** : il déplace la branche vers un commit antérieur, **réécrivant** l'historique local. Trois modes, qui diffèrent par ce qu'ils font de tes fichiers et de l'index :

| Mode | La branche | L'index (staging) | Tes fichiers |
|---|---|---|---|
| `--soft` | recule | **conservé** (tout est staged) | conservés |
| `--mixed` (défaut) | recule | **vidé** (modifications en rép. de travail) | conservés |
| `--hard` | recule | vidé | **écrasés — dangereux** |

Analogie : `--soft` tu décroches la photo du mur mais la gardes en main ; `--mixed` tu la poses sur la table ; `--hard` tu la déchires.

### Le filet de sécurité : `git reflog`

Git enregistre **chaque déplacement de HEAD** (`reflog`). Même après un `reset --hard`, l'ancien commit existe encore dans le dépôt — il est juste « orphelin ». `git reflog` le montre, et `git reset --hard <hash>` (ou une branche créée dessus) le ramène. C'est la raison pour laquelle on ne supprime **jamais** le dossier `.git/`.

### Quand utiliser quoi ?

- Erreur **non poussée** sur le dernier commit → `--amend`.
- Erreur **poussée/partagée** → `git revert` (jamais de reset de force).
- Nettoyage **local** (commits en trop avant rebase, mauvaise branche) → `reset --soft` ou `--mixed`.
- Tout a disparu après une manipulation → `git reflog` puis `reset --hard <hash>`.

---

## 📖 Vocabulaire / Abréviations

| Terme | Définition (une ligne) |
|---|---|
| **git reset** | déplacer HEAD (soft : garde les fichiers, hard : tout écrase) |
| **git revert** | créer un commit INVERSE (annule en ajoutant, sans réécrire l'historique) |
| **git commit --amend** | corriger le dernier commit (message ou contenu) |
| **reflog** | journal des déplacements de HEAD : retrouve un commit « perdu » |
| **Détaché (detached HEAD)** | être posé sur un commit, pas sur une branche |
| **Branche orpheline (dangling)** | commit plus référencé par aucune branche — récupérable via reflog |
| **Rotation (secret)** | révoquer/remplacer une clé exposée (leçon 7 du Bloc 5) |
| **Clone d'entraînement** | copie du dépôt où l'on casse volontairement des choses |

---

## 3. Exemples concrets

> 🧪 Sur `outil-diagnostic`. On va casser exprès, puis réparer.

### 3.1 Reset : les trois modes

```bash
# Deux commits de trop sur main (non poussés)
git log --oneline -3
# → c3d4e5f (HEAD -> main) feat: a ne pas garder
# → b2c3d4e docs: a ne pas garder
# → a1b2c3d feat: dernier bon commit

# --soft : tout remonte au premier plan (staged)
git reset --soft a1b2c3d
git status      # → Changes to be committed (tes modifications sont prêtes à recommiter)

# --mixed : pareil, mais déstagé
git reset --mixed a1b2c3d
git status      # → Changes not staged

# --hard : tout disparaît (dangereux, réfléchis avant)
git reset --hard a1b2c3d
git status      # → working tree clean
```

### 3.2 Reflog : la résurrection

```bash
# Oups, le --hard a effacé un commit utile
git reflog
# → a1b2c3d HEAD@{0}: reset: moving to a1b2c3d
# → c3d4e5f HEAD@{1}: commit: feat: a ne pas garder
# → ...

# Le commit "perdu" c3d4e5f est toujours là : on le restaure
git reset --hard c3d4e5f
git log --oneline -1   # → c3d4e5f : de retour ✔
```

### 3.3 Revert : annuler du partagé

```bash
git log --oneline -3
# → d4e5f6a (HEAD -> main) feat: bloc charge systeme (bug)

git revert d4e5f6a          # Git crée le commit inverse, l'éditeur s'ouvre pour le message
# → Revert "feat: bloc charge systeme"
# → This reverts commit d4e5f6a.

git log --oneline -3        # le commit bug EST TOUJOURS dans l'historique, suivi de son annulation
git push                    # réversible, honnête, sans danger pour l'équipe
```

### 3.4 Le projet récapitulatif — le workflow complet

Le déroulé détaillé est dans **`02-exercice.md`** ; voici la carte du voyage, exactement le schéma de la roadmap :

```
1. git switch -c feature/rapport-final       # branche
2. ... commits atomiques ...                 # commit (Leçon 1-2)
3. git push -u origin feature/rapport-final  # push (Leçon 3)
4. Pull Request + description 3 sections     # PR (Leçon 5)
5. self-review + Approve                     # review
6. Merge pull request                        # merge
7. git switch main && git pull --rebase      # resynchroniser
```

## 4. Bonnes pratiques modernes (2025-2026)

1. **`revert` par défaut, `reset` en dernier recours** : sur une branche partagée, la seule annulation acceptable est `revert` (ou un nouveau commit correctif).
2. **`reflog` avant de paniquer** : 90 % des « j'ai tout perdu » se règlent en 2 commandes.
3. **Alias de sécurité** : `git config --global alias.undo "reset --soft HEAD~1"` pour le cas fréquent « je veux défaire mon dernier commit local, en gardant les changements ».
4. **`--hard` toujours précédé d'un `git status`** : confirme ce que tu es sur le point d'écraser.
5. **Les protections de branche** (Leçon 5) font aussi office de garde-fou : elles empêchent le pire (force push, réécriture) sur `main`.
6. **Outil graphique si besoin** : `git log --oneline --graph --all` dans le terminal, ou une UI (GitLens, lazygit) pour **visualiser** avant d'agir. Visualiser, c'est déjà réparer à moitié.

## 5. Pièges à éviter

### ❌ Piège 1 : `reset --hard` sur une branche partagée

```bash
git switch main && git reset --hard HEAD~3 && git push --force
# MAUVAIS : 3 commits de l'équipe viennent de disparaître du remote
```

```bash
# BON : annulation visible et sûre
git revert d4e5f6a c3d4e5f b2c3d4e    # un commit d'annulation par commit fautif
```

### ❌ Piège 2 : confondre « annuler » et « faire disparaître »

`revert` **préserve** l'historique (le bug et son annulation y figurent) : c'est une **trace honnête**, exigée en entreprise. « Faire disparaître » un commit poussé n'est acceptable que sur ta branche de feature, jamais sur `main`.

### ❌ Piège 3 : faire confiance à sa mémoire plutôt qu'au reflog

« Je crois que le commit s'appelait... » — non. `git reflog` + `git show <hash>` te donnent les **faits**. Le reflog couvre aussi les rebase interrompus et les resets : tout déplacement de HEAD y figure (~90 jours par défaut).

### ❌ Piège 4 : `reset --hard` avec du travail non commité

Le `--hard` écrase **aussi** tes modifications non commitées — elles, en revanche, **ne sont pas dans le reflog** (jamais commitées = jamais enregistrées). Avant un `--hard` : stash ou commit de sécurité sur une branche `wip/...`.

## 6. Exercice pratique

👉 Voir **`02-exercice.md`** : 3 scénarios de crash à réparer, puis le projet récapitulatif complet.

## 7. Correction de l'exercice

👉 Voir **`03-correction.md`**.

## 8. Checklist de validation

- [ ] Je sais corriger le dernier commit non poussé avec `--amend`.
- [ ] Je sais annuler un commit poussé avec `git revert` (et j'explique pourquoi pas `reset`).
- [ ] Je connais les 3 modes de `reset` (`--soft` / `--mixed` / `--hard`) et ce que chacun conserve.
- [ ] Je sais retrouver des commits « perdus » avec `git reflog` + `git reset --hard <hash>`.
- [ ] Je sais qu'un travail **jamais commité** n'est récupérable nulle part.
- [ ] J'ai enchaîné seul le workflow complet : branche → commits → push → PR → review → merge → resync.

---

*Prochaine étape :* Fin du **Bloc 04** — relis la checklist globale de l'introduction du bloc. La suite logique de la roadmap : le Bloc 05 **Réseautage et sécurité**, puis tes commits alimenteront directement les pipelines du **Bloc 11 (CI/CD)** et le **Bloc 13 (GitOps)**.

