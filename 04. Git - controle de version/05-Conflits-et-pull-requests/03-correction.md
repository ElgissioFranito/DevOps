# Correction détaillée — Leçon 5 : conflits et Pull Request

## Partie 1 — Conflit en merge

```bash
# Collègue
cd ~/projets/collegue-diagnostic
git switch -c fix/fin-rapport
sed -i 's/=== Fin du diagnostic ===/--- Rapport termine ---/' diagnostic.sh
git add diagnostic.sh && git commit -m "fix: ligne de fin de rapport plus explicite"
git push -u origin fix/fin-rapport

# Toi
cd ~/projets/outil-diagnostic
git switch -c feature/en-tete
sed -i 's/=== Fin du diagnostic ===/=== FIN ===/' diagnostic.sh
git add diagnostic.sh && git commit -m "feat: fin de rapport compacte"

git fetch origin
git merge origin/main
# → CONFLICT (content): Merge conflict in diagnostic.sh
```

**Pourquoi un conflit ici ?** La même ligne (`=== Fin du diagnostic ===`) a été remplacée par deux textes différents. Git fusionne tout seul les changements sur des lignes distinctes ; sur des lignes communes, il refuse d'arbitrer.

Résolution, étape par étape :

```bash
git status          # 1) identifier : "both modified: diagnostic.sh"
```

```text
2) comprendre — le fichier contient :
   <<<<<<< HEAD
   echo "=== FIN ==="              ← ton intention : compact
   =======
   echo "--- Rapport termine ---"  ← intention du collègue : explicite
   >>>>>>> origin/main
```

L'intention « explicite » est la meilleure : on garde la ligne du collègue et on **supprime les 3 lignes de marqueurs** (3).

```bash
bash -n diagnostic.sh && ./diagnostic.sh    # 4) tester : la sortie affiche "--- Rapport termine ---"
grep -n '<<<<<<<\|>>>>>>>' diagnostic.sh    # ne doit rien afficher ✔
git add diagnostic.sh
git commit                                   # 5) commit de fusion (message par défaut de Git)
```

`git log --oneline --graph` montre maintenant le commit de fusion avec deux parents.

## Partie 2 — Conflit en rebase

Même collision, autre contexte :

```bash
git switch feature/test-conflit
git rebase origin/main
# → CONFLICT (content): Merge conflict in diagnostic.sh
# → error: could not apply <hash>... feat: ...
```

Le rebase est **en pause** : il rejouait ton commit et bute sur le conflit. Après résolution :

```bash
git add diagnostic.sh
git rebase --continue        # PAS "git commit" : c'est le rebase qui pilote
```

**Vocabulaire à retenir** : en **merge**, tu termines avec `git commit` (tu crées le commit de fusion) ; en **rebase**, tu termines avec `git rebase --continue` (le rebase reprend sa liste de commits à rejouer). Et `git rebase --abort` à tout moment remet l'état d'avant — le bonus le démontre : `git log --oneline` est identique avant/après l'abandon.

## Partie 3 — Pull Request de bout en bout

1. `git push -u origin feature/en-tete` si ce n'est pas déjà fait, puis sur GitHub : *Compare & pull request*. La description en 3 sections (Quoi / Pourquoi / Comment tester) est ce qui rend une PR **reviewable** — un titre seul oblige le réviseur à deviner.
2. Review : *Files changed* → survole une ligne → `+` → commentaire (ex. « Pense à réutiliser ce bloc pour le check mémoire ») → *Review changes* → **Approve**.
3. Fusion : *Merge pull request* → *Confirm merge* → *Delete branch* (supprime la branche distante ; le commit reste, seule l'étiquette disparaît).
4. Synchronisation locale :

```bash
git switch main
git pull --rebase          # ramène le merge de la PR
git branch -d feature/en-tete
```

## Bonus — branch protection

Le `git push` direct sur `main` est refusé par la plateforme :

```
remote: error: GH006: Protected branch update failed for refs/heads/main.
```

C'est exactement le comportement attendu : la **protection technique** force le workflow `branche → PR → review → merge`. Tu viens de voir la raison pour laquelle toutes les pratiques des leçons 4-5 existent.

---

## ✅ Checklist de validation (réécrite)

- [ ] Je sais provoquer puis reconnaître un conflit (`git status` → *both modified*).
- [ ] J'applique les 5 étapes : identifier → comprendre (les **intentions**) → résoudre → **tester** (`bash -n` + exécution) → commit.
- [ ] Je vérifie l'absence de marqueurs avant de committer (`grep '<<<<<<<'`).
- [ ] Je connais la différence de fin de conflit : `git commit` (merge) vs `git rebase --continue` (rebase), et l'issue de secours `--abort`.
- [ ] Je sais ouvrir une PR structurée (Quoi/Pourquoi/Comment tester), la reviewer et la merger.
- [ ] Je sais expliquer PR (GitHub) vs MR (GitLab) et pourquoi `main` est protégée en équipe.
- [ ] Je sais situer SVN/CVS comme systèmes **centralisés** anciens, par opposition au Git distribué.

## 🧭 Conseils

- **Réflexe** : un conflit n'est pas un accident, c'est une **décision à prendre**. Prends 30 secondes pour comprendre chaque intention avant de trancher, et teste toujours après.
- **Munition de vocabulaire** : marqueurs de conflit, both modified, commit de fusion, rebase en pause, PR/MR, review, branch protection, squash merge.
- Tu maîtrises maintenant le **workflow complet de la roadmap** : `feature branch → commit → push → PR → review → merge`. La Leçon 6 ajoute la compétence de survie (réparer tes erreurs) puis l'enchaîne tout entier dans un projet récapitulatif.

---

*Prochaine étape :* Leçon 6 — **Réparation et projet récapitulatif** dans `06-Reparation-et-projet-recapitulatif/`.
