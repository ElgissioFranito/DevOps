# Leçon 7 — Serverless : Lambda

> **Bloc 6 · Cloud Providers** — Leçon 7 sur 8
> 🧭 **Pont depuis les Leçons 3-6** : tu sais créer des **VM (EC2)**, stocker des **objets (S3)**, gérer une **base (RDS)** et contrôler **les droits (IAM)**. Mais faut-il **toujours** louer une machine qui tourne 24h/24 pour exécuter une petite tâche ? Souvent non : il existe une façon de faire tourner du **code** sans gérer le moindre serveur. C'est le **serverless** — et chez AWS, le service phare s'appelle **Lambda**. On va comprendre le « pourquoi, comment, quand » et le comparer à EC2.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Expliquer** le concept de « serverless » (sans serveur à gérer) et le rôle de **Lambda**.
2. **Distinguer** Lambda (serverless) vs EC2 (VM) : quand utiliser l'un ou l'autre.
3. **Définir** les notions clés : fonction, événement (déclencheur), lancement à la demande, facturation à l'exécution, limites.
4. **Comprendre** le schéma `événement → Lambda → résultat` et des cas d'usage réels (redimensionner une image, traiter un upload).
5. **Connaître** les bonnes pratiques 2025-2026 (durée, droits IAM minimaux, idempotence, observabilité).

---

## 2. Explication simple

### 2.1 Le « pourquoi » : ne pas gérer de serveur quand on n'en a pas besoin

Avec une **VM (EC2)** : tu loues une machine, elle tourne **en continu**, tu paies **à l'heure**, et tu dois la **mettre à jour, surveiller, dimensionner**… Même si elle ne fait presque rien, elle coûte et s'entretient.

Or une grosse partie du travail d'une application, ce sont des **petites tâches** : redimensionner une image quand on la téléverse, envoyer une notification, générer un PDF, traiter un fichier déposé sur S3… Pour ça, une VM dédiée est un **gaspillage**.

> 💡 **Analogie** : une VM, c'est **louer un restaurant** (avec cuisinier, local et frigo) — même pour ne cuisiner qu'un seul plat par jour. Le **serverless**, c'est **appeler un livreur** à chaque commande : tu ne paies **que** la course, et tu n'entretiens **rien**. Le « livreur » (le cloud) possède et gère tout.

**Le principe central du serverless** : tu fournis uniquement **ton code**, le cloud s'occupe de **tout le reste** (serveur, mise à l'échelle, disponibilité). Le serveur existe, mais tu n'as **pas à le voir ni à le gérer** — d'où le nom.

### 2.2 Le « comment » : événement → fonction → résultat

Chez AWS, le service serverless de référence est **Lambda**. Tu écris une **fonction Lambda** (un petit morceau de code, ex. Python ou Node.js), et elle s'exécute **à chaque événement** qui la déclenche.

```
Événement (déclencheur)        Fonction Lambda         Résultat
  fichier déposé sur S3   →    "redimensionne"    →    image réduite + notification
  requête HTTP (API)      →    "génère le PDF"    →    fichier renvoyé
  minuteur (cron)         →    "nettoie les vieux logs" →  suppression
```

Les **déclencheurs (triggers)** les plus courants : un **upload sur S3** (Leçon 4), une requête **HTTP/API**, un **message** dans une file d'attente, une **horloge** (toutes les 10 min). À chaque événement, AWS **réveille** ta fonction, l'exécute, puis l'**éteint** quand c'est fini.

### 2.3 Le « quand » : Lambda ou EC2 ? (savoir choisir)

C'est LA question qu'un DevOps doit savoir trancher :

| Critère | **Lambda (serverless)** | **EC2 (VM)** |
|---------|--------------------------|--------------|
| **Type de travail** | Courts (secondes à quelques minutes), à la demande | Long, permanent, lourd |
| **Serveur à gérer** | Aucun (le cloud s'en occupe) | Toi (mises à jour, dimensionnement) |
| **Facturation** | **Par exécution et par durée** (0 € si rien ne tourne) | **À l'heure** (tourne en continu) |
| **Mise à l'échelle** | Automatique (milliers d'exécutions en parallèle) | Manuelle ou autoscaling |
| **Limites** | Durée max d'exécution (souvent 15 min max), mémoire bornée | Mémoire / disque / CPU quasi illimités selon le type |

> 🔑 **Règle pratique** : **court, intermittent, variable** → Lambda. **Long, permanent, exigeant** (ex. ton backend Spring Boot toujours actif, une base de données) → EC2 ou services managés. Beaucoup d'architectures **mélangent** : une app sur EC2 + des petites tâches serverless autour.

### 2.4 Le lien avec l'existant : IAM et droits

Une fonction Lambda a besoin de **droits** (par exemple lire le bucket S3, écrire dans un autre). Comme à la Leçon 6, on lui attache un **rôle IAM** avec une politique au **moindre privilège**. Jamais de clés embarquées : la fonction prend son rôle automatiquement.

### 2.5 Les limites à connaître (pour éviter l'étonnement)

- **Durée max** : une fonction Lambda s'arrête après une durée configurable (couramment 15 min) — pas de tâche infinie.
- **Mémoire** : bornée (de 128 Mo à quelques Go) ; le CPU suit la mémoire choisie.
- **« Cold start » (démarrage à froid)** : si la fonction n'a pas tourné depuis longtemps, AWS doit la « réveiller » → le premier appel peut être un peu plus lent. Bien maîtrisé en 2025-2026 avec des configurations adaptées.
- **Stateless** : la fonction n'a **pas d'état** conservé d'un appel à l'autre — pour garder des données, on utilise S3, une base, etc.

---

## 📖 Vocabulaire / Abréviations

> Définitions d'une ligne pour ne jamais être perdu(e).

- **Serverless** : exécuter du code **sans gérer de serveur** (le cloud gère tout, tu fournis le code).
- **Lambda** : le service serverless d'AWS (fonction à la demande).
- **Fonction Lambda** : un petit bout de code (Python, Node.js…) exécuté à chaque événement.
- **Événement / déclencheur (trigger)** : ce qui réveille la fonction (upload S3, requête HTTP, minuteur…).
- **Facturation à l'exécution** : on paie par appel et par durée d'exécution (0 € si la fonction ne tourne pas).
- **Stateless** : sans état interne conservé d'un appel à l'autre (les données vont dans S3 / une base).
- **Cold start** : léger délai au premier appel après une période d'inactivité.
- **Rôle IAM** : identité pour une machine/application (Leçon 6) — la façon d'autoriser une Lambda.
- **Trigger** : synonyme de déclencheur.
- **Endpoint HTTP / API** : une URL qui déclenche la fonction (souvent via API Gateway).

---

## 3. Exemples concrets

### 3.1 Une fonction Lambda simple (Python) + déclencheur S3

Voici l'idée d'une fonction qui **redimensionne une image** quand un fichier arrive sur S3. On ne te demande pas de tout retenir : c'est pour **voir** la forme d'une fonction Lambda (le code, l'événement reçu, ce que tu renvoies).

```python
# Lambda "resize_image" — déclenchée à chaque nouvel objet dans le bucket "uploads"
import json

def lambda_handler(event, context):
    # event = "paquet" que le déclencheur envoie : il décrit l'événement (quoi, où).
    print("Événement reçu :", json.dumps(event))   # on trace pour déboguer
    # Ici : lire l'image depuis S3, la redimensionner, la réécrire.
    return {
        "statusCode": 200,
        "body": json.dumps("Image traitée !")        # ce que renvoie la fonction
    }
```

> 📌 **`event`** = les infos reçues du déclencheur ; **`context`** = des infos sur l'exécution (durée restante…). Le **`print`** part dans les logs (observabilité — Bloc 12).

### 3.2 Le schéma de déclenchement (à mémoriser)

```
[ fichier téléversé dans le bucket S3 ]  ← événement / trigger
        ↓
[ Lambda : redimensionne + sauvegarde ]  ← ton code, avec un rôle IAM minimal
        ↓
[ nouvelle image dans le bucket "processed" + notification ]
```

### 3.3 Exercice mental « Lambda ou EC2 ? » (décision rapide)

```bash
# Pour chaque tâche, réponds : Lambda (L) ou EC2 (E) ?
#  - Traiter une image à chaque upload → L
#  - Faire tourner le backend Spring Boot 24h/24 → E
#  - Nettoyer les vieux logs toutes les heures → L
#  - Un traitement de données de 2 heures → E (ou service adapté)
# Réflexe : court + intermittent = L ; long + permanent = E.
```

---

## 4. Bonnes pratiques modernes (2025-2026)

- **Droits IAM minimaux** : un rôle Lambda ne fait que ce qu'il doit (Leçon 6).
- **Courte durée par conception** : concevoir des fonctions courtes ; si une tâche dépasse les limites, choisir une autre solution.
- **Idempotence** : si l'événement arrive **deux fois**, le résultat doit être le même (pas de doublon de traitement). Pratique clé 2025-2026 (les déclencheurs peuvent réessayer).
- **Observabilité** : logs structurés + métriques (durée, erreurs, cold starts) — Bloc 12.
- **Gérer le « cold start »** pour les usages sensibles à la latence.
- **Versions et alias** : versionner les fonctions (comme un tag Git) pour pouvoir revenir en arrière.

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi c'est dangereux/inefficace | ✅ Version correcte |
|----------------|-------------------------------------|---------------------|
| Mettre tout le backend dans Lambda | Durée/session trop longues → erreurs ou facture surprenante | Réserver Lambda aux **petites tâches** ; backend = EC2/services |
| Oubli de la limite de temps | Fonction coupée en plein travail (on croit à un bug) | Concevoir court + surveiller la durée |
| Lambda avec des clés embarquées | Fuite d'identifiants | **Rôle IAM** minimal (Leçon 6) |
| Ignorer les logs/erreurs | Impossible de diagnostiquer | Logs structurés + métriques (Bloc 12) |
| Fonction non idempotente | Traitement double si l'événement est relancé | Rendre la fonction **idempotente** |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`**, la correction dans **`03-correction.md`**.

**Énoncé court** : dans `notes-exercice-07.md`, écris le schéma `événement → Lambda → résultat` pour **3 cas d'usage** de ton application (ex. upload d'avatar, génération d'un PDF, nettoyage des vieux backups), et pour chacun **Lambda ou EC2 ?** avec 1 phrase. Ajoute la comparaison Lambda vs EC2 (tableau 5 critères) et le **plan anti-piège** (3 règles : idempotence, limites, IAM).

---

## 7. Correction détaillée de l'exercice

> La correction complète est dans **`03-correction.md`**. On y valide tes 3 cas, le tableau comparatif, et le plan anti-piège.

---

## 8. Checklist de validation

- [ ] J'explique le serverless et le rôle de Lambda.
- [ ] Je distingue Lambda vs EC2 et je choisis le bon selon la tâche.
- [ ] Je définis fonction, événement/trigger, facturation à l'exécution, limites, cold start.
- [ ] Je schématise `événement → Lambda → résultat` pour des cas réels.
- [ ] J'attache un rôle IAM minimal à une Lambda (Leçon 6).
- [ ] J'applique idempotence, logs et gestion du cold start.

---

🧭 **Pont vers la suite** — Tu sais maintenant **construire** l'architecture (réseau, VM, stockage, base, droits) et **choisir** le bon service. Dernier pilier du bloc : le **coût**. Une belle architecture trop chère n'est pas une bonne architecture. La Leçon 8 (FinOps) t'apprend à **maîtriser la facture** — le sujet que la roadmap considère comme acquis pour clore le bloc.

---

*Prochaine étape :* Leçon 8 — **FinOps et optimisation des coûts** dans `08-FinOps-et-optimisation-des-couts/`.
