# Leçon 4 — Stockage objet : S3

> **Bloc 6 · Cloud Providers** — Leçon 4 sur 8
> 🧭 **Pont depuis la Leçon 3** : ton application tourne sur une **VM (EC2)**, mais une VM peut être remplacée ou détruite… **où ranger les fichiers durables** (images, PDF, sauvegardes, fichiers statiques) ? Pas sur le disque de l'instance : dans un **stockage séparé, très robuste**, appelé **stockage d'objets**. Chez AWS, ce service s'appelle **S3**. C'est une brique que tu retrouveras partout (sauvegarde, hébergement de fichiers, CI/CD plus tard).

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Expliquer** ce qu'est le stockage d'objets et en quoi il diffère d'un disque classique de VM.
2. **Définir** bucket, objet, clé (nom), et les cas d'usage typiques (images, PDF, backups, fichiers statiques).
3. **Utiliser** les commandes S3 de base (`mb`, `cp`, `ls`, `sync`) avec des alternatives locales (MinIO) sans dépenser.
4. **Comprendre** et appliquer les **permissions** S3 (privé par défaut, jamais exposé par erreur).
5. **Connaître** le versioning, le cycle de vie (lifecycle) et les coûts de stockage (premier pas vers FinOps).

---

## 2. Explication simple

### 2.1 Le « pourquoi » : pourquoi un stockage séparé de la VM ?

Une VM (Leçon 3) est **éphémère** : on peut la détruire, la remplacer, l'autoscaler. Si tes **fichiers importants** (photos utilisateurs, PDF de factures, sauvegardes de base) vivaient **sur le disque de la VM**, ils **disparaîtraient** dès qu'on supprime l'instance. Il faut donc un endroit **de stockage séparé, très fiable**, auquel l'application accède **par le réseau**, et qui survit à toute panne de machine : c'est **S3**.

> 💡 **Analogie** : la VM, c'est ton **bureau** (tu y travailles, mais on peut te le changer) ; S3, c'est le **garde-meubles** où tu ranges durablement tes affaires, à l'abri, avec un inventaire (le « bucket »). On ne stocke pas les archives au bureau ; on les met au garde-meubles.

### 2.2 Le « comment » : objets, buckets, clés

**S3** = **Simple Storage Service** : le service de **stockage d'objets** d'AWS. « Stockage d'objets » = on stocke des **fichiers** (images, PDF, vidéo, sauvegardes, logs) avec un **nom unique** et des **métadonnées** (taille, date, type), sans hiérarchie de dossiers complexe.

Les concepts essentiels :

| Terme | C'est quoi ? | Analogue classique |
|-------|--------------|--------------------|
| **Bucket** | Un conteneur de rangement, au **nom globalement unique** (dans tout AWS) | Un **garde-meubles** |
| **Objet** | Un fichier stocké (+ ses métadonnées) | Une **boîte étiquetée** |
| **Clé (key)** | Le **nom complet** de l'objet à l'intérieur du bucket | L'**étiquette** sur la boîte |
| **URL** | L'adresse web de l'objet : `https://bucket.s3.region.amazonaws.com/ma-cle.jpg` | L'adresse de livraison du garde-meubles |

Exemple d'arborescence logique (les « dossiers » `/` ne sont qu'une **convention de nommage** — S3 ne connaît que des clés) :

```
bucket-mes-images/
├── photos/2026/profil.jpg      ← clé : "photos/2026/profil.jpg"
├── factures/facture-42.pdf
└── backups/bdd-2026-09-10.sql  ← sauvegarde de la base (bien au chaud !)
```

> 💡 **Important** : dans S3, il n'y a **pas de vrai dossier**. La **barre `/` fait partie du nom** (la clé). C'est une différence subtile mais importante pour l'utiliser sans étonnement.

### 2.3 Le « quand » : les cas d'usage principaux

| Cas d'usage | Exemple |
|-------------|---------|
| **Fichiers utilisateurs** | Images de profil, photos, avatars |
| **Documents** | PDF, factures, exports |
| **Sauvegardes (backups)** | Dump de base (Bloc 7), snapshots de VM (Leçon 3) |
| **Fichiers statiques** | Le frontend Angular compilé (servi par Nginx — Bloc 5) |
| **Logs et exports** | Journaux d'application centralisés |

**Quand S3 et pas une VM ?** Dès que le fichier doit **survivre** (durée dans le temps), être **partagé** entre machines, ou être **accessible par URL**. La VM, elle, **calcule** ; S3 **stocke**.

### 2.4 Le « comment » : permissions — privé par défaut

S3 applique le principe **moindre exposition** du Bloc 5 : un **bucket est privé par défaut**. Personne ne peut lire tes objets sauf si tu l'autorises explicitement. On distingue :
- **Lecture/écriture** par l'application via **IAM** (gestion des identités et permissions — Leçon 6) : l'application a le droit via ses identifiants, sans rendre le bucket public.
- **Accès public volontaire** : pour un site statique, un PDF public, une image à partager… On l'active **consciemment**, jamais « pour voir ».

> 🔑 **Règle d'or S3** : **privé par défaut, public seulement si nécessaire et voulu.** Un bucket public par erreur = **fuite de données** (les fichiers deviennent indexables par Google !).

---

## 📖 Vocabulaire / Abréviations

> Définitions d'une ligne pour ne jamais être perdu(e).

- **S3** (Simple Storage Service) : le service de stockage d'objets d'AWS.
- **Stockage d'objets** : des fichiers avec nom + métadonnées, accessibles par réseau/URL, très durables.
- **Bucket** : conteneur de stockage (nom unique dans AWS).
- **Objet** : un fichier + ses métadonnées dans S3.
- **Clé (key)** : le nom complet de l'objet dans le bucket (les `/` sont des conventions visuelles).
- **MinIO** : logiciel **open-source** qui simule S3 **en local** (mêmes commandes) pour s'entraîner sans AWS.
- **Versioning** : conserver les versions précédentes d'un objet quand il change ou est supprimé.
- **Lifecycle (cycle de vie)** : règle automatique (ex. supprimer les sauvegardes de plus de 30 jours) pour maîtriser volume/coûts.
- **IAM** : gestion des identités et permissions (détaillé à la Leçon 6).
- **URL** : adresse web d'un objet S3 (si publique).
- **AWS CLI / `aws s3`** : les commandes de l'outil `aws` pour piloter S3 (installé en Leçon 1).
- **`--dry-run`** : option « simule sans rien faire » — parfait pour apprendre sans risque.

---

## 3. Exemples concrets

> ⚠️ **Réalité pratique** : créer un bucket AWS réel exige un compte + clés (Leçon 6), et stocker coûte un peu. On montre donc les **commandes AWS réelles** ET une **alternative locale (MinIO ou un script)** pour tout expérimenter gratuitement.

### 3.1 Commandes AWS réelles (avec compte configuré)

```bash
# 1. Créer un bucket (nom globalement unique, minuscules). --region = zone géographique.
aws s3 mb s3://mon-premier-bucket --region eu-west-3

# 2. Copier un fichier local dans le bucket.
# s3://mon-premier-bucket/mon-dossier/ = la cible ("dossier" = convention de nom).
aws s3 cp photo-profil.jpg s3://mon-premier-bucket/photos/photo-profil.jpg

# 3. Lister les objets du bucket.
aws s3 ls s3://mon-premier-bucket

# 4. Télécharger un objet.
aws s3 cp s3://mon-premier-bucket/photos/photo-profil.jpg ./photo-local.jpg

# 5. Synchroniser un dossier local avec le bucket (très utile pour des backups).
# --delete = supprime côté S3 ce qui n'existe plus en local (à utiliser avec prudence !).
aws s3 sync ./mes-backups/ s3://mon-premier-bucket/backups/ --delete

# 6. Mode "--dry-run" : on affiche ce qui SE PASSERAIT, sans rien faire (exercice sûr).
aws s3 sync ./mes-backups/ s3://mon-premier-bucket/backups/ --dry-run

# 7. Vider un bucket (avant suppression éventuelle).
aws s3 rm s3://mon-premier-bucket/ --recursive
```

### 3.2 Alternative locale : MinIO (simule S3 gratuitement)

**MinIO** reproduit S3 sur ta machine (mêmes concepts de buckets/objets, mêmes commandes de l'outil) :

```bash
# (1) Installation (linux 64 bits) — une seule fois.
# wget télécharge ; chmod +x rend exécutable (Bloc 2).
wget https://dl.min.io/server/minio/release/linux-amd64/minio
chmod +x minio
# (2) Lancement local, port 9000 (données dans ./data).
MINIO_ROOT_USER=minioadmin MINIO_ROOT_PASSWORD=minioadmin ./minio server ./data --console-address ":9001"
```

Dans un **autre terminal**, tu peux utiliser l'**AWS CLI** pointée sur MinIO (via `--endpoint-url`) : les commandes `s3 mb/cp/ls` deviennent simulables chez toi !

```bash
# Créer un bucket EN LOCAL (mêmes commandes que sur AWS réel).
aws --endpoint-url http://localhost:9000 s3 mb s3://mon-premier-bucket
# Copier un fichier local.
aws --endpoint-url http://localhost:9000 s3 cp photo.jpg s3://mon-premier-bucket/photo.jpg
```

> 💡 Si MinIO te paraît trop à installer tout de suite : le script d'**affichage** (Section 3.3) suffit pour la logique ; reviens à MinIO quand tu voudras approfondir.

### 3.3 Script local d'entraînement (aucune installation)

```bash
# simuler-s3.sh — refait le "chemin" d'un objet S3 pour bien comprendre la clé.
echo "🧺 Bucket : mon-premier-bucket"
echo "  Objet   : photos/2026/profil.jpg   (la CLÉ contient des / mais ce ne sont pas des vrais dossiers)"
echo "  URL (si public) : https://mon-premier-bucket.s3.eu-west-3.amazonaws.com/photos/2026/profil.jpg"
echo ""
echo "🔒 Par défaut : PRIVÉ. On n'ouvre en public que si besoin et voulu."
```

---

## 4. Bonnes pratiques modernes (2025-2026)

- **Bucket privé par défaut** ; n'ouvrir en public que si vraiment nécessaire (et via une **politique explicite**, pas une option cliquée par hasard).
- **Versioning** sur les buckets importants : conserve l'historique des objets (anti-suppression accidentelle).
- **Lifecycle** : définir des règles (ex. garder 30 jours, puis archiver/supprimer) pour maîtriser le volume et la facture.
- **Ne pas stocker de secrets** dans S3 (clés, mots de passe) : secrets dans un coffre (Leçon 6, et Bloc 5).
- **Noms de bucket** en minuscules, sans accents (obligation AWS) — cohérent avec nos conventions de fichiers.
- **Backups automatiques** (ex. dump de base du Bloc 7 programmé vers S3) plutôt que manuels.

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi c'est dangereux/inefficace | ✅ Version correcte |
|----------------|--------------------------------------|---------------------|
| Bucket public « pour tester » | Les fichiers deviennent publics et indexables (fuite de données) | **Privé par défaut**, public seulement si exigé |
| Stocker les fichiers dans la VM (EC2) au lieu de S3 | Perte de tout à la suppression de l'instance | **S3 séparé**, l'application accède via l'API/CLI |
| Oublier le versioning | Un `cp` ou `rm` accidentel détruit la dernière version | **Versioning activé** pour les fichiers critiques |
| Jamais de lifecycle | Le volume et la facture grossissent sans fin | **Règles de cycle de vie** (purge automatique) |
| Mettre des mots de passe dans un fichier « public » de S3 | Fuite de secrets (Bloc 5 — secrets) | Secrets dans un coffre / secret manager (Leçon 6) |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`**, la correction dans **`03-correction.md`**.

**Énoncé court** : crée le script `simuler-s3.sh`, lance-le, puis rédige dans `notes-exercice-04.md` : les **cas d'usage** de S3 pour ton application (frontend Angular + backend), le **chemin d'un objet** (bucket, clé, URL), et une **mini-politique de sécurité** (privé par défaut, versioning, lifecycle). Si tu te sens motivé : installe MinIO et teste `s3 mb` / `cp` / `ls` en local en `--dry-run`.

---

## 7. Correction détaillée de l'exercice

> La correction complète est dans **`03-correction.md`**. On y valide : la logique bucket/clé/URL, l'architecture de stockage de ton app (frontend sur S3, backups sur S3, objets en privé), et la politique de sécurité.

---

## 8. Checklist de validation

- [ ] J'explique le stockage d'objets et sa différence avec un disque de VM.
- [ ] Je définis bucket, objet, clé, URL.
- [ ] Je liste les cas d'usage (images, PDF, backups, statiques).
- [ ] J'utilise `s3 mb/cp/ls/sync` (AWS CLI ou MinIO en local).
- [ ] Je sais que S3 est **privé par défaut** et pourquoi.
- [ ] J'explique versioning, lifecycle et premier lien avec les coûts (FinOps).

---

🧭 **Pont vers la suite** — Les **fichiers** ont leur maison robuste (S3). Mais les **données structurées** de l'application (les utilisateurs, les commandes) ? Elles vivent dans une **base de données**, et dans le cloud on préfère une base **managée** par AWS : **RDS** — la Leçon 5.

---

*Prochaine étape :* Leçon 5 — **Bases de données managées (RDS)** dans `05-Bases-de-donnees-managees-RDS/`.
