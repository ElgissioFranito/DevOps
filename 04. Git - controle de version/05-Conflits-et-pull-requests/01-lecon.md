# Leçon 5 — Conflits et Pull Requests

> **Bloc 04 — Git, Leçon 5/6.** Prérequis : leçons 1 à 4. C'est la leçon « travail en équipe » de la roadmap : résoudre un **conflit** sans panique (identifier → comprendre → résoudre → tester → commit) et soumettre son travail via une **Pull Request** (GitHub) ou **Merge Request** (GitLab).

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. Expliquer **pourquoi un conflit apparaît** (deux modifications concurrentes sur les mêmes lignes) et pourquoi ce n'est ni une erreur ni une punition.
2. Suivre le processus de résolution exigé par la roadmap : **identifier → comprendre → résoudre → tester → commit**.
3. Lire et éditer les **marqueurs de conflit** (`<<<<<<<`, `=======`, `>>>>>>>`).
4. Abandonner proprement une fusion qui tourne mal (`merge --abort`).
5. Créer une **Pull Request**, la décrire pour un réviseur, et la fusionner après review.
6. Situer **GitHub et GitLab** (et le vieux SVN) dans l'écosystème DevOps.

---

## 2. Explication simple

### Pourquoi les conflits ?

Deux personnes réparent la même fuite en même temps, chacune à sa façon. Git ne peut pas inventer la version finale : il te signale la collision et **te laisse trancher**. C'est un conflit. Concrètement : Git fusionne automatiquement tant que les changements touchent des **lignes différentes** ; il se déclare en échec uniquement quand **les mêmes lignes** ont été modifiées des deux côtés (ou quand un fichier a été supprimé d'un côté et modifié de l'autre).

### Comment ? Les marqueurs de conflit

Git te laisse le fichier en l'état, avec les deux versions visibles :

```
<<<<<<< HEAD
echo "Charge : $(cat /proc/loadavg)"
=======
echo "Charge systeme : $(uptime)"
>>>>>>> feature/check-load
```

Lecture : `<<<<<<<` ouvre la zone, `HEAD` = **ta** version (celle de la branche où tu es), `=======` sépare, `>>>>>>>` = **l'autre** version. Ton travail : choisir l'une, l'autre, ou **combiner**, puis retirer les marqueurs. Le fichier avec ses marqueurs ne doit **jamais** être commité tel quel.

### Le processus en 5 étapes (celui de la roadmap)

```
identifier le conflit  →  git status liste les fichiers "both modified"
        ↓
comprendre les changements  →  ouvre le fichier, lis les 2 versions :
        ↓                        « quelle est l'intention de chacun ? »
résoudre  →  édite pour garder la bonne version (les marqueurs disparaissent)
        ↓
tester  →  exécute le script/le code : la fusion doit FONCTIONNER
        ↓
commit  →  git add <fichier> puis git commit (ou git rebase --continue)
```

L'étape **tester** est celle que les débutants sautent — et c'est elle qui fait la différence : un conflit « résolu » qui casse le build est pire qu'un conflit laissé en attente.

### La Pull Request : « vérifiez mon code avant de l'intégrer »

Sur un projet d'équipe, **personne ne pousse directement sur `main`**. On pousse sa branche de feature, puis on ouvre une **PR** : une page sur la plateforme qui montre *ce qui changerait* si on fusionnait (diff), où les collègues **commentent ligne par ligne**, demandent des modifications, et où la CI peut lancer les tests (Bloc 11). Quand tout est vert et validé : **merge**. C'est l'assurance qualité du code moderne.

### Plateformes : GitHub, GitLab (et le vieux monde)

- **GitHub** : la plateforme la plus répandue ; terme = **Pull Request** ; CI = GitHub Actions.
- **GitLab** : très présente en entreprise (souvent auto-hébergée) ; terme = **Merge Request** (même idée) ; CI intégrée = GitLab CI/CD.
- Les deux fournissent : repos, issues, PR/MR, CI/CD, permissions, releases. Git reste **le moteur** : tout ce que tu as appris s'applique identiquement sur les deux.
- Culture générale : **SVN** (centralisé : un seul serveur garde l'historique) et **CVS** (son ancêtre) se croisent encore sur des projets anciens. Rétention : « centralisé ≠ distribué », rien de plus.

---

## 📖 Vocabulaire / Abréviations

| Terme | Définition (une ligne) |
|---|---|
| **Conflit** | deux modifications concurrentes sur les mêmes lignes que Git refuse d'arbitrer |
| **Marqueurs de conflit** | les repères `<<<<<<<`, `=======`, `>>>>>>>` dans le fichier à résoudre |
| **Résolution** | choisir/écrire la bonne version, puis `git add` le fichier réconcilié |
| **Pull Request (PR)** | demande de fusion relue (review) avant d'intégrer dans la branche principale |
| **Review** | lecture critique des changements par un pair (commentaires, suggestions) |
| **Approve** | validation de la review, condition pour merger |
| **Merge commit** | commit spécial qui « noue » deux branches |
| **CI** | le pipeline qui teste automatiquement chaque PR (Bloc 11) |

---

## 3. Exemples concrets

> 🧪 On fabrique un vrai conflit entre « toi » et « ton collègue » (le second clone de la Leçon 3).

### 3.1 Provoquer le conflit

```bash
# Collègue : modifie la ligne de fin de rapport et pousse
cd ~/projets/collegue-diagnostic
git switch -c fix/fin-rapport
sed -i 's/=== Fin du diagnostic ===/--- Rapport termine ---/' diagnostic.sh
git add diagnostic.sh && git commit -m "fix: ligne de fin de rapport plus explicite"
git push -u origin fix/fin-rapport

# Toi : modifie LA MÊME ligne, autrement, sur ta branche
cd ~/projets/outil-diagnostic
git switch -c feature/en-tete
sed -i 's/=== Fin du diagnostic ===/=== FIN ===/' diagnostic.sh
git add diagnostic.sh && git commit -m "feat: fin de rapport compacte"
```

### 3.2 Le merge qui coince

```bash
git fetch origin
git merge origin/main
# → CONFLICT (content): Merge conflict in diagnostic.sh
# → Automatic merge failed; fix conflicts and then commit the result.

git status
# → Unmerged paths: both modified: diagnostic.sh   ← ÉTAPE 1 : identifier
```

### 3.3 Comprendre, résoudre, tester

```bash
# ÉTAPE 2 : comprendre — ouvre le fichier, tu verras :
# <<<<<<< HEAD
# echo "=== FIN ==="
# =======
# echo "--- Rapport termine ---"
# >>>>>>> origin/main
```

Décision : la version du collègue est plus explicite, on la garde (ÉTAPE 3 : supprime les 3 lignes de marqueurs et garde sa ligne).

```bash
# ÉTAPE 4 : tester — non négociable
bash -n diagnostic.sh && ./diagnostic.sh     # syntaxe + exécution réelle

# ÉTAPE 5 : commit de fusion
git add diagnostic.sh
git commit               # Git propose un message de merge par défaut
git log --oneline -1
```

### 3.4 La porte de sortie

```bash
git merge --abort        # annule le merge, revient à l'état d'avant (comme rebase --abort)
```

### 3.5 Ouvrir une Pull Request

Côté plateforme (GitHub), cela se fait **dans l'interface** : bouton *Compare & pull request* (ou onglet *Pull requests* → *New*), base = `main`, compare = ta branche. Description type :

```
## Quoi
Ajout de la vérification de charge système.

## Pourquoi
Complète le diagnostic (disque, mémoire, services) demandé par l'équipe.

## Comment tester
./diagnostic.sh → un bloc "Charge systeme" s'affiche.
```

Le réviseur lit l'onglet *Files changed*, commente, approuve ; tu fusionnes avec *Merge pull request*. Chez GitLab : mêmes étapes, nommées *Merge Request*.

## 4. Bonnes pratiques modernes (2025-2026)

1. **Tout passe par une PR/MR** : branche protégée sur `main`, au moins une review. Même seul, prends ce réflexe — c'est lui qui est attendu en entreprise.
2. **PR courtes** : < ~400 lignes modifiées si possible. Une PR de 3000 lignes n'est jamais vraiment relue.
3. **Description qui permet de tester** : quoi / pourquoi / comment tester. Le réviseur ne devine pas.
4. **Rebase avant d'ouvrir la PR** (Leçon 4) : ta PR est à jour de `main` et le réviseur ne digère pas tes conflits.
5. **Résoudre les conflits dans le bon sens** : quand la PR dit « conflict », c'est **ta branche** qui se met à jour (`git rebase origin/main` ou `git merge origin/main` depuis ta feature), jamais l'inverse.
6. **Relecture bienveillante et factuelle** : on commente le code, pas la personne ; les demandes sont concrètes (« extraire cette fonction ») pas vagues (« bof »).

## 5. Pièges à éviter

### ❌ Piège 1 : commité les marqueurs de conflit

```bash
git add . && git commit -m "fix merge"   # MAUVAIS : le fichier contient encore <<<<<<< HEAD
```

```bash
# BON : je vérifie qu'il ne reste AUCUN marqueur avant d'ajouter
grep -n '<<<<<<<\|=======\|>>>>>>>' diagnostic.sh   # ne doit rien afficher
git add diagnostic.sh && git commit
```

### ❌ Piège 2 : « résoudre » sans comprendre ni tester

Prendre machinalement « la version du haut » (`HEAD`) pour aller vite. La bonne question : **quelle est l'intention de chaque modification ?** Puis exécuter le code. Un conflit mal tranché casse silencieusement une fonctionnalité.

### ❌ Piège 3 : la PR de 3 semaines

Une branche ouverte longtemps diverge : conflits en cascade, review impossible. Une PR doit vivre **heures ou jours**, pas semaines. Découpe le travail.

### ❌ Piège 4 : push direct sur `main` « parce que c'était petit »

Le « petit push » contourne la review et la CI. S'il casse la prod, personne ne l'a vu venir. Même une typo passe par une branche + PR (en équipe).

## 6. Exercice pratique

👉 Voir **`02-exercice.md`** : provoquer et résoudre un conflit en 2 contextes (merge et rebase), puis ouvrir, reviewer et merger ta propre PR sur GitHub.

## 7. Correction de l'exercice

👉 Voir **`03-correction.md`**.

## 8. Checklist de validation

- [ ] Je sais expliquer quand un conflit apparaît (mêmes lignes modifiées des deux côtés).
- [ ] Je connais les 5 étapes : identifier → comprendre → résoudre → **tester** → commit.
- [ ] Je sais lire les marqueurs (`<<<<<<<` HEAD / `=======` / `>>>>>>>`) et je vérifie qu'ils ont tous disparu avant de committer.
- [ ] Je sais annuler un merge ou un rebase bloqué (`--abort`).
- [ ] Je sais ouvrir une PR avec une description quoi/pourquoi/comment-tester et la fusionner après review.
- [ ] Je sais situer GitHub vs GitLab (PR vs MR, CI intégrée) et ce qu'était SVN (centralisé).

---

*Prochaine étape :* Leçon 6 — **Réparation et projet récapitulatif** dans `06-Reparation-et-projet-recapitulatif/` : survivre à tes erreurs Git et enchaîner le workflow complet.


