# Référence rapide — Leçon 4 : Stockage objet (S3)

> Bloc 6 · Leçon 4 — Aide-mémoire.

## Concepts
- **S3** = Simple Storage Service → stockage d'**objets** d'AWS.
- **Bucket** : conteneur, nom **globalement unique** en minuscules.
- **Objet** : fichier + métadonnées.
- **Clé (key)** : nom complet de l'objet (les `/` sont des conventions, pas des vrais dossiers).
- **URL** : `https://bucket.s3.region.amazonaws.com/ma-cle`
- **Privé par défaut** — le public est l'exception.

## Commandes AWS s3 (ou via MinIO avec `--endpoint-url`)
```bash
aws s3 mb s3://mon-bucket --region eu-west-3        # make bucket (créer)
aws s3 cp fichier.jpg s3://mon-bucket/chemin/       # copier vers S3
aws s3 ls s3://mon-bucket                           # lister
aws s3 cp s3://mon-bucket/chemin/fichier.jpg .      # télécharger
aws s3 sync ./local/ s3://mon-bucket/ --delete      # synchroniser
aws s3 sync ./local/ s3://mon-bucket/ --dry-run     # simuler sans rien faire
aws s3 rm s3://mon-bucket/ --recursive              # vider
```

## Permissions & bonnes pratiques
- Priver par défaut, ouvrir en public seulement si voulu.
- Versioning (historique) sur les buckets critiques.
- Lifecycle (ex. purge après 30 jours).
- Jamais de secrets dans S3 (→ coffres, Leçon 6).
- Accès application via IAM (Leçon 6), pas une clé publique.

## Cas d'usage
Avatars, factures, PDF, backups, frontend statique (Angular), logs, exports.

## MinIO (local, gratuit)
```bash
wget https://dl.min.io/server/minio/release/linux-amd64/minio
chmod +x minio
MINIO_ROOT_USER=minioadmin MINIO_ROOT_PASSWORD=minioadmin ./minio server ./data --console-address ":9001"
aws --endpoint-url http://localhost:9000 s3 mb s3://mon-bucket
```
C'est le **même S3**, mais chez toi, pour t'entraîner sans dépenser.