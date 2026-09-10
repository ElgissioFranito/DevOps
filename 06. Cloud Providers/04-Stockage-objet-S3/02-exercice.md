# Exercice — Leçon 4 : Stockage objet (S3)

> **Bloc 6 · Leçon 4** — Exercice en autonomie. Rien de coûteux : script local, et au choix MinIO en local (gratuit) pour tester les vraies commandes `aws s3`.

---

## Contexte

Ton application web (frontend Angular + backend Spring Boot) doit stocker : les **avatars** des utilisateurs, les **PDF de factures**, et les **backups de la base** (qui viendra à la Leçon 5). Tu dois décider **où** et **comment** dans S3.

---

## Énoncé

### Étape 1 — Script `simuler-s3.sh`

Crée puis exécute le script d'affichage de la Section 3.3 :

```bash
nano simuler-s3.sh
bash simuler-s3.sh
```

Colle la sortie dans `notes-exercice-04.md`.

### Étape 2 — Concevoir l'organisation (dans `notes-exercice-04.md`)

Pour les **3 types de fichiers** (avatars, factures, backups), propose :
- un **nom de bucket** (minuscules, unique),
- une **clé** (chemin logique),
- **public ou privé ?** + pourquoi.

> 📌 Rappel : les `/` ne sont que des conventions de nommage ; l'objet = le nom complet.

### Étape 3 — Mini-politique de sécurité (dans `notes-exercice-04.md`)

Rédige 5 règles pour sécuriser tes buckets (pense : privé par défaut, versioning, lifecycle, pas de secrets, accès par l'application via IAM).

### Étape 4 — (Optionnel, recommandé) Tester avec MinIO en local

Si tu te sens prêt : installe MinIO (Section 3.2) et exécute en local :
```bash
aws --endpoint-url http://localhost:9000 s3 mb s3://mon-premier-bucket
aws --endpoint-url http://localhost:9000 s3 cp photo.jpg s3://mon-premier-bucket/photo.jpg
aws --endpoint-url http://localhost:9000 s3 ls s3://mon-premier-bucket
```
Colle les résultats dans `notes-exercice-04.md`.

---

## Livrable

`notes-exercice-04.md` (sortie du script + organisation + politique + éventuels résultats MinIO).

Correction détaillée dans **`03-correction.md`**.