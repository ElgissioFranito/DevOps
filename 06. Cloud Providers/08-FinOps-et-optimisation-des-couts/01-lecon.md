# Leçon 8 — FinOps et optimisation des coûts

> **Bloc 6 · Cloud Providers** — Leçon 8 sur 8
> 🧭 **Pont depuis les Leçons 1-7** : tu sais maintenant **construire** une architecture cloud complète (VPC, VM/EC2, S3, RDS, IAM) et **choisir** le bon service (Lambda vs EC2). Mais une architecture parfaite techniquement peut être **ruineuse** financièrement. Ce dernier pilier du bloc s'appelle **FinOps** : la discipline pour **maîtriser la facture cloud**. C'est aussi le sujet que la roadmap considère comme **critère de fin du bloc** : savoir estimer ce que coûte une architecture, trouver les gaspillages et les optimiser.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Expliquer** ce qu'est FinOps, le **triptyque** « informer → optimiser → opérer » et la notion de **coût variable**.
2. **Identifier** les principaux postes de coût : VM, stockage, réseau, données sortantes.
3. **Repérer les gaspillages** : VM inutilisées / surdimensionnées (rightsizing), stockage orphelin, oubli de suppression.
4. **Expliquer** l'autoscaling comme outil d'optimisation et ses limites.
5. **Mettre en place** budgets, alertes de coût et monitoring (Cost Explorer), et proposer des optimisations.

---

## 2. Explication simple

### 2.1 Le « pourquoi » : le cloud facture ce que tu utilises

Contrairement à un serveur acheté une fois (capital fixe), le cloud te facture **ce que tu utilises et tant que tu l'utilises**. C'est une bénédiction (flexibilité) mais aussi un piège : sans vigilance, une ressource oubliée te facture **pour toujours**, et une VM surdimensionnée paie **du vide**.

> 💡 **Analogie** : le cloud, c'est **l'électricité** de ta maison : tu paies selon ta consommation. Personne ne laisse toutes les lumières allumées « au cas où » — pourtant, c'est exactement ce que font des équipes avec des VM qui tournent sans rien faire.

**FinOps** est la discipline (devenue standard) pour faire de la **gestion des coûts cloud** une habitude d'équipe : tout le monde (dev, ops, finance) regarde la facture et agit. Trois axes, en boucle :

```
Informe (je vois les coûts)
   ↓
Optimise (je réduis le gaspillage)
   ↓
Opère (je continue à tourner correctement, sans casser l'app)
   ↓
(recommence)
```

### 2.2 Le « comment » : les principaux postes de coût

| Poste | C'est quoi | Pourquoi ça peut coûter cher |
|-------|-----------|------------------------------|
| **VM / instances (EC2)** | Coût **à l'heure** d'exécution | Une machine qui tourne = facture qui tourne. Taille trop grande = trop cher |
| **Stockage** | Volume **stocké** (S3, disques EBS des VM) | Chaque Go compte, même sans activité ; versioning sans limite = gonflement |
| **Réseau / transfert de données** | Données **sortantes** (vers Internet) | Le trafic sortant est facturé (souvent plus que l'entrant) |
| **Bases de données (RDS)** | Instance + stockage + backup | Backups conservés à vie = coût caché |

> 🔑 **Les 3 gaspillages les plus courants** :
> 1. **VM inutilisée** : créée pour un test, oubliée, elle tourne 24h/24 → suppression après usage.
> 2. **VM surdimensionnée** : le **rightsizing** = choisir la bonne taille (une app à 10 % CPU n'a pas besoin de 32 CPU).
> 3. **Ressources orphelines** : volumes de stockage, adresses IP, snapshots, backups conservés sans raison.

### 2.3 Le « comment » (suite) : rightsizing, autoscaling, budgets

- **Rightsizing** (right = juste, size = taille) : **adapter la taille** de chaque ressource à son besoin réel. Observer (Bloc 12) pour savoir, puis réduire si la ressource est sous-utilisée.
- **Autoscaling** (vu à la Leçon 3) : **ajouter automatiquement** des instances dans les pics et **retirer** dans les creux. Résultat : on ne paie **que** la capacité utile. ⚠️ À coupler avec une **bonne taille de base** et des règles bien réglées (l'autoscaling ne corrige pas une machine déjà trop grosse).
- **Budgets et alertes** : on fixe une **limite** (ex. 10 €/mois) et le cloud **alerte** quand on l'approche (email, etc.). C'est le **réveil** qui évite les mauvaises surprises.
- **Monitoring des coûts** : **Cost Explorer** (l'outil AWS qui montre où va l'argent, par service, par tag) — l'équivalent de ta facture détaillée.

```
Budget (limite) → Monitoring des coûts (Cost Explorer) → Alerte (email à ~10 €) → Action
```

### 2.4 Le « quand » : quand appliquer FinOps ?

- **Dès la conception** d'une architecture (choisir les bonnes tailles).
- **Après chaque mise en production** (vérifier que rien ne tourne inutilement).
- **Régulièrement** : un créneau « tri des ressources » (comme on range son bureau) — on supprime, réduit, archive.

> ⚠️ **Ne pas confondre optimiser et casser** : FinOps ne consiste pas à tout éteindre. Le but est d'avoir **la bonne capacité au bon moment avec le moins de gaspillage**, sans dégrader l'expérience utilisateur.

---

## 📖 Vocabulaire / Abréviations

> Définitions d'une ligne pour ne jamais être perdu(e).

- **FinOps** : discipline de gestion et d'optimisation des coûts cloud (informe → optimise → opère).
- **Coût variable** : ce qu'on paie selon l'utilisation (vs un achat fixe).
- **Rightsizing** : choisir la **bonne taille** de ressource pour le besoin réel (ni trop, ni trop peu).
- **Autoscaling** : augmenter/réduire automatiquement les instances selon la charge (Leçon 3).
- **Budget** : une limite financière fixée à l'avance.
- **Alerte de coût** : notification (souvent par email) quand on approche/dépasse un seuil.
- **Cost Explorer** : l'outil AWS qui montre les coûts par service/tag.
- **VM inutilisée / orpheline** : ressource qui existe encore mais n'est plus utilisée (test oublié, snapshot inutile…).
- **Réseau sortant (egress)** : les données qui sortent vers Internet (facturées).
- **Tag** : étiquette (ex. `projet`, `env`) posée sur les ressources pour identifier et trier les coûts.
- **EBS** (Elastic Block Store) : le stockage attaché à une EC2, facturé même si la VM est arrêtée (souviens-toi : stopped = disque payé).

---

## 3. Exemples concrets

### 3.1 Simuler un « audit FinOps » avec un script (tout local)

```bash
# audit-finops.sh — passe en revue les gaspillages classiques (sans AWS).
echo "🔍 Audit FinOps rapide de l'architecture (leçon 8)"
echo ""
echo "1) VM : combien d'instances tournent ? (test oublié ?)"
echo "   ➜  une VM running = facture à l'heure"
echo "2) Taille : est-elle adaptée ? (rightsizing)"
echo "   ➜  si CPU moyen < 10 % avec 32 CPU → surdimensionnée"
echo "3) Stockage : des buckets/snapshots orphelins ?"
echo "   ➜  on supprime ce qui ne sert plus"
echo "4) Réseau : des transferts sortants massifs ? (egress facturé)"
echo "5) Budget : a-t-on une alerte à 80 % du budget ?  (sinon → à créer)"
```

Puis lance-le : `bash audit-finops.sh`.

### 3.2 Les réflexes en ligne de commande (avec compte configuré)

```bash
# 1. Voir les coûts récents par service (Cost Explorer) — nécessite le compte.
aws ce get-cost-and-usage \
  --time-period Start=$(date -d '-7 days' +%Y-%m-%d),End=$(date +%Y-%m-%d) \
  --granularity DAILY \
  --metrics "UnblendedCost" \
  --group-by Type=DIMENSION,Key=SERVICE

# 2. Lister les instances EC2 de tous les états pour repérer les oubliées.
aws ec2 describe-instances --query "Reservations[].Instances[].{ID:InstanceId,Type:InstanceType,Etat:State.Name}"
```

> 📌 `date -d '-7 days'` = date d'il y a 7 jours ; `--granularity DAILY` = découpage par jour ; `UnblendedCost` = coût réel. Ces commandes te montrent **où va l'argent** — le premier pas du réflexe FinOps.

### 3.3 Estimation d'une petite architecture (ordre de grandeur)

Pour t'entraîner sans rien dépenser, calcule l'estimation **à la main** d'une petite archi (tarifs indicatifs, ils changent) :

```
VM t3.micro (1 CPU / 1 Go) à l'heure           ≈ 0,011 $/h  ≈ 8 $/mois
  + 20 Go de disque EBS                        ≈ +2 $/mois
Stockage S3 (10 Go)                             ≈ +0,2 $/mois
RDS db.t3.micro (PostgreSQL)                    ≈ +12 $/mois
Total estimé ≈ 22 $/mois
```

Le but n'est pas de mémoriser les prix, mais de savoir **estimer l'ordre de grandeur** et repérer ce qui est gros (VM et RDS dominent ; le stockage « passif » est faible).

---

## 4. Bonnes pratiques modernes (2025-2026)

- **Voir avant d'agir** : Cost Explorer + **tags** systématiques (ex. `env=prod`, `projet=app`) dès la création.
- **Budgets + alertes dès le premier jour** (même petits) : alerte à 50 % et 80 %.
- **Rightsizing régulier** : revoir la taille des instances tous les trimestres, sur la base des métriques réelles (Bloc 12).
- **Autoscaling aux bons endroits** (pics de trafic) — avec des règles testées.
- **Supprimer, pas seulement arrêter** : une VM `stopped` paie encore son disque (EBS) ; les ressources orphelines se suppriment.
- **Sauvegardes bornées** : lifecycle (Leçon 4) pour ne pas garder les backups éternellement.
- **Commencer par le plus gros poste** : attaquer d'abord ce qui coûte le plus (souvent VM et bases), pas les cents économisés ailleurs.

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi c'est dangereux/inefficace | ✅ Version correcte |
|----------------|-------------------------------------|---------------------|
| VM de test qui tourne pendant des mois | Facture continue pour rien | La **supprimer** (ou l'arrêter) dès la fin du test |
| Choisir une grosse VM « au cas où » | On paie du vide (surdimensionnement) | **Rightsizing** : commencer petit, observer, adapter |
| Aucun budget / alerte | La facture explose sans prévenir | **Budget + alerte** à 50 % / 80 % |
| Garder tous les backups indéfiniment | Volume et coût gonflent sans fin | **Lifecycle** de rétention (Leçon 4) |
| Vouloir « tout éteindre » pour réduire | On casse l'application / l'expérience | **Optimiser en continu** sans dégrader le service |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`**, la correction dans **`03-correction.md`**.

**Énoncé court** : crée `audit-finops.sh` et exécute-le ; puis dans `notes-exercice-08.md` : liste les **4 postes de coût** de ton architecture, indique les **3 gaspillages probables** (VM oubliée, taille trop grosse, backups illimités), propose pour chacun une **optimisation**, et rédige un **plan budget/alerte** (seuil + réaction).

---

## 7. Correction détaillée de l'exercice

> La correction complète est dans **`03-correction.md`**. On y confronte ton audit au corrigé type, on détaille les estimations, et on valide le plan budget/alerte.

---

## 8. Checklist de validation

- [ ] J'explique FinOps et le triptyque informer / optimiser / opérer.
- [ ] Je liste les 4 postes de coût (VM, stockage, réseau, bases).
- [ ] J'identifie les gaspillages (VM inutilisée, surdimensionnement, orphelines, backups infinis).
- [ ] J'explique rightsizing, autoscaling, budgets, alertes, Cost Explorer.
- [ ] J'estime un ordre de grandeur du coût d'une petite architecture.
- [ ] Je propose des optimisations sans dégrader le service.

---

🧭 **Pont vers la suite (et fin du bloc)** — Le Bloc 6 est complet : tu sais concevoir une architecture cloud (réseau, VM, stockage, base, accès), choisir entre Lambda et EC2, et **maîtriser les coûts**. La roadmap considère le bloc acquis si tu peux « concevoir une petite architecture cloud et expliquer pourquoi chaque service existe » — c'est ce que récapitule l'**introduction du bloc** (`00-Introduction-Bloc.md`). Ce socle cloud te rend prêt pour le **Bloc 7 (Bases de données & Data Operations)**, où tu approfondiras PostgreSQL, et le **Bloc 8 (IaC)**, où tu automatiseras en code tout ce que tu viens de construire à la main.

---

*Étape finale :* `00-Introduction-Bloc.md` — l'introduction et le glossaire global du Bloc 6.
