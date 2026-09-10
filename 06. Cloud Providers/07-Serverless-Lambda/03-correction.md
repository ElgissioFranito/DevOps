# Correction — Leçon 7 : Serverless (Lambda)

> **Bloc 6 · Leçon 7** — Correction pas à pas.

---

## Étape 1 — Schémas (réponse type)

**A — Avatar téléversé**
```
[Upload avatar sur S3] → [Lambda "resize_avatar"] → [miniature dans un bucket "processed" + URL]
```

**B — PDF de facture**
```
[Requête HTTP de l'application] → [Lambda "generer_pdf"] → [PDF généré renvoyé / stocké sur S3]
```

**C — Purge des backups**
```
[minuteur (cron) à 02h00] → [Lambda "nettoyer_backups"] → [suppression des objets de + de 30 jours]
```

**Explication** : chaque tâche est **courte et déclenchée par un événement** → c'est le profil typique de Lambda. Le déclencheur (S3, HTTP, horloge) réveille la fonction, qui fait son travail puis s'éteint.

## Étape 2 — Lambda ou EC2 ?

- **A** : **L** — courte, à chaque upload (intermittent), scale auto.
- **B** : **L** — générer un PDF prend quelques secondes, à chaque demande.
- **C** : **L** — courte, toutes les nuits, automatisée par une horloge.

Aucune des trois n'exige une machine permanente → **Lambda** partout pour ces cas. (Le backend Spring Boot, lui, reste sur EC2 : long + permanent.)

## Étape 3 — Tableau comparatif (attendu)

| Critère | Lambda | EC2 |
|---------|--------|-----|
| Travail | Court, à la demande | Long, permanent |
| Serveur à gérer | Aucun | Toi |
| Facturation | Par exécution/durée | À l'heure |
| Mise à l'échelle | Automatique | Manuelle/autoscaling |
| Limites | Durée/mémoire bornées | CPU/RAM quasi libres sel. type |

## Étape 4 — Plan anti-piège (3 règles)

1. **Idempotence** : si l'événement est relancé, pas de traitement double (pas de doublon de facture).
2. **Rester dans les limites** : fonctions courtes, bien dimensionnées en mémoire/durée.
3. **IAM minimal + logs** : uniquement les droits nécessaires via un rôle IAM, et des logs structurés pour diagnostiquer (Bloc 12).

---

## Checklist de validation (leçon 7)

- [ ] J'explique le serverless et Lambda.
- [ ] Je distingue Lambda vs EC2 et choisis le bon.
- [ ] Je définis fonction, événement/trigger, facturation, limites, cold start.
- [ ] Je schématise `événement → Lambda → résultat`.
- [ ] J'attache un rôle IAM minimal.
- [ ] J'applique idempotence, logs, gestion du cold start.

---

## 🧠 Conseils pour la suite

- Lambda ne remplace **pas** EC2 : il **complète**. Apprends à **choisir** (court/intermittent vs long/permanent).
- La facturation « 0 € quand ça ne tourne pas » est très utile pour des petites tâches.
- Prochaine étape : la **Leçon 8 (FinOps)** — maîtriser la facture cloud de toute l'architecture.