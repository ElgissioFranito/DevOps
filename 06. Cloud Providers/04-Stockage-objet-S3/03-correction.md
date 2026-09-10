# Correction — Leçon 4 : Stockage objet (S3)

> **Bloc 6 · Leçon 4** — Correction pas à pas.

---

## Étape 1 — Script simulé

`simuler-s3.sh` doit afficher le bucket, l'objet avec sa clé (avec `/` dans le nom, pas un vrai dossier) et l'URL. C'est une **maquette mentale** : elle te fait retenir « bucket + clé + URL ».

## Étape 2 — Organisation type

| Fichier | Bucket (proposé) | Clé | Public / privé ? |
|---------|------------------|-----|------------------|
| Avatars | `app-avatars-prod` | `avatars/utilisateur-42.jpg` | **Privé**, servi par l'application via des liens temporaires (ou un CDN) |
| Factures | `app-factures-prod` | `factures/2026-09/facture-0001.pdf` | **Privé** (documents confidentiels) |
| Backups de base | `app-backups-prod` | `backups/bdd-2026-09-10.sql` | **Privé** (jamais exposé !) |

**Explication** : les noms de bucket sont en minuscules (obligation AWS), faciles à identifier (`prod` vs `dev`). Tout est **privé** — sauf peut-être un hébergement de frontend statique dans un bucket dédié **pensé pour être public** (et souvent placé derrière un **CDN** — Content Delivery Network, « réseau de livraison de contenu » : des serveurs répartis dans le monde qui servent tes fichiers depuis l'endroit le plus proche du visiteur, pour aller plus vite). Les **liens temporaires** (presigned URLs chez AWS) sont des URL signées avec une date d'expiration : l'application les génère pour laisser un utilisateur télécharger **son** fichier, sans rendre le bucket public.

## Étape 3 — Mini-politique de sécurité (réponse type)

1. **Bucket privé par défaut** ; aucun accès public sauf si le cas est explicite.
2. **Versioning activé** sur `app-backups-prod` et `app-factures-prod` (anti-suppression accidentelle).
3. **Lifecycle** : backups conservés 30 jours puis purgés automatiquement (maîtrise du coût).
4. **Jamais de secrets** dans S3 (mots de passe, clés → coffre, Leçon 6).
5. **Accès application via IAM** (rôles et autorisations, Leçon 6) plutôt qu'une clé publique.

## Étape 4 — MinIO

`aws --endpoint-url http://localhost:9000 s3 mb …` crée un bucket **en local** : les mêmes commandes que sur AWS, mais **gratuitement**. `mb` = make bucket, `cp` = copie, `ls` = liste. L'URL local remplace l'URL AWS (mêmes concepts : bucket + clé + endpoint).

---

## Checklist de validation (leçon 4)

- [ ] J'explique le stockage d'objets vs un disque de VM.
- [ ] Je définis bucket, objet, clé, URL.
- [ ] Je liste les cas d'usage (avatars, factures, backups, statiques).
- [ ] J'utilise `s3 mb/cp/ls/sync` (AWS CLI ou MinIO local).
- [ ] Je sais que S3 est privé par défaut et pourquoi.
- [ ] J'explique versioning, lifecycle et le lien avec les coûts.

---

## 🧠 Conseils pour la suite

- Un bucket **public par erreur** est un incident de sécurité : vérifie toujours ton paramétrage.
- Les backups vont **vers S3** ; la base de données elle-même se gère à la Leçon 5 (RDS).
- MinIO te permet de t'entraîner en local sans compte : c'est un excellent réflexe d'autodidacte.