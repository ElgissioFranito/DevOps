# Leçon 8 — Scalabilité, haute disponibilité et disaster recovery

> **Bloc 8 · Infrastructure as Code (IaC)** — Leçon 8 sur 9
> 🧭 **Pont depuis la Leçon 7** : tu sais désormais **créer** l'infrastructure en code (Terraform) et **configurer** les machines en code (Ansible) — et tu as vu à la Leçon 7 qu'ajouter une machine coûte une ligne d'inventaire. Mais **pourquoi** construit-on des architectures en code ? Parce qu'il faut pouvoir : **supporter plus d'utilisateurs** (scalabilité), **résister à des pannes** (haute disponibilité, tolérance aux fautes), et **se reconstruire après un incident majeur** (disaster recovery). C'est la partie « Architecture système » de la roadmap — une leçon de **conception**, plus que de code : à la fin, tu sauras répondre aux deux questions du bloc : *« Que se passe-t-il si Server 1 tombe ? »* et *« Que se passe-t-il si la base est détruite ? »*.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Définir** la **scalabilité** et distinguer **scalabilité verticale** (agrandir la machine) et **horizontale** (ajouter des machines), avec leurs limites.
2. **Définir** la **haute disponibilité (HA)** et la **tolérance aux fautes**, et repérer les **points de défaillance uniques** (*SPOF*).
3. **Définir** la **reprise après sinistre (DR)** et les deux indicateurs **RTO** et **RPO**, et les relier aux sauvegardes (Bloc 7, Leçon 4).
4. **Dessiner** l'architecture de référence : load balancer + N serveurs + base + backups — et **analyser** le comportement en cas de panne.
5. **Relier** la conception à l'IaC : pourquoi le code (Terraform + Ansible) rend ces architectures **réalisables et reconstruisibles**.

---

## 2. Explication simple

### 2.1 Le « pourquoi » : une seule machine, c'est tout ou rien

Reprends l'architecture que tu as codée en Leçon 5 : **une** machine, **une** base. Elle fonctionne… jusqu'à :

- la charge **monte** (le succès ! 10 000 visiteurs au lieu de 100) → la machine s'étouffe ;
- la machine **tombe** (matériel, bug, mise à jour ratée) → le service disparaît ;
- la région cloud a une **panne** (ça arrive — les grandes pannes font les gros titres) ;
- quelqu'un **supprime la base** (erreur humaine, attaque — le Bloc 7 t'a déjà montré les dégâts possibles).

Chacun de ces scénarios est la réponse à une question de conception. La roadmap l'exprime en un objectif : *concevoir une infrastructure qui supporte plus d'utilisateurs, résiste à certaines pannes, se restaure après un incident, évolue sans tout reconstruire*. Décomposons-la mot à mot.

### 2.2 La scalabilité : supporter la montée de la charge

La **scalabilité** est la capacité d'un système à supporter une **augmentation de charge** (plus d'utilisateurs, plus de requêtes). Deux approches opposées :

| Approche | Principe | Analogie | Avantage | Limite |
|----------|----------|----------|----------|--------|
| **Verticale** | Agrandir **la même** machine : 2 CPU → 8 CPU, 8 Go → 64 Go | **Agrandir sa maison** : un étage de plus | Simple : rien à refaire, l'adresse ne change pas | **Un plafond** : pas de machine infinie ; arrêt pendant l'upgrade ; toujours un seul point à casser |
| **Horizontale** | Ajouter **des machines** : 1 serveur → 3 serveurs | **Acheter plus de maisons** identiques | Quasi sans plafond ; panne tolérable (les autres continuent) | Plus complexe : il faut **répartir** la charge entre les maisons |

> 💡 **Analogie de la file d'attente** : verticale = **une seule caisse de supermarché avec un caissier plus rapide** ; horizontale = **ouvrir d'autres caisses**. À la longue, on ouvre des caisses : un caissier, si doué soit-il, a un maximum physique.

**Quand choisir quoi ?** La verticale convient au **début** (simple, rien à refaire) et à certains composants qui n'aiment pas être répartis (la base de données, souvent). L'horizontale est la voie **principale en cloud** — et voici pourquoi l'IaC la rend accessible : au Bloc 6, dupliquer une EC2 à la main = refaire toutes les étapes ; en IaC, dupliquer = **faire répéter le code** (Terraform sait créer N exemplaires d'une resource avec l'option `count` — tu en as vu l'esprit côté Ansible à la Leçon 7 : une ligne d'inventaire de plus).

### 2.3 La haute disponibilité : survivre à une panne

La **haute disponibilité (HA)** — *High Availability* — vise à **réduire les interruptions de service**. Le remède de base : **supprimer les points de défaillance uniques**.

Le **SPOF** (*Single Point Of Failure*) est tout composant dont la panne **arrête tout** :

```
Architecture fragile              Architecture disponible (HA)
[Load Balancer] ◄── SPOF !        [Load Balancer (géré, redondé par le cloud)]
      │                                  │
   [Server 1]  ◄── SPOF !          [Server 1]   [Server 2]
      │                                  │
   [Database]  ◄── SPOF !          [Database répliquée (Bloc 7, Leçon 6 / Multi-AZ)]
```

La **tolérance aux fautes** (*fault tolerance*) est le cran au-dessus : le système **continue de fonctionner malgré** une panne (l'utilisateur ne remarque rien), là où la HA « réduit l'interruption » (quelques secondes de bascule). L'outil central du HA applicatif est le **load balancer** (*répartiteur de charge* — rappel du Bloc 5, Leçon 6) : il reçoit toutes les requêtes et les distribue entre les serveurs ; un **health check** (test de santé régulier, ex. une requête toutes les 10 s) détourne le trafic d'un serveur mort.

> 💡 **Analogie** : le load balancer est le **réceptionniste** de l'hôtel qui oriente les clients vers la réception **disponible**. Si la caisse 1 ferme, il n'envoie plus personne à la caisse 1 : personne ne fait la queue devant une porte fermée.

Note utile chez AWS : les load balancers managés (ex. l'ALB — *Application Load Balancer*) sont **rédundants par construction** (le provider les héberge sur plusieurs zones) : utiliser un service managé supprime un SPOF que tu devrais autrement redonder toi-même.

### 2.4 La reprise après sinistre : se reconstruire

La **DR** (*Disaster Recovery*) est le **plan** qui permet de récupérer le système après un **incident majeur** : perte d'un serveur, suppression de données, panne régionale, corruption. La HA gère une **panne locale automatiquement** ; la DR gère le **sinistre** qu'on n'a pas pu éviter.

Deux indicateurs — les chiffres à connaître et à **quantifier** :

| Indicateur | Question à laquelle il répond | Analogie |
|------------|-------------------------------|----------|
| **RTO** (*Recovery Time Objective*) | **Combien de temps** peut-on accepter de rester hors service ? | « Dans combien de temps le magasin rouvre après l'incendie ? » |
| **RPO** (*Recovery Point Objective*) | **Quelle quantité de données** accepte-t-on de perdre ? | « Jusqu'à quand les reçus sont-ils à l'abri ? » |

Le RPO se mesure **en arrière depuis l'incident** : si les sauvegardes (Bloc 7, Leçon 4) sont **horaires**, un incident à 14 h 30 perd au pire 30 min de données — RPO ≈ 1 h. Pour un RPO de 15 min : des sauvegardes toutes les 15 min (ou moins), ou une réplication quasi synchrone (Multi-AZ de RDS, Bloc 7, Leçon 6 : chaque écriture copiée en continu dans une autre zone).

**Le lien avec l'IaC — le cœur de ce bloc** : une DR « à l'ancienne » repose sur un runbook papier et des heures d'installation manuelle (ton Bloc 7). Avec l'IaC, le plan de reprise tient en trois étapes scriptables : **`terraform apply`** (recréer l'infrastructure) → **`ansible-playbook`** (reconfigurer les machines) → **restaurer le backup** (Bloc 7, Leçon 4). La DR devient **rapide, répétée et testable** — exactement le critère d'acquis du bloc : *supprimer son infrastructure et la reconstruire de manière reproductible*.

### 2.5 Le « quand » : dès que le service est réel

Un prototype jetable peut rester sur une machine. Dès qu'un service a de **vrais utilisateurs**, les questions se posent dans cet ordre : 1) les sauvegardes (RPO — le Bloc 7 l'a fait) ; 2) la redondance applicative (LB + N serveurs) ; 3) la redondance de la base (Multi-AZ/réplication) ; 4) le plan DR écrit et **testé**. En termes de parcours : c'est ce qui prépare Docker + Kubernetes (Blocs 9-10), où la redondance sera la norme plutôt qu'un choix.

---

## 📖 Vocabulaire / Abréviations

| Terme | Définition (une ligne) | Où |
|-------|------------------------|-----|
| **Scalabilité** | Capacité à supporter une augmentation de charge | § 2.2 |
| **Scalabilité verticale** | Agrandir la même machine (plus de CPU/RAM) | § 2.2 |
| **Scalabilité horizontale** | Ajouter des machines identiques | § 2.2 |
| **HA (High Availability)** | Réduire les interruptions de service | § 2.3 |
| **SPOF** | *Single Point Of Failure* : composant dont la panne arrête tout | § 2.3 |
| **Fault tolerance** | Continuer à fonctionner malgré une panne | § 2.3 |
| **Load balancer / ALB** | Répartiteur de charge (ALB = Application Load Balancer d'AWS) | § 2.3 |
| **Health check** | Test de santé régulier d'un serveur, pour détourner le trafic | § 2.3 |
| **DR** | *Disaster Recovery* : le plan de récupération après sinistre majeur | § 2.4 |
| **RTO** | Temps maximal acceptable pour restaurer le service | § 2.4 |
| **RPO** | Quantité maximale de données acceptée comme perdue | § 2.4 |
| **Multi-AZ** | Copie de la base dans une autre zone (Bloc 7, Leçon 6) | § 2.4 |
| **Autoscaling** | Ajouter/retirer automatiquement des machines selon la charge (mention) | § 3.4 |

---

## 3. Exemples concrets

Passons de la théorie au dessin : l'architecture de référence, puis son analyse incident par incident — exactement ce que l'exercice te demandera de produire.

### 3.1 L'architecture de référence (celle que la roadmap demande)

```
                        Utilisateurs
                              │
                        [Load Balancer]        ← (ALB) : répartit + détecte les pannes
                        /             \
                 [Server 1]        [Server 2]   ← EC2 t3.micro (identiques : IaC + Ansible)
                        \             /
                      [Database]
                    PostgreSQL (RDS)            ← Multi-AZ : réplication dans une 2ᵉ zone
                          │
                      [Backups]
                    sauvegardes régulières      ← Bloc 7, Leçon 4 : testées !
                    (hors zone / région)
```

**Ce que chaque brique protège** :

| Brique | Contre quoi | Où tu l'as déjà vue |
|--------|-------------|---------------------|
| Load balancer | La panne **d'un serveur** + la répartition de la charge | Bloc 5, Leçon 6 |
| N serveurs identiques | La panne **d'un serveur** + la montée en charge | Leçon 8 (§ 2.2) |
| Multi-AZ / réplication | La panne **d'une zone** ou de la base | Bloc 7, Leçon 6 |
| Backups testés | La **perte de données** (RPO) | Bloc 7, Leçon 4 |
| Code IaC dans Git | La **reconstruction** après sinistre | Ce bloc (Leçons 2-7) |

### 3.2 L'analyse des deux questions de la roadmap

**« Que se passe-t-il si Server 1 tombe ? »** — Parcours de la panne :

```
1. Server 1 ne répond plus.
2. Le health check du load balancer échoue (2-3 tests manqués, ~30 s).
3. Le LB retire Server 1 de la rotation : plus aucune requête vers lui.
4. Tout le trafic passe à Server 2. L'utilisateur : au pire une requête
   échouée pendant la bascule, puis plus rien.
5. Reconstruire Server 1 : terraform apply (recrée) + ansible-playbook
   (reconfigure) — ou automatiquement via autoscaling (§ 3.4).
```

**« Que se passe-t-il si la base est détruite ? »** — Parcours :

```
1. Les serveurs ne peuvent plus lire/écrire la base → erreurs applicatives.
2. Deux trajectoires :
   a) Panne matérielle : le Multi-AZ bascule sur la réplique (minutes).
   b) Suppression logique (erreur humaine) : la réplique a propagé la MÊME
      erreur ! → il faut RESTAURER une sauvegarde (Bloc 7, Leçon 4).
3. La perte de données max = le RPO : la distance depuis la DERNIÈRE sauvegarde.
4. Restauration : recréer l'instance depuis le snapshot (ou pg_restore), puis
   re-pointer les serveurs vers le nouvel endpoint (Bloc 6, Leçon 5 : l'endpoint).
```

> 🔑 **La subtilité à retenir** : la réplication protège contre la **panne**, pas contre l'**erreur** — une réplique copie aussi les erreurs (une `DROP TABLE` est répliquée !). Seule la **sauvegarde datée** remonte dans le temps. C'est la leçon du Bloc 7 appliquée à l'architecture.

### 3.3 RTO/RPO : un calcul de conception

Exigence : **RTO ≤ 30 min, RPO ≤ 15 min**. Traduction concrète :

| Exigence | Traduction technique |
|----------|----------------------|
| RPO ≤ 15 min | Sauvegardes **au moins toutes les 15 min** (ou réplication continue) — sinon le RPO n'est pas garanti, quel que soit le reste |
| RTO ≤ 30 min | Un plan **scripté et testé** : recréer l'infra (IaC, ~10 min) + reconfigurer (Ansible, ~5 min) + restaurer la base (~10 min) — à **chronométrer** au test trimestriel, pas à deviner |

Un RTO s'annonce **après un test** : « notre plan a été rejoué le 3 mars, il a pris 22 minutes ». C'est la différence entre une DR **écrite** et une DR **démontrée** — le même esprit que « un backup non testé n'est pas un backup » (Bloc 7).

### 3.4 Le pont vers le code : ce que l'IaC rend possible

Trois briques déjà dans ton vocabulaire, orientées HA :

- **`count`** (Terraform) : l'option qui crée N exemplaires d'une resource — `count = 2` sur l'`aws_instance` de la Leçon 5 = deux serveurs depuis le même code (le frère Ansible en est la Leçon 7 : une ligne d'inventaire).
- **`aws_lb`** : le load balancer en code, avec ses **health checks** déclarés — la HA applicative devient un bloc HCL de plus, prévisualisable en plan.
- **Autoscaling** (à mentionner, sans creuser) : le groupe d'instances qui **ajoute/retire** des serveurs selon la charge — l'horizontale + l'automatique, gérés par AWS.

Le code rend l'architecture **révisable** : passer de 1 à 3 serveurs est une **modification de code** relue dans Git (Bloc 4), pas une série de clics — et une reconstruction après sinistre est le **même code** appliqué ailleurs.

---

## 4. Bonnes pratiques modernes (2025-2026)

- **L'horizontale par défaut, la verticale en exception** : en cloud, on ajoute des machines plutôt qu'on n'agrandit une seule — sauf pour les composants mono-instance (la base relationnelle, souvent).
- **Des services managés pour supprimer les SPOF** : un ALB est répliqué par AWS ; une RDS Multi-AZ bascule automatiquement — construire sa propre redondance n'est utile que si le service n'existe pas.
- **Quantifier RTO et RPO AVANT de choisir les technologies** : les exigences dictent l'architecture (sauvegardes 15 min ? réplication ? région secondaire ?), pas l'inverse.
- **Backups hors de la zone (voire hors de la région)** : une sauvegarde qui brûle avec le serveur ne sauvegarde rien (le choix vu au Bloc 7).
- **Tester la DR au moins trimestriellement** : un test chronométré (rebuild IaC + restore) remplace la confiance par une mesure.
- **Tout l'infra en code, le plan DR dans le dépôt** : le runbook DR est un fichier versionné, relu et amélioré comme le reste.

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi c'est dangereux/inefficace | ✅ Version correcte |
|-----------------|--------------------------------------|---------------------|
| Tout miser sur la verticale (toujours plus gros) | Plafond matériel, arrêt à chaque upgrade, SPOF permanent | Horizontale par défaut ; verticale pour les cas limités |
| « On a des backups, donc on est DR-ready » | Non testés = incertains ; hors région = sauvés de la panne régionale | Test trimestriel chronométré, backups hors zone |
| Compter sur la réplication contre les erreurs humaines | La `DROP TABLE` est **répliquée** aussi ! | Réplication pour la panne ; **sauvegardes datées** pour l'erreur |
| Un RTO « annoncé » jamais chronométré | Jour J : le plan prend 3 h au lieu de 30 min | Chronométrer au test ; ajuster le plan ou le RTO |
| Un load balancer artisanal (une seule machine LB maison) | Le LB devient le nouveau SPOF | Load balancer **managé** (ALB), redondé par le provider |
| Oublier de re-pointer les clients vers le nouvel endpoint après restore | Le service reste « planté » sur une base morte | Endpoint dans une variable/config, jamais en dur (Bloc 6, Leçon 5) |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`**, la correction dans **`03-correction.md`**.

**Énoncé court** : pour l'application bibliothèque (10 000 utilisateurs, RTO 30 min / RPO 15 min) — complète le schéma d'architecture cible (LB, N serveurs, base protégée, backups), choisis et justifie vertical/horizontal par composant, analyse 4 incidents (serveur mort, saturation, base détruite, région hors service), écris la stratégie de sauvegarde qui garantit le RPO et le plan DR en 6 étapes testées trimestriellement, et relie le tout au code (count, `aws_lb`, rôle Ansible).

---

## 7. Correction détaillée de l'exercice

> La correction complète (architecture de référence, analyses de pannes, plan DR) est dans **`03-correction.md`**.

---

## 8. Checklist de validation

- [ ] Je définis la scalabilité et je distingue verticale (agrandir) / horizontale (ajouter), avec leurs limites.
- [ ] Je définis HA, tolérance aux fautes et SPOF, et je repère les SPOF d'un schéma.
- [ ] Je définis DR, RTO, RPO — et je relie RPO à la fréquence des sauvegardes.
- [ ] Je dessine l'architecture LB + N serveurs + base répliquée + backups, et j'analyse une panne serveur par serveur.
- [ ] Je réponds aux deux questions de la roadmap (serveur tombé / base détruite) sans hésiter.
- [ ] J'explique pourquoi l'IaC (Terraform + Ansible + code dans Git) est l'accélérateur de la DR — et je le relie au critère d'acquis du bloc.

---

🧭 **Pont vers la suite** — Il ne reste qu'une chose à prouver, celle que la roadmap exige pour clore le bloc : *« Tu peux supprimer ton infrastructure et la reconstruire de manière reproductible »*. La **Leçon 9** est le projet récapitulatif : un scénario de sinistre complet — infra détruite, état perdu, backup disponible — que tu vas surmonter **de bout en bout** : Terraform recrée, Ansible reconfigure, la base se restaure, et tu chronomètres. Le scénario A se joue en local, le scénario B sur AWS — comme toujours, sans coût surpris.

---

*Prochaine étape :* Leçon 9 — **Projet récapitulatif : détruire et reconstruire** dans `09-Projet-recapitulatif-detruire-et-reconstruire/`.