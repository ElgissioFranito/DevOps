# Correction — Leçon 8 : concevoir pour survivre

> **Bloc 8 · Leçon 8** — Correction complète : une **version de référence** du document d'architecture demandé. Compare-la avec tes réponses : ce qui compte, c'est de pouvoir **justifier** chaque choix.

---

## Étape 1 — Le schéma d'architecture cible (version de référence)

```
                        Utilisateurs
                              │
                        [Load Balancer]        aws_lb (ALB) : répartit la charge,
                        /             \        health checks → détecte les pannes
                 [Server 1]        [Server 2]   aws_instance (count=2) : machines
                        \             /        identiques, configurées par le rôle Ansible
                      [Database]
                    PostgreSQL (RDS)           aws_db_instance + Multi-AZ : réplication
                                               dans une 2ᵉ zone (panne de zone absorbée)
                          │
                      [Backups]               snapshots RDS planifiés + pg_dump
                    (hors zone / région)      toutes les 15 min (S3, versionné)
```

**Justifications par brique** :
- **Load balancer** : unique point d'entrée, mais **managé** (répliqué par AWS sur plusieurs zones) → il n'est pas un SPOF. Il détecte les serveurs morts (health check) et masque leur panne.
- **2 serveurs identiques** : la charge se répartit ; la panne d'un serveur n'interrompt pas le service.
- **Base Multi-AZ** : la panne d'une zone bascule sur la réplique, automatiquement.
- **Backups 15 min, hors zone** : garantissent le RPO 15 min **et** survivent à un sinistre de zone.

## Étape 2 — Choix de scalabilité

| Composant | Choix | Justification |
|-----------|-------|---------------|
| Machines applicatives | **Horizontale** | « Stateless » (*sans état* : aucune donnée locale vitale) → dupliquables à volonté ; la répartition est résolue par le LB |
| Base PostgreSQL | **Verticale d'abord, puis réplication** | Une base relationnelle écrit de façon cohérente en un point ; on agrandit (instance class), puis on réplique (Multi-AZ) pour la disponibilité |
| Stockage documents | **Horizontale, automatique** | S3 s'élargit tout seul : c'est le service managé le plus scalable — rien à décider |

---

## Étape 3 — L'analyse des 4 pannes

**1. Server 1 tombe.**
- *Immédiat* : le health check du LB échoue (~30 s), Server 1 est retiré de la rotation.
- *Utilisateur* : au pire une requête en échec pendant la bascule, puis service normal.
- *Action* : recréation — `terraform apply` (machine) + `ansible-playbook site.yml` (config), ou automatique via autoscaling.
- *RTO approx.* : ~10-15 min (sans autoscaling) / ~5 min (avec).

**2. Saturation (10 000 → 50 000 utilisateurs).**
- *Immédiat* : latences croissantes, CPU en pic sur les deux serveurs.
- *Utilisateur* : lenteurs, parfois des timeouts.
- *Action* : **scale out** — `count = 3` dans le code → `terraform apply` → une ligne dans l'inventaire Ansible → `ansible-playbook`. Le LB répartit vers le nouveau serveur sans rien changer d'autre.
- *RTO approx.* : non applicable (évolution anticipée, pas une panne) — c'est la scalabilité qui la rend banale.

**3. La base est détruite** *(la question de la roadmap !)*.
- *Immédiat* : les serveurs échouent à joindre la base ; erreurs applicatives partout.
- *Utilisateur* : service indisponible (pages en erreur).
- *Action* : deux trajectoires — panne matérielle : bascule Multi-AZ (minutes). Suppression **logique** : la réplique a la même erreur → **restauration depuis le backup daté** (snapshot RDS ou `pg_restore`), puis re-pointer les serveurs sur le **nouvel endpoint** (dans une variable, jamais en dur).
- *RPO* : perte max = temps depuis la dernière sauvegarde = **15 min** (c'est la définition).
- *RTO approx.* : ~20-25 min (recréer l'instance + restore + re-pointage) — dans l'objectif des 30 min, **à condition d'avoir chronométré**.

**4. La région est hors service 2 h.**
- *Immédiat* : tout le VPC est inaccessible.
- *Utilisateur* : service indisponible.
- *Action* (plan DR) : le code étant **versionné et régionalisable** (une variable `region`), on recrée l'infra dans une autre région (`terraform apply` avec la nouvelle région), on reconfigure (Ansible), on restaure les données depuis les backups **stockés hors région** (précondition !), on bascule le DNS vers le nouveau endpoint (Bloc 5 : la propagation DNS prend du temps — comptée dans le RTO).
- *Honnêteté technique* : à ce niveau, la bascule n'est **pas automatique** (une vraie multi-région automatisée est un projet lourd, hors périmètre) ; le plan DR **scripté et répété** est la réponse réaliste — et la sauvegarde hors région est la condition non négociable.

---

## Étape 4 — RTO/RPO et plan de sauvegarde

**Stratégie garantissant RPO 15 min** : sauvegardes **toutes les 15 min** (snapshot RDS automatisé avec fenêtre planifiée + `pg_dump` pour le dump logique), stockées dans **un autre AZ et une autre région** (S3, versionné — Bloc 7, Leçon 4 + Bloc 6, Leçon 4). Rétention : 7 jours de snapshots complets, plus des archives mensuelles.

**Le plan DR en 6 étapes** :

| # | Étape | Mode |
|---|-------|------|
| 1 | Constat du sinistre (alertes — le monitoring arrive au Bloc 12 ; à ce stade : surveillance + alerte budget) | Surveillance |
| 2 | Recréer l'infrastructure dans la zone/région saine | **IaC** : `terraform apply` |
| 3 | Reconfigurer les machines | **IaC** : `ansible-playbook site.yml` |
| 4 | Restaurer la base depuis le dernier backup sain | Backup (Bloc 7, Leçon 4) |
| 5 | Re-pointer les applications (nouvel endpoint, via variables) | Config (Ansible) |
| 6 | Vérifier le service (health checks, tests applicatifs) + **chronométrer** le total | Validation |

**Le test trimestriel** : chaque trimestre, rejouer les 6 étapes **dans un environnement jetable** (dev) : détruire, restaurer, mesurer. Le résultat — durée réelle + pertes réelles — met à jour les RTO/RPO **annoncés** (et non plus espérés). C'est le test mensuel de restauration du Bloc 7, élevé à l'échelle de l'infrastructure.

---

## Étape 5 — Le lien avec le code

1. **`skip_final_snapshot = true`** (Leçon 5) : à la destruction, **aucune** sauvegarde finale — parfait pour l'exercice (tout disparaît net), **interdit en production** : là, `skip_final_snapshot = false` + `final_snapshot_identifier` = un dernier snapshot obligatoire, la protection ultime contre un « destroy » accidentel.
2. **Passer à 2 EC2** : `count = 2` sur `aws_instance` (la répétition de resource), + **`aws_lb`** obligatoire (plus d'IP publique unique : le LB devient l'adresse d'entrée, avec ses health checks) + un security group du LB vers les serveurs. Les deux serveurs sont ensuite configurés par **le même appel Ansible** (inventaire : 2 lignes).
3. **Le rôle Ansible de la Leçon 7** est la moitié de la solution car la scalabilité horizontale a deux faces : créer N machines (Terraform, `count`) **et** les configurer identiquement (Ansible : une seule source, N cibles). Sans le rôle, chaque serveur « copié » divergerait — l'anti-IaC.

> 🔑 **La synthèse du bloc en une phrase** : l'architecture HA/DR se **décrit en code** (Terraform), se **configure en code** (Ansible), se **sauvegarde** (Bloc 7), et se **reconstruit en exécutant le code** — c'est exactement le critère d'acquis de la roadmap, que le projet final (Leçon 9) va faire prouver.

---

## Checklist de validation (leçon 8)

- [ ] Je définis la scalabilité et je distingue verticale (agrandir) / horizontale (ajouter), avec leurs limites.
- [ ] Je définis HA, tolérance aux fautes et SPOF, et je repère les SPOF d'un schéma.
- [ ] Je définis DR, RTO, RPO — et je relie RPO à la fréquence des sauvegardes.
- [ ] Je dessine l'architecture LB + N serveurs + base répliquée + backups, et j'analyse une panne serveur par serveur.
- [ ] Je réponds aux deux questions de la roadmap (serveur tombé / base détruite) sans hésiter.
- [ ] J'explique pourquoi l'IaC (Terraform + Ansible + code dans Git) est l'accélérateur de la DR — et je le relie au critère d'acquis du bloc.

---

## 🧠 Conseils pour la suite

- **Entraîne-toi à l'oral** : explique ton schéma à voix haute en 2 min (LB, serveurs, base, backups — et ce que chacun protège). Les entretiens DevOps posent littéralement ces questions.
- **La subtilité réplication ≠ backup** est une réponse qui distingue un junior formé d'un junior improvisé : garde-la.
- **Leçon 9** : le projet final va faire **vivre** ce plan — prépare ton compartiment S3 du backend (Leçon 5) et ton rôle Ansible (Leçon 7), ils entrent en scène ensemble.
