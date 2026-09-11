# Référence rapide — Leçon 6 : Contrôle d'accès & secrets

> Bloc 5 · Leçon 6 — Aide-mémoire.

## Concepts
- **Authentification** : qui es-tu ? (identité)
- **Autorisation** : que peux-tu faire ? (droits)
- **RBAC** : droits par rôles fixes (simple, par défaut).
- **ABAC** : droits par attributs + règles conditionnelles (fin mais complexe).
- **Moindre privilège** : ne donner que le nécessaire.

## Secrets — règles d'or
- **Jamais** de secret dans Git / dans le code.
- `.gitignore` AVANT de committer (`.env`, `*.key`, `*.pem`).
- Variable d'environnement ou coffre (Vault / AWS / K8s).
- Rotation si compromis ; ne jamais logger un secret.

## Commandes
| Besoin | Commande | Note |
|--------|----------|------|
| Ignorer un fichier | `echo ".env" >> .gitignore` | |
| Vérifier l'ignorance | `git status` / `git check-ignore -v .env` | |
| Charger un `.env` | `set -a && source .env && set +a` | `-a` = exporter |
| Passer un secret | `DB_PASSWORD=xxx python3 script.py` | env à l'éxécution |

## Modèles (choix rapide)
- Des rôles suffisent → **RBAC**.
- Règles fines (dépt, horaires, niveau) → **ABAC**.