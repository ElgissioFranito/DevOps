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

---

## 🎯 Défi bonus (pour aller plus loin)

1. **Jeu des 7 différences** : compare les coûts d'une VM `t3.small` allumée 24/7 vs **allumée 12h/jour** (arrêt la nuit). Estime en % l'économie, puis explique pourquoi `stopped` ne coûte que le disque (rappel Leçon 3 : états running/stopped/terminated).
2. **Piège des données sortantes** : ton site sert 100 Go/mois depuis S3 vers Internet. Cherche ce qu'est le **trafic sortant (data transfer out)** et pourquoi il est souvent le poste surprise d'une facture AWS.
3. **Décision** : ton équipe propose de remplacer la VM de dev par une instance **2× plus grosse** « pour aller plus vite ». Quel argument FinOps utilises-tu pour demander une mesure AVANT de payer ? (indice : profil d'utilisation, heures actives, FinOps = informer → optimiser → opérer)