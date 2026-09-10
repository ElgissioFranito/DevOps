# Exercice — Leçon 6 : IAM et sécurité des accès

> **Bloc 6 · Leçon 6** — Exercice en autonomie. Certaines étapes exigent un **compte AWS** (création d'utilisateur IAM) ; sinon, on **simule** le fichier factice (Section 3.2 de la leçon). Rien de coûteux ne sera lancé.

---

## Contexte

Ton application a besoin de droits précis : l'application **lit les avatars** depuis le bucket S3, un **développeur** gère le bucket dev, un **admin d'infra** gère les VM. Tu dois organiser tout ça avec IAM et **sans jamais fuiter de clé**.

---

## Énoncé

### Étape 1 — Répondre aux 3 questions IAM (dans `notes-exercice-06.md`)

Pour ton projet, complète :
- **Qui ?** (liste les identités : application, dev, admin…)
- **Peut faire quoi ?** (une action par identité)
- **Sur quelle ressource ?** (le bucket, la VM, la base…)

### Étape 2 — Les 4 briques IAM

En 1 phrase chacune : **utilisateur**, **groupe**, **rôle**, **politique**.

### Étape 3 — Écrire une mini-politique JSON (dans `notes-exercice-06.md`)

Écris une politique qui autorise **uniquement** `s3:GetObject` (lire) sur le bucket `app-avatars-prod`, chemin `avatars/*`. Réutilise la structure vue dans le cours (`Effect`, `Action`, `Resource`, ARN).

### Étape 4 — Le plan anti-fuite de clés (dans `notes-exercice-06.md`)

Écris 3 règles que tu appliqueras **dès aujourd'hui** (pense `.gitignore`, `chmod`, secrets).

### Étape 5 — Pratique CLI (selon ton compte)

- **Avec un compte AWS** : crée un utilisateur IAM admin (console), génère des clés, exécute `aws configure`, puis `aws sts get-caller-identity`.
- **Sans compte** : simule le fichier `~/.aws/credentials` factice (Fausses valeurs !), puis observe que les commandes réelles refusent — note bien le message d'erreur.

Colle tes sorties dans `notes-exercice-06.md`.

---

## Livrable

`notes-exercice-06.md` (réponses 3 questions + 4 briques + mini-politique + plan anti-fuite + sorties CLIs).

Correction détaillée dans **`03-correction.md`**.