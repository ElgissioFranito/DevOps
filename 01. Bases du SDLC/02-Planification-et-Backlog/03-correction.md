# Correction détaillée — Planification et backlog

> **Bloc 1 · Leçon 2** — Correction pas-à-pas de `02-exercice.md`. Nous corrigeons avec l'exemple du **panier d'achat**. Si tu as choisi autre chose, le raisonnement doit être identique.

---

## ✅ Étape 1 — Backlog produit (exemple complet)

```text
BACKLOG PRODUIT
├── [User Story] En tant que client, je veux ajouter un produit au panier afin de commander en une fois.
├── [User Story] En tant que client, je veux voir le total de mon panier afin de vérifier ma commande.
├── [Tâche] Créer la table panier_lignes en base de données (produit, quantité, prix).
├── [Bug] Le total n'est plus à jour après suppression d'une ligne.
└── [User Story] En tant que client, je veux supprimer une ligne de mon panier afin de corriger ma commande.
```

**Pourquoi c'est une bonne base** : 3 User Stories, 1 tâche technique (invisible pour l'utilisateur), 1 bug précis et vérifiable. Les User Stories sont au bon format et portent chacune une valeur utilisateur distincte.

---

## ✅ Étape 2 — Priorisation MoSCoW justifiée

```text
[M] Ajouter un produit au panier          → cœur du panier : sans lui, rien ne peut se faire.
[M] Créer la table panier_lignes          → support technique indispensable à l'ajout.
[M] Voir le total du panier               → rassure l'utilisateur, nécessaire à la commande.
[S] Corriger le bug du total après suppression → important pour la confiance, mais lié à la suppression.
[C] Supprimer une ligne du panier         → utile mais on peut livrer d'abord un panier minimal.
[W] (ex.) Historique détaillé des commandes → plus tard, pas nécessaire au démarrage.
```

**Pourquoi** : le [M] regroupe ce **sans quoi le produit ne marche pas** (ajouter + stocker + afficher le prix). Le [S] et [C] sont des enrichissements qu'on peut décaler. Le [W] est une idée gardée en tête mais reportée.

> 💡 **Tolérance** : la hiérarchie exacte M/S/C peut varier selon l'équipe. L'important est la **cohérence** entre la lettre et la justification. Si tu as classé « supprimer » en [M], c'est défendable — mais explique-le.

---

## ✅ Étape 3 — Estimation en points justifiée

```text
[2 points] Ajouter un produit au panier    → page simple + règle d'ajout.
[3 points] Créer la table panier_lignes    → model + migration + branchement.
[2 points] Voir le total du panier         → simple calcul de somme + affichage.
[2 points] Corriger le bug du total        → bug ciblé, cas connu.
[3 points] Supprimer une ligne             → action + mise à jour du total (liée au bug).
```

**Pourquoi des points et pas des heures ?** Les points sont **relatifs** : « supprimer » (3) ≈ « table » (3) ≈ même effort, « ajouter » (2) plus léger. On compare, on ne prévoit pas une date exacte.

---

## ✅ Étape 4 — Sprint 1 (exemple de choix)

```text
SPRINT 1 (1 semaine, objectif réaliste ~ 4 à 5 points)
[M] Créer la table panier_lignes          (3 pts)
[M] Ajouter un produit au panier          (2 pts)
→ TOTAL ≈ 5 points
```

**Pourquoi ce choix** :
- Il livre la **boucle minimale** : la fonctionnalité « ajouter au panier » fonctionne réellement.
- Le bug du total et la suppression attendent le sprint 2 (ils touchent le même code, mieux vaut les faire ensemble en une fois).
- Le nombre de points (~5) est **réalisable** pour un développeur seul, alors que tout le backlog (~12 points) ne l'est pas.

> 💡 Autres choix valables : mettre « voir le total » dès le sprint 1 est cohérent car c'est [M]. L'important est de ne **pas tout promettre** dans une semaine.

---

## ✅ Étape 5 — Auto-vérification (réponses types)

1. **Ma User Story a-t-elle les 3 parties ?** → Oui : acteur (client), action (ajouter/voir/supprimer), bénéfice (commander en une fois / vérifier / corriger).
2. **Mes [M] sont-ils vraiment indispensables ?** → Oui : ajouter, stocker et afficher le prix sont le minimum vital du panier. « Supprimer » ou « email de confirmation » peuvent être décalés.
3. **Mon sprint est-il réalisable ?** → Oui : ~5 points sur une semaine, c'est cohérent ; un sprint surchargé donne le contraire (qualité et délai cassés).

---

## 📝 Checklist de validation (récapitulatif + conseils)

- [ ] Je sais **écrire** une User Story complète au bon format.
- [ ] Je sais **distinguer** User Story, tâche, bug et épic.
- [ ] Je sais **prioriser** avec MoSCoW et justifier chaque lettre.
- [ ] Je sais **estimer** en points relatifs (1-2-3-5-8) avec une justification.
- [ ] Je sais **choisir un sprint réalisable** sans surcharger.
- [ ] J'ai obtenu, comme dans cette correction, un backlog cohérent, priorisé, estimé et découpé.

### 💡 Conseils pour la suite

- **Re-teste-toi** en refaisant l'exercice sur une autre fonctionnalité dans quelques jours.
- Si tu travailles avec un vrai outil, essaie de créer ce backlog dans **Jira, GitHub Projects ou Trello** (l'outil importe peu ; le raisonnement importe).
- Règle d'or pour la suite : **on ne code jamais un élément du backlog qui n'est pas écrit, priorisé et estimé.** Ça sera le socle de la Leçon 3 (tests & environnements) où tu dérouleras tes User Stories en tests.

---

*Prochaine étape :* Leçon 3 — **Tests et environnements** → dossier `03-Tests-et-Environnements/`.