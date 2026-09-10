# Référence rapide — Leçon 7 : Serverless (Lambda)

> Bloc 6 · Leçon 7 — Aide-mémoire.

## Concepts
- **Serverless** : exécuter du code **sans gérer de serveur** (le cloud gère tout).
- **Lambda** : le service serverless d'AWS.
- **Fonction Lambda** : petit bout de code (Python, Node.js…) exécuté à chaque événement.
- **Déclencheur / trigger** : upload S3, requête HTTP, message, minuteur.
- **Facturation à l'exécution** : par appel + durée ; 0 € si la fonction ne tourne pas.
- **Stateless** : pas d'état d'un appel à l'autre.
- **Cold start** : léger délai au premier appel après inactivité.
- **Rôle IAM** : autoriser la fonction (Leçon 6), jamais de clé embarquée.

## Schéma type
```
Événement (S3/HTTP/horloge) → Lambda → Résultat
```

## Lambda ou EC2 ?
- **Court, intermittent, variable** → **Lambda**.
- **Long, permanent, exigeant** → **EC2** (ou services managés).

| | Lambda | EC2 |
|--|--------|-----|
| Facturation | Par exécution/durée | À l'heure |
| Serveur | Aucun | Toi |
| Scale | Automatique | Manuelle/autoscaling |
| Limites | Durée/mémoire bornées | Grandes sel. type |

## Fonction Python minimale
```python
import json
def lambda_handler(event, context):
    print("Événement reçu :", json.dumps(event))
    return {"statusCode": 200, "body": json.dumps("Traité !")}
```

## Bonnes pratiques
- IAM minimal ; idempotence ; logs structurés ; cold start maîtrisé.
- Respecter la durée/mémoire ; versionner les fonctions.