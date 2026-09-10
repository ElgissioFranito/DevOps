# Référence rapide — Leçon 6 : IAM et sécurité des accès

> Bloc 6 · Leçon 6 — Aide-mémoire.

## Les 3 questions IAM
```
Qui ?            → identité (personne ou machine)
Peut faire quoi ? → action (lire, écrire, supprimer…)
Sur quelle ressource ? → la cible (bucket, VM…)
```

## Les 4 briques
- **Utilisateur** : identité d'une personne.
- **Groupe** : ensemble d'utilisateurs avec les mêmes droits (ex. `devs`).
- **Rôle** : identité temporaire pour une machine/application.
- **Politique** : document JSON (Effect / Action / Resource).

## Exemple de politique (moindre privilège)
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

## Configuration CLI
```bash
aws configure                    # remplit ~/.aws/credentials
aws sts get-caller-identity      # affiche QUI je suis (ARN)
chmod 700 ~/.aws                 # protège le dossier des clés
```

## Règles d'or
- Compte **root** réservé aux opérations rares ; quotidien = **utilisateur IAM**.
- **Moindre privilège** : jamais `AdministratorAccess` à la légère.
- Clé **jamais** dans Git ; `.gitignore` + `.aws/` ; `chmod`.
- **MFA** sur root et admins (norme 2025-2026).
- **Rôles pour les machines** plutôt que clés embarquées.
- **Rotation** des clés ; désactiver les inutilisées.

## Distinction importante
- Clés **AWS** (IAM) ≠ credentials de la **base RDS** (Leçon 5).