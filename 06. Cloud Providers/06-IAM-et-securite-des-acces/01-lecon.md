# Leçon 6 — IAM et sécurité des accès

> **Bloc 6 · Cloud Providers** — Leçon 6 sur 8
> 🧭 **Pont depuis les Leçons 1-5** : tu as vu les briques (VM, S3, RDS) et la règle « privé par défaut ». Il manque la pièce qui **contrôle qui peut faire quoi** sur tout ça : **IAM** (Identity and Access Management, « gestion des identités et des accès »). C'est **LA** brique de sécurité transversale — celle qui prolonge le RBAC vu au Bloc 5 et que tu retrouveras dans tous les clouds. Et c'est **ici** que tu créerás enfin tes **clés d'accès AWS** pour piloter la ligne de commande (promesse des Leçons 1-5).

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Définir** IAM et répondre aux trois questions : **qui** peut faire **quoi** sur **quelle ressource**.
2. **Distinguer** utilisateurs, groupes, rôles, politiques — et le lien avec RBAC (Bloc 5).
3. **Appliquer le moindre privilège** : jamais d'admin général pour une simple application.
4. **Créer** un utilisateur + des clés d'accès, et configurer l'AWS CLI (`aws configure`).
5. **Éviter** les pièges : clé dans Git, clé root partagée, politique trop large.

---

## 2. Explication simple

### 2.1 Le « pourquoi » : sans contrôle d'accès, tout est exposé

Une infrastructure cloud, c'est un **immeuble plein de ressources** (serveurs, stockage, bases). **N'importe quelle personne ou application** qui a les clés peut tout faire — y compris tout **détruire** ou faire **exploser la facture**. Il faut donc un système qui réponde à **trois questions** (la définition même d'IAM) :

```
Qui ?                  → l'identité (une personne, une application, une machine)
Peut faire quoi ?      → l'action (lire, écrire, supprimer, lister…)
Sur quelle ressource ? → la cible (ce bucket-là, cette VM-là, tout le compte ?)
```

> 💡 **Analogie** : dans un immeuble d'entreprises :
> - Le **badge** dit **qui** tu es (identité).
> - Le **droit du badge** dit ce que tu peux ouvrir (**quoi**).
> - La **porte** à laquelle le badge donne accès est la **ressource**.
> Tu n'as pas un badge « tout-puissant » ; tu as un badge qui ouvre **juste les portes nécessaires** à ton travail.

**Pourquoi c'est vital en DevOps ?** Parce qu'un pipeline CI/CD ou une application Spring Boot **doit** accéder à des ressources (lire un bucket, contacter la base) **sans** détenir les clés d'un administrateur général. Bien faire l'IAM limite les dégâts si une clé se fait voler.

### 2.2 Le « comment » : les 4 briques d'IAM

IAM s'appuie sur **4 concepts** :

| Concept | C'est quoi ? | Analogie immeuble |
|---------|--------------|-------------------|
| **Utilisateur IAM** | Une identité **pour une personne** (ou un poste) | Le badge personnel |
| **Groupe IAM** | Un ensemble d'utilisateurs partageant les **mêmes droits** (ex. `devs`, `admins`) | Le badge « catégorie » : tous les devs ont le même étage |
| **Rôle IAM** | Une identité **pour une machine/application** (pas une personne) qui prend des droits temporaires | Un badge automatique donné à une machine |
| **Politique (policy)** | Le **document** qui décrit ce qui est autorisé (et/ou refusé), au format JSON | Le **règlement d'accès** gravé dans le badge |

La **politique** est le cœur : elle se lit comme une phrase **« qui peut faire quoi sur quoi »**.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::mon-bucket/avatars/*"
    }
  ]
}
```

> (On ne te demande pas de réciter ça ; on t'y prépare : `Effect` = autoriser ou refuser, `Action` = la chose, `Resource` = la cible. `arn:aws:s3:::mon-bucket/avatars/*` = « le contenu du dossier avatars de mon bucket » — une **ARN** est l'adresse formelle d'une ressource AWS.)

### 2.3 Le « comment » (suite) : RBAC et moindre privilège

Tu as vu **RBAC** (Role-Based Access Control, contrôle d'accès par rôles) au Bloc 5 : on crée des **rôles** (admin, lecteur, développeur) avec des permissions fixes, et on y attache les utilisateurs. IAM fait exactement ça : les **politiques** sont attachées à des groupes/rôles, et les utilisateurs rejoignent les groupes. Résultat : tu ne donnes **jamais** une permission individuelle au hasard ; tu définis des **profils** (groupes) et tu y ranges les gens.

**Le moindre privilège** (principe vu au Bloc 5) : chaque identité n'a que les droits **nécessaires** pour son travail — pas un de plus.
- Un développeur : lecture/écriture sur son bucket de dev, **pas** la suppression en production.
- Une application : droit de **lire** ses avatars (S3 `GetObject`), pas de créer une VM.
- Un admin d'infra : droit d'admin, mais **compte à part** (jamais la clé root du compte).

> 🔑 **Le piège classique** : donner `AdministratorAccess` (admin complet) à une application ou à tous les devs « pour aller vite ». C'est le **premier vecteur de faille** : un seul vol de clé et tout le compte est compromis.

### 2.4 Le « quand » et la pratique : comptes, clés, région

**Deux types de « compte »** à ne pas confondre :
1. **Compte root AWS** (email + mot de passe de création du compte) : l'administrateur ultime. **Pas de clés quotidiennes** ; on crée un **utilisateur IAM** pour le quotidien.
2. **Utilisateur IAM** : les identités (personnes / apps / machines), chacune avec ses droits et ses **clés d'accès**.

**Clés d'accès AWS** = le duo sécurisé qui permet à la CLI de dire « je suis untel » :
- **Access Key ID** (publique, ex. `AKIA...`) — l'identifiant.
- **Secret Access Key** (secrète, affichée **une seule fois** à la création) — la preuve.

On les range dans `~/.aws/credentials` via `aws configure` (Leçon 1). **On ne les commit jamais.** On choisit aussi **une région** (ex. `eu-west-3`) : chaque région est indépendante.

### 2.5 Pour l'apprentissage : l'environnement « sandbox » local

⚠️ **Réalité** : pour créer un **vrai utilisateur IAM** il faut un **compte AWS** (création gratuite, carte bancaire demandée pour l'identité, mais tu peux rester dans les limites gratuites). Si tu ne peux pas encore t'inscrire, tu peux **simuler** la configuration en local (fichier `~/.aws/credentials` factice) pour **comprendre** le mécanisme — la création de vraies clés se fera le jour où tu auras ton compte. La leçon reste 100 % utile : le vocabulaire et les principes sont universels.

---

## 📖 Vocabulaire / Abréviations

> Définitions d'une ligne pour ne jamais être perdu(e).

- **IAM** (Identity and Access Management) : le service qui gère **qui peut faire quoi sur quoi** dans le cloud.
- **Identité** : une personne (utilisateur) ou une machine/application (rôle) qui fait des actions.
- **Utilisateur IAM** : identité pour une personne (ou un poste), avec ses droits et ses clés.
- **Groupe IAM** : ensemble d'utilisateurs avec les mêmes permissions (ex. `devs`, `admins`).
- **Rôle IAM** : identité temporaire **pour une machine/application** (prend des droits sans clés persistantes).
- **Politique (policy)** : document JSON décrivant les autorisations/refus.
- **Effect** : « Allow » (autoriser) ou « Deny » (refuser) dans une politique.
- **Action** : l'opération autorisée (ex. `s3:GetObject`, `ec2:*`).
- **Resource / ARN** : la cible (ex. `arn:aws:s3:::mon-bucket/avatars/*`). **ARN** = Amazon Resource Name, l'adresse formelle d'une ressource AWS.
- **Clés d'accès (Access Key ID + Secret Access Key)** : le duo qui identifie un utilisateur auprès de la CLI AWS.
- **Compte root** : le compte principal de création AWS (par mail) — à utiliser le moins possible.
- **Région** : zone géographique AWS (ex. `eu-west-3` = Paris) — indépendante des autres.
- **Moindre privilège** : n'accorder que les droits strictement nécessaires.
- **RBAC** : contrôle d'accès par rôles (Bloc 5) — le modèle utilisé par IAM.
- **`~/.aws/credentials`** : le fichier local où l'AWS CLI range tes clés (à protéger !).
- **`aws configure`** : l'assistant qui remplit ce fichier (vu en Leçon 1).

---

## 3. Exemples concrets

### 3.1 La configuration sécurisée (réelle ou simulée)

```bash
# 1. L'assistant de configuration : il écrit dans ~/.aws/credentials et ~/.aws/config.
#    AWS Access Key ID / AWS Secret Access Key / région / format de sortie.
aws configure
# → exemple de saisie :
#   AWS Access Key ID [None]: AKIA1234567890ABC
#   AWS Secret Access Key [None]: (coller la clé secrète — affichée une seule fois à la création)
#   Default region name [None]: eu-west-3
#   Default output format [None]: json
```

```bash
# 2. Vérifier QUI je suis (l'utilisateur IAM actif).
#    (Avec des clés factices, AWS répondra "erreur d'autorisation" — c'est NORMAL : le mécanisme est compris.)
aws sts get-caller-identity
```

```bash
# 3. Protéger le dossier .aws : seuls ton compte et toi peuvent le lire (Bloc 2, permissions).
chmod 700 ~/.aws
```

> 🔑 **Le réflexe sécurité n°1** : regarder `~/.aws/credentials` une fois pour voir son contenu, puis **ne jamais** le committer. Vérifie que ton `.gitignore` contient bien `.aws/` et `credentials`.

### 3.2 Pour tester la logique SANS compte : simulation locale

Crée un **fichier de credentials factice** pour t'habituer au format (uniquement de FAUSSES valeurs) :

```bash
mkdir -p ~/.aws
nano ~/.aws/credentials
# Coller (FAUSSES clés de démonstration — ne jamais utiliser en vrai) :
# [default]
# aws_access_key_id = AKIAFAKE2026
# aws_secret_access_key = fakeSecretKeyDeDemonstration000
```

`nano` crée le fichier (Bloc 2/3). Les vraies commandes AWS échoueront (identité inconnue), mais tu auras **compris** le mécanisme. Avec un vrai compte, tu répéteras les mêmes gestes avec de **vraies** clés.

### 3.3 Le « pourquoi » des rôles pour les machines (aperçu)

```bash
# Quand une application EC2 doit lire un bucket S3, on lui attache un RÔLE IAM (pas une clé embarquée).
# Schéma conceptuel :
#   Instance EC2 --(rôle attribué)--> Politique "s3:GetObject" sur le bucket
# L'application n'a plus AUCUNE clé secrète embarquée — AWS gère l'identité à sa place.
```

---

## 4. Bonnes pratiques modernes (2025-2026)

- **MFA (multi-factor authentication, authentification multi-facteurs)** sur le compte root ET les comptes admins : un code supplémentaire (application mobile) exigé à la connexion — norme de sécurité 2025-2026.
- **Compte root réservé** aux opérations rares ; tout le reste passe par des **utilisateurs IAM**.
- **Moindre privilège** systématique : politique la plus fine possible, jamais `*` si on peut l'éviter.
- **Rôles pour les machines** plutôt que clés embarquées dans l'application.
- **Rotation des clés** : renouveler régulièrement et désactiver celles qui ne servent plus.
- **Auditer** avec `IAM Access Analyzer` (détection d'accès trop ouverts).
- **Never commit** : `.aws/`, clés, secrets → toujours en `.gitignore`.

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi c'est dangereux/inefficace | ✅ Version correcte |
|----------------|-------------------------------------|---------------------|
| Committer `~/.aws/credentials` ou une clé dans le code | Vol de clé = contrôle total du compte | **Jamais** ; `.gitignore` ; clé dans un coffre (Bloc 5) |
| Donner `AdministratorAccess` à tous « pour aller vite » | Un vol de clé expose tout | Groupes avec **moindre privilège** |
| Utiliser la clé root au quotidien | La clé maîtresse est exposée inutilement | **Utilisateur IAM** avec un compte régulier |
| Partager une clé d'application dans une doc/tchat | Fuite d'identifiants (Bloc 5) | **Rôles IAM** pour les machines |
| Écrire les clés dans un script versionné | Fuite immédiate (Git historique !) | Variables d'environnement + secret manager |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`**, la correction dans **`03-correction.md`**.

**Énoncé court** : rédige dans `notes-exercice-06.md` : les réponses aux 3 questions d'IAM appliquées à ton projet, la liste des **4 briques IAM** avec une phrase chacune, une **mini-politique** (Effect/Action/Resource) pour « lire uniquement les avatars », et le plan du réflexe anti-fuite de clés (3 étapes). Si tu as un compte : crée un utilisateur IAM, configure `aws configure` et vérifie avec `aws sts get-caller-identity` (sinon, simule le fichier factice).

---

## 7. Correction détaillée de l'exercice

> La correction complète est dans **`03-correction.md`**. On y détaille : la politique type en JSON, le plan d'anti-fuite, et la procédure de création d'utilisateur IAM + `aws configure`.

---

## 8. Checklist de validation

- [ ] Je réponds aux 3 questions IAM (qui / quoi / sur quelle ressource).
- [ ] Je distingue utilisateurs, groupes, rôles, politiques.
- [ ] J'explique le moindre privilège et le lien avec RBAC (Bloc 5).
- [ ] Je configure l'AWS CLI avec des clés (`aws configure`) et vérifie l'identité.
- [ ] J'ai le réflexe anti-fuite : clés jamais dans Git, `.gitignore`, `chmod 700 ~/.aws`.
- [ ] Je sais NE PAS donner `AdministratorAccess` à la légère.

---

🧭 **Pont vers la suite** — Tu contrôles **qui fait quoi** et tu as tes **clés**. Mais faut-il louer une VM pour chaque petite tâche ? Il y a plus malin pour du code « à la demande sans serveur » : le **serverless Lambda**, la Leçon 7.

---

*Prochaine étape :* Leçon 7 — **Serverless (Lambda)** dans `07-Serverless-Lambda/`.
