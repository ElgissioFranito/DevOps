# Exercice — Leçon 7 : Serverless (Lambda)

> **Bloc 6 · Leçon 7** — Exercice en autonomie. **Rien de coûteux ne sera lancé** : travail sur schéma et décision (pas de création de fonction AWS).

---

## Contexte

Ton application web (frontend Angular + backend Spring Boot) a plusieurs **petites tâches** : créer une miniature quand un utilisateur téléverse son avatar, générer un PDF de facture, nettoyer les vieux backups sur S3. Tu dois choisir le bon outil et comprendre le déclenchement.

---

## Énoncé

### Étape 1 — Schéma `événement → Lambda → résultat` (dans `notes-exercice-07.md`)

Pour **chacune** de ces 3 tâches, écris :
1. Le **déclencheur** (l'événement),
2. Ce que fait la **fonction**,
3. Le **résultat**.

- Tâche A : redimensionner un avatar à l'upload.
- Tâche B : générer un PDF de facture à la demande.
- Tâche C : purger les backups de plus de 30 jours chaque nuit.

### Étape 2 — Lambda ou EC2 ? (dans `notes-exercice-07.md`)

Pour chaque tâche : **Lambda (L)** ou **EC2 (E)** ? Justifie en **1 phrase** (pense : court/intermittent vs long/permanent).

### Étape 3 — Comparaison (tableau dans `notes-exercice-07.md`)

Résume en tableau : **Lambda vs EC2** sur 5 critères (travail, serveur à gérer, facturation, mise à l'échelle, limites).

### Étape 4 — Plan anti-piège (dans `notes-exercice-07.md`)

Liste **3 règles** pour éviter les pires pièges Lambda (pense : idempotence, durée, IAM, logs).

---

## Livrable

`notes-exercice-07.md` (3 schémas + décisions + tableau + 3 règles).

Correction détaillée dans **`03-correction.md`**.