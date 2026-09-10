# Correction — Leçon 8 : FinOps et optimisation des coûts

> **Bloc 6 · Leçon 8** — Correction pas à pas.

---

## Étape 1 — Script audit-finops.sh

Le script doit poser les 5 questions : (1) VM qui tourne sans utilité ? (2) taille adaptée (rightsizing) ? (3) stockage orphelin ? (4) transferts sortants massifs ? (5) budget + alerte en place ? C'est une **checklist de réflexe**, pas un vrai outil — mais c'est exactement la démarche FinOps que tu appliqueras avec Cost Explorer une fois le compte créé.

## Étape 2 — Les 4 postes de coût (réponse type)

| Poste | Ce qui coûte | Réflexe |
|-------|--------------|---------|
| **VM (EC2)** | À l'heure d'exécution | Supprimer/arrêter les inutilisées ; rightsizing |
| **Stockage** | Volume (S3 + disques EBS) | Purge des orphelins ; lifecycle (Leçon 4) |
| **Réseau** | Données sortantes (egress) | Surveiller les gros transferts |
| **Base (RDS)** | Instance + storage + backups | Bonne classe ; backups bornés |

## Étape 3 — Les 3 gaspillages

1. **VM de test oubliée** : elle tourne 24h/24 → **suppression** (ou arrêt) après le test.
2. **VM surdimensionnée** : CPU moyen très bas sur une grosse machine → **rightsizing** (réduire la taille).
3. **Backups / buckets illimités** : volume qui gonfle → **lifecycle** de rétention (ex. 30 jours).

## Étape 4 — Plan budget/alerte (exemple)

- **Budget mensuel** : 25 € (raisonnable pour une petite archi de débutant).
- **Alertes** : à **50 %** (≈ 12,5 €) et à **80 %** (≈ 20 €).
- **Réaction** : à 50 % → inspecter Cost Explorer, chercher les gaspillages ; à 80 % → **couper/stopper** ce qui n'est pas indispensable et prévenir l'équipe. Objectif : **ne jamais atteindre 100 % en silence**.

---

## Checklist de validation (leçon 8)

- [ ] J'explique FinOps (informer → optimiser → opérer).
- [ ] Je liste les 4 postes de coût.
- [ ] J'identifie les gaspillages et leurs actions.
- [ ] J'explique rightsizing, autoscaling, budgets, alertes, Cost Explorer.
- [ ] J'estime un ordre de grandeur de coût.
- [ ] Je propose des optimisations sans dégrader le service.

---

## 🧠 Conseils pour la suite

- **Voir avant d'agir** : tags + Cost Explorer + budgets tôt.
- **Commencer par le plus gros poste** (souvent VM + base) — ne pas perdre de temps sur les cents.
- **Auto-scaling** aux pics, mais avec la bonne taille de base.
- Ce socle cloud est maintenant acquis → le **Bloc 7 (Bases de données)** et le **Bloc 8 (IaC/Terraform)** pourront s'appuyer dessus : tu automatiseras toute cette architecture en code.