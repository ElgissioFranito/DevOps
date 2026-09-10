# Correction — Leçon 3 : Machines virtuelles (EC2)

> **Bloc 6 · Leçon 3** — Correction pas à pas.

---

## Étape 1 — Fiche de décision (réponse type)

| Question | Ton choix | Pourquoi |
|----------|-----------|----------|
| Quoi (service AWS) | **EC2** | C'est le service de VM d'AWS (La VM louée). |
| AMI (système) | **Ubuntu 24.04 LTS** | Compatible avec ton parcours Linux (Bloc 2), grande communauté. |
| Type d'instance | **`t3.micro`** (1 CPU / 1 Go) | 50 req/min = faible charge ; on commence petit (sizing). |
| Subnet (public ou privé) | **Privé** (si derrière un LB) / **public** (si exposée directement) | Règle d'or : moins exposé, mieux. Derrière un LB (Leçon 2) → privé. |
| Security group (ports) | **443/80** depuis le LB, **SSH restreint à ton IP** | N'ouvrir que le strict nécessaire (défaut-deny, Bloc 5). |
| Clé SSH (nom) | `ma-cle-ssh` | C'est ta paire de clés ; la clé privée reste chez toi en `chmod 600`. |
| Snapshot (fréquence) | **quotidien + avant chaque changement majeur** | Pour pouvoir restaurer après un incident (DR, Bloc 8). |

## Étape 2 — Script simulé

`simuler-instance.sh` doit afficher les infos choisies (AMI, type, état, subnet, clé) et les questions de sizing. Le lancer avec `bash simuler-instance.sh` produit une **trace écrite** de la décision — c'est exactement le genre de doc qu'on garde (et qu'on automatise plus tard en IaC, Bloc 8).

## Étape 3 — Réflexion

**1. Running / stopped / terminated ?**
> - **running** = l'instance tourne **et coûte** (à l'heure).
> - **stopped** = arrêtée ; on paie seulement le **disque** (beaucoup moins cher).
> - **terminated** = **détruite** ; on ne paie plus rien, mais **tout est perdu** (d'où les snapshots !).

**2. Si la demande explose x1000 ?**
> On **dimensionne** (passer à un type plus gros) et surtout on passe en **autoscaling + load balancer** : plusieurs instances `t3` derrière un load balancer (Leçon 2), avec des règles automatiques (CPU > 80 % → +1 instance). Voire, pour les pics, des alternatives serverless (Leçon 7, Lambda). On évite d'acheter une grosse machine dès le départ.

---

## Checklist de validation (leçon 3)

- [ ] J'explique EC2 (la VM d'AWS) et je le place dans un VPC.
- [ ] Je définis instance, type d'instance, AMI, key pair, security group.
- [ ] Je distingue running / stopped / terminated et leurs coûts.
- [ ] Je crée et connecte une instance (démarche + commandes / simulation).
- [ ] Je dimensionne (sizing) et j'explique l'autoscaling.
- [ ] Je fais un snapshot et j'explique son utilité.

---

## 🧠 Conseils pour la suite

- **Commencer petit et observer** ; ne jamais lancer de grosse instance sans besoin.
- **La clé privée ne se commit jamais** et reste en `chmod 600`.
- L'instance est **éphémère** : les données durables ne vivent pas sur la VM (→ S3, Leçon 4, et base managée, Leçon 5).