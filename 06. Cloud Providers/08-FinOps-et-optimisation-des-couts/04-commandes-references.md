# Référence rapide — Leçon 8 : FinOps et optimisation des coûts

> Bloc 6 · Leçon 8 — Aide-mémoire.

## FinOps en 1 phrase
Gérer et optimiser les coûts cloud : **informe → optimise → opère** (en boucle).

## Les 4 postes de coût
| Poste | Coût | Réflexe |
|-------|------|---------|
| VM (EC2) | À l'heure | supprimer/arrêter les inutiles ; rightsizing |
| Stockage | Volume (S3 + EBS) | purge orphelins ; lifecycle |
| Réseau | Sortant (egress) | surveiller les gros transferts |
| Base (RDS) | Instance + stockage + backups | bonne classe ; backups bornés |

## Les 3 gaspillages classiques
1. VM de test oubliée → **supprimer**.
2. VM surdimensionnée → **rightsizing** (adapter la taille).
3. Backups/buckets illimités → **lifecycle** (rétention bornée).

## Outils / pratiques
- **Cost Explorer** : voir les coûts par service (`aws ce get-cost-and-usage`).
- **Budgets + alertes** à 50 % / 80 %.
- **Autoscaling** aux pics (avec la bonne taille de base).
- **Tags** (`env=prod`, `projet=…`) dès la création des ressources.

## Estimations rapides (ordre de grandeur, tarifs variables)
- t3.micro ≈ 0,011 $/h ≈ 8 $/mois (+ disque EBS ≈ 2 $/mois)
- RDS db.t3.micro PostgreSQL ≈ +12 $/mois
- → Commencer par le plus gros poste (souvent VM + base).

## Règle d'or
**Optimiser sans dégrader le service** : la bonne capacité au bon moment, avec le moins de gaspillage.