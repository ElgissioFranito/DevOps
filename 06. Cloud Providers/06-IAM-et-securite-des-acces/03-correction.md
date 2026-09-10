# Correction — Leçon 6 : IAM et sécurité des accès

> **Bloc 6 · Leçon 6** — Correction pas à pas.

---

## Étape 1 — 3 questions IAM (réponse type)

- **Qui ?** : l'application Spring Boot (lire les avatars), le développeur (gérer le bucket dev), l'admin infra (gérer les VM).
- **Peut faire quoi ?** : l'app → `s3:GetObject` sur le bucket des avatars ; le dev → gestion du bucket **dev** uniquement ; l'admin → gestion des **VM** (et seulement ça).
- **Sur quelle ressource ?** : le bucket `app-avatars-prod/avatars/*` (app), le bucket dev (dev), les instances du projet (admin).

## Étape 2 — 4 briques IAM

- **Utilisateur** : identité d'une personne avec ses droits.
- **Groupe** : ensemble d'utilisateurs avec les mêmes droits (ex. `devs`).
- **Rôle** : identité temporaire pour une machine/application.
- **Politique** : document JSON décrivant les autorisations/refus.

## Étape 3 — Mini-politique JSON

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::app-avatars-prod/avatars/*"
    }
  ]
}
```
**Explication** : `Effect: Allow` = autoriser ; `Action: s3:GetObject` = l'action « lire un objet » ; `Resource: arn:aws:s3:::app-avatars-prod/avatars/*` = « uniquement le contenu du dossier avatars ». C'est du **moindre privilège** : ni liste complète du bucket, ni écriture, ni autre bucket.

## Étape 4 — Plan anti-fuite (3 règles)

1. **`.gitignore`** contient `.aws/`, `credentials`, `*.pem` → les clés ne partent jamais dans Git (et jamais dans l'historique !).
2. **`chmod 700 ~/.aws`** (et `chmod 600` sur les clés privées) → seul ton compte lit les secrets.
3. **Jamais de clé dans le code/script** → on passe par `aws configure`, des variables d'environnement, ou un coffre/secret manager (Bloc 5).

## Étape 5 — Pratique CLI

- **Avec compte** : `aws configure` remplit `~/.aws/credentials` ; `aws sts get-caller-identity` répond avec ton ARN d'utilisateur (ex. `arn:aws:iam::123456789012:user/mon-user`) — c'est la preuve que la CLI sait **qui** tu es.
- **Sans compte** : les vraies commandes refusent (« InvalidClientTokenId » ou « The security token included in the request is invalid ») — c'est **normal et souhaitable** : cela confirme que la sécurité fonctionne (identité inconnue → refus).

---

## Checklist de validation (leçon 6)

- [ ] Je réponds aux 3 questions IAM.
- [ ] Je distingue utilisateurs, groupes, rôles, politiques.
- [ ] J'explique le moindre privilège et le RBAC (Bloc 5).
- [ ] Je configure l'AWS CLI (`aws configure`) et vérifie avec `aws sts get-caller-identity`.
- [ ] J'ai le réflexe anti-fuite : `.gitignore`, `chmod`, jamais de clé dans Git.
- [ ] Je ne donne pas `AdministratorAccess` à la légère.

---

## 🧠 Conseils pour la suite

- **MFA** (application d'authentification) sur root + admins : la norme 2025-2026.
- **Rôles pour les machines** : une EC2 prend son rôle, pas une clé embarquée.
- **Rotation des clés** : renouveler régulièrement et retirer / désactiver les clés inutilisées.
- Prochaine étape : le **serverless Lambda** — quand tu ne veux pas gérer de serveur du tout (Leçon 7).