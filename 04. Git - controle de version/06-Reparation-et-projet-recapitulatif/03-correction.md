# Correction détaillée — Leçon 6 : réparer, puis le projet récapitulatif

## Scénario A — le commit sur la mauvaise branche

```bash
# L'erreur : j'ai commité sur main
echo "# TODO charge reseau" >> diagnostic.sh
git add diagnostic.sh && git commit -m "feat: debut check reseau"

# La réparation : le commit ME SUIT quand je crée la branche,
# puis main recule d'un cran sans rien perdre
git switch -c feature/check-reseau     # le commit vient avec moi
git switch main
git reset --hard HEAD~1                # main revient avant l'erreur (local, non poussé → légitime)
git log --oneline -1                   # main est propre
git switch feature/check-reseau        # le commit est là ✔
```

**Le mécanisme** : un commit n'appartient pas à une branche ; la branche est une **étiquette**. On déplace l'étiquette `main` en arrière et on en pose une nouvelle (`feature/check-reseau`) sur le commit. `reset --hard` est ici sûr car le commit n'a **jamais été poussé**.

## Scénario B — le reset de trop

```bash
# 2 commits faits, puis l'erreur :
git reset --hard HEAD~2
git log --oneline -1        # les 2 commits ont disparu de la branche...

git reflog
# → b7a6c5d HEAD@{0}: reset: moving to HEAD~2
# → d8e9f0a HEAD@{1}: commit: feat: moyenne de charge
# → e9f0a1b HEAD@{2}: commit: feat: bloc ping

git reset --hard d8e9f0a    # retour à l'état d'avant le reset
git log --oneline -3        # les 2 commits sont de retour, mêmes hash ✔
```

**Pourquoi ça marche** : `reset` ne détruit pas les commits, il **détache** la branche d'eux. Ils restent dans le dépôt (référencés par le reflog, ~90 jours) tant qu'on ne lance pas le garbage collector. Le seul travail **irrécupérable** est celui **jamais commité**.

## Scénario C — le commit partagé à annuler

```bash
echo "exit 1" >> diagnostic.sh        # le "bug" : le script s'arrête avant la fin
git add diagnostic.sh && git commit -m "feat: ajout du bloc reseau (bug)"
git push

./diagnostic.sh          # échoue : "bloc reseau" puis arrêt

git revert HEAD          # Git crée le commit inverse (message auto proposé)
./diagnostic.sh          # refonctionne ✔
git push
```

**Pourquoi `revert` et pas `reset --hard` + force push ?** Le commit bug est **poussé et visible de l'équipe**. `revert` l'annule par un **nouveau commit** : l'historique raconte la vérité (« on a ajouté, puis retiré »), personne ne perd de travail, aucun force push. C'est l'annulation standard en entreprise.

## Partie 2 — le projet récapitulatif, point par point

```bash
# 1) Partir d'une main à jour (réflexe Leçon 3-4)
git switch main && git fetch origin && git pull --rebase
git switch -c feature/check-reseau-final

# 2) Commits atomiques (exemple de bloc ping)
cat >> diagnostic.sh << 'EOF'
echo "Reseau :"
ping -c 1 -W 2 8.8.8.8 > /dev/null 2>&1 && echo "Internet : OK" || echo "Internet : KO"
EOF
git add diagnostic.sh && git commit -m "feat: verification de la connectivite reseau"

# 3) Nettoyage si besoin
git rebase -i main        # squash des micro-commits en un commit propre

# 4-5) Push + PR (description Quoi/Pourquoi/Comment tester)
git push -u origin feature/check-reseau-final
# → GitHub : "Compare & pull request"

# 6) Self-review puis Merge pull request (interface)

# 7) Resynchronisation locale
git switch main && git pull --rebase
git branch -d feature/check-reseau-final
```

**Vérification finale** : `git log --oneline --graph -10` montre l'historique propre (merge de la PR, pas de branche orpheline) ; `./diagnostic.sh` affiche le bloc réseau.

**Évaluation du critère du bloc** : si tu as enchaîné ces 8 étapes **sans relire les leçons**, le critère « bloc acquis » de la roadmap est atteint : *tu peux travailler sur un repo proprement, sans casser l'historique ni avoir peur des conflits*.

---

## ✅ Checklist de validation (réécrite)

- [ ] Je sais déplacer un commit de `main` vers une branche (`switch -c` puis `reset --hard HEAD~1` sur main) **sans le perdre**.
- [ ] Je sais restaurer des commits après un `reset --hard` grâce à `git reflog` + `git reset --hard <hash>`.
- [ ] Je sais annuler un commit **poussé** avec `git revert` et j'explique pourquoi c'est la seule option sûre sur une branche partagée.
- [ ] Je connais les 3 modes de `reset` et le piège du travail non commité (irrécupérable).
- [ ] J'ai réalisé seul le **workflow complet** : branche → commits conventionnels → rebase propre → push → PR structurée → review → merge → resync.
- [ ] J'ai documenté mes réparations (bonus `REPARATIONS.md`) — le réflexe DevOps de tracer.

## 🧭 Conseils

- **Réflexe de survie** : avant toute manipulation risquée, `git status` + un commit ou stash de sécurité. Après toute frayeur, `git reflog` : c'est ta boîte noire.
- **Munition de vocabulaire** : amend, revert, reset (soft/mixed/hard), reflog, commit orphelin, garbage collector, force push, branch protection.
- **Fin du Bloc 04** : tu sais versionner, brancher, pousser, collaborer via PR, résoudre des conflits et réparer. C'est exactement le socle que les **Blocs 11 (CI/CD)** et **13 (GitOps)** supposent acquis : tes prochains « commit + push » déclencheront des pipelines entiers.

---

*Prochaine étape :* Relis la checklist globale dans **`00-Introduction-Bloc.md`**, puis poursuis la roadmap avec le **Bloc 05 — Réseautage et sécurité**.
