# Exercice — Leçon 8 : FinOps et optimisation des coûts

> **Bloc 6 · Leçon 8** — Exercice en autonomie. **Aucun coût** : script local + réflexion (pas de création/coût AWS).

---

## Contexte

Ton architecture (frontend Angular + backend Spring Boot + PostgreSQL) vient d'être mise en production sur AWS. La facture arrive dans un mois et tu veux **éviter les mauvaises surprises**. Tu fais ton premier **audit FinOps**.

---

## Énoncé

### Étape 1 — Créer et lancer `audit-finops.sh`

Reprends le script de la Section 3.1 (les 5 questions de l'audit), exécute-le avec `bash audit-finops.sh`, et colle la sortie dans `notes-exercice-08.md`.

### Étape 2 — Les 4 postes de coût (dans `notes-exercice-08.md`)

Pour ton architecture, liste les **4 postes** (VM, stockage, réseau, base) en donnant pour chacun : **ce qui coûte** et **le réflexe d'optimisation**.

### Étape 3 — Les 3 gaspillages (dans `notes-exercice-08.md`)

Identifie les 3 gaspillages les plus probables chez un débutant, et pour chacun : **symptôme** + **action**.

### Étape 4 — Plan budget/alerte (dans `notes-exercice-08.md`)

Écris : le **seuil** de budget mensuel, les **2 alertes** (pourcentages), et la **réaction** à chaque alerte (que fais-tu ?).

---

## Livrable

`notes-exercice-08.md` (sortie du script + 4 postes + 3 gaspillages + plan budget/alerte).

Correction détaillée dans **`03-correction.md`**.