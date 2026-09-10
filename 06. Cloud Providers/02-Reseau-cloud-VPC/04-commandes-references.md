# Référence rapide — Leçon 2 : Réseau cloud (VPC)

> Bloc 6 · Leçon 2 — Aide-mémoire.

## Le principe d'or
La **base de données ne doit jamais être accessible depuis Internet** : subnet privé + security group restreint à l'application.

## Architecture type
```
Internet → DNS → IGW → Load Balancer [public] → App [privé] → DB [privé]
```

## Les briques du VPC
| Composant | Rôle |
|-----------|------|
| VPC | Réseau privé virtuel global du projet |
| Subnet public | Zone exposée (load balancer) |
| Subnet privé | Zone cachée (app, DB) |
| Internet Gateway (IGW) | Porte du VPC vers Internet |
| NAT Gateway | Sortie discrète des ressources privées |
| Security Group | Pare-feu virtuel par ressource |
| Route Table | Où envoyer les paquets |
| DNS | Nom → IP |
| Load Balancer | Répartit la charge |
| VPN | Tunnel chiffré qui relie deux réseaux séparés par Internet comme s'ils étaient un seul réseau privé (ex. bureau → VPC) |

## Commandes AWS (après clés — Leçon 6)
```bash
aws ec2 create-vpc --cidr-block 10.0.0.0/16       # créer un VPC
aws ec2 create-subnet --vpc-id vpc-0abc123 --cidr-block 10.0.1.0/24
aws ec2 describe-vpcs                             # lister les VPC
aws ec2 describe-subnets                          # lister les subnets
```

## Mémo CIDR (rappel Bloc 5)
- `/16` → 65 536 IP (grand réseau) ; `/24` → 256 IP (un subnet typique).

## Réflexes sécurité (défense en profondeur)
- Base privée, jamais exposée.
- Security Group en « défaut-deny » : n'autoriser que le strict nécessaire.
- Le NAT Gateway sert à sortir discrètement, pas à exposer.

## Bonnes pratiques
- Un VPC clair avec subnets public/privé plutôt que plusieurs VPC confus.
- Tout réseau en code au Bloc 8 (IaC/Terraform).
- Nommer les ressources pour les retrouver et les facturer.