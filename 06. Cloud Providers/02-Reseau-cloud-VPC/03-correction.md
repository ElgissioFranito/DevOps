# Correction — Leçon 2 : Réseau cloud (VPC)

> **Bloc 6 · Leçon 2** — Correction pas à pas.

---

## Étape 1 — Schéma de référence

```
Internet
   ↓ (DNS : exemple.com → IP)
Internet Gateway (IGW)   [PUBLIC, la porte vers Internet]
   ↓
Load Balancer            [PUBLIC, joignable depuis Internet — c'est l'ENTRÉE que voient les clients]
   ↓
Application (Spring Boot) [PRIVÉ, dans un subnet privé — seule l'application interne et le LB la joignent]
   ↓
Base de données PostgreSQL [PRIVÉ, la plus cachée — jamais accessible depuis Internet]
```

**Choix public/privé justifiés** :
- **DNS, IGW, Load Balancer** = public : ils sont le point d'entrée que le monde extérieur voit et joint.
- **Application** = privé : elle ne doit pas être directement attaquable ; seul le load balancer (interne) et les autres services internes l'atteignent.
- **Base de données** = privé : la donnée sensible est la cible des pirates ; on la met au bout, derrière toutes les couches.

## Étape 2 — Rôle de chaque brique

| Composant | Rôle |
|-----------|------|
| VPC | Réseau privé virtuel global du projet cloud |
| Subnet public | Zone du VPC exposée (via l'IGW) — load balancer |
| Subnet privé | Zone du VPC cachée — application, base |
| Internet Gateway (IGW) | La porte qui relie le VPC à Internet |
| NAT Gateway | Sortie discrète vers Internet pour les ressources privées |
| Security Group | Pare-feu virtuel qui filtre le trafic par ressource |

## Étape 3 — Script Bash

```bash
# afficher-vpc.sh
echo "🏢 Mon VPC 10.0.0.0/16"
echo "  ├── [PUBLIC] load balancer  10.0.1.10  → joignable depuis Internet"
echo "  ├── [PUBLIC] internet gateway (IGW)     → la porte vers Internet"
echo "  └── [PRIVÉ ] application    10.0.2.10  → cachée, IP privée"
echo "      └── [PRIVÉ ] base de données 10.0.3.10 → totalement cachée"
```
Puis `chmod +x afficher-vpc.sh` (donne le droit d'exécution, Bloc 2) et `./afficher-vpc.sh` (l'exécute depuis le dossier courant).

## Étape 4 — Réflexion

**1. Base dans un subnet public + security group ouvert ?**
> Catastrophe de sécurité : la base serait **joignable depuis tout Internet** (`0.0.0.0/0`). N'importe quel pirate pourrait tenter de s'y connecter, voler ou détruire les données. C'est exactement l'anti-pattern à éviter absolument.

**2. NAT Gateway pour l'application privée, jamais pour la base ?**
> L'application privée doit parfois **sortir** vers Internet (mises à jour, sauvegardes externes) sans être joignable → le NAT Gateway lui donne une sortie discrète. La base, elle, n'a **pas besoin de sortir** : si elle doit sortir, c'est un signal d'alerte. On la garde 100 % fermée (moindre exposition).

---

## Checklist de validation (leçon 2)

- [ ] Je définis le VPC et son rôle.
- [ ] J'explique subnet, route table, IGW, NAT, Security Group, DNS, LB, VPN.
- [ ] Je distingue subnet public/privé et IP publique/privée.
- [ ] Je dessine l'architecture réseau (LB → App → DB) et justifie chaque choix.
- [ ] Je sais pourquoi la base ne doit jamais être publique.
- [ ] Je lis les commandes AWS de base VPC/subnet et respecte la règle d'or.

---

## 🧠 Conseils pour la suite

- La base **toujours privée** = réflexe n°1 de toute architecture cloud.
- On décrira ce VPC en **code** (Terraform) au Bloc 8 — retiens bien la vision, tu vas l'automatiser.
- Le Cloud met le réseau du Bloc 5 « à l'échelle » : réutilise ce que tu sais déjà (subnet, pare-feu, LB).
- Prochaine étape : la **machine** qui héberge l'application → Leçon 3 (VM / EC2).