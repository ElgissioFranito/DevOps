# Leçon 9 — Projet récapitulatif : détruire et reconstruire

> **Bloc 8 · Infrastructure as Code (IaC)** — Leçon 9 sur 9 (projet final)
> 🧭 **Pont depuis la Leçon 8** : tu as conçu l'architecture qui survit aux pannes et tu sais pourquoi l'IaC est « l'accélérateur de disaster recovery ». Il ne reste qu'à **prouver** le critère d'acquis de la roadmap : *« Tu peux supprimer ton infrastructure et la reconstruire de manière reproductible »*. Ce projet fait vivre le plan DR de la Leçon 8 de bout en bout : rassembler le stack complet (Terraform + Ansible), **détruire**, puis **reconstruire depuis le code seul** — et **chronométrer**. Deux scénarios au choix : **A (100 % local, gratuit)** et **B (AWS free tier)**.

---

## 1. Objectifs du projet

À la fin de ce projet, tu seras capable de :

1. **Assembler** un projet complet : infrastructure Terraform + configuration Ansible, organisés en un dépôt lisible.
2. **Écrire le plan de reprise** (le « runbook DR ») en 6 étapes numérotées, versionné avec le code.
3. **Détruire intégralement** ton infrastructure (`terraform destroy`) sans jeter le code.
4. **Reconstruire** l'infrastructure **et** sa configuration en exécutant uniquement des commandes documentées.
5. **Chronométrer et documenter** le résultat — la preuve chiffrée du RTO réel.

---

## 2. Explication simple

### 2.1 Le « pourquoi » : transformer le critère en preuve

La roadmap clôt le bloc ainsi : *« Bloc acquis si : tu peux supprimer ton infrastructure et la reconstruire de manière reproductible. »* Tout ce que tu as construit — le langage (Leçons 2-4), le state (Leçon 3), le cloud (Leçon 5), la configuration (Leçons 6-7), la conception (Leçon 8) — sert maintenant à **un seul exercice** : la démo. C'est exactement le test trimestriel de la Leçon 8, rejoué par toi, mesuré.

**La règle du jeu est simple et stricte :**

1. **Avant** la destruction : tout le code est versionné dans Git (Bloc 4) — s'il n'est pas commité, il n'existe pas.
2. Pendant la reconstruction : tu ne peux utiliser **que** le dépôt + les commandes écrites dans ton plan de reprise. Pas d'improvisation.
3. **Après** : tu écris le rapport (durées, erreurs rencontrées, corrections apportées au code).

> 💡 **Analogie** : c'est l'exercice du **sapeur-pompier de la Leçon 8** : on ne devient pas bon en incendie en théorie — on rehausse ses compétences en rejouant le sinistre dans un cadre maîtrisé. Chaque erreur commise ici est une erreur **gratuite** qui ne se reproduira pas en vrai.

### 2.2 Le « comment » : le dépôt complet et le plan de reprise

Le livrable central est un **dépôt unique**, organisé comme le recommande la Leçon 4 :

```
bibliotheque-iac/                 ← TON dépôt (Git)
├── terraform/                    ← l'infrastructure en code
│   ├── versions.tf
│   ├── backend.tf                (le state dans S3 — scénario B ; absent en A)
│   ├── variables.tf
│   ├── main.tf
│   └── outputs.tf
├── ansible/
│   ├── inventory.ini
│   ├── site.yml
│   └── roles/serveur_web/        ← le rôle de la Leçon 7
├── docs/
│   ├── plan-reprise.md           ← le runbook DR (6 étapes, Leçon 8)
│   └── rapport-reprise.md        ← LE rapport de ce projet (les chronos)
└── README.md
```

Le **plan de reprise** (`docs/plan-reprise.md`) reprend la table en 6 étapes de la Leçon 8 — mais avec **tes commandes exactes**, prêtes à être collées. C'est lui que tu exécuteras étape par étape, et que tu corrigeras si une étape échoue (puis commiteras la correction : le plan est du code aussi).

### 2.3 Les deux scénarios

| | Scénario A — local | Scénario B — AWS (free tier) |
|---|--------------------|------------------------------|
| Infrastructure « détruite/reconstruite » | Le projet `atelier-securise` (fichiers + secrets, Leçons 2-4) | L'architecture complète du Bloc 6 (VPC, EC2, S3, RDS — Leçon 5) |
| Configuration Ansible | Le rôle `serveur_web` sur `localhost` (Leçon 7) | Le même rôle sur l'EC2, par SSH |
| Coût | Zéro | Free tier, garde-fous de la Leçon 5 (budget, destroy final) |
| Difficulté | Convient si tu n'as pas de compte AWS | C'est la preuve complète |

Les deux scénarios valident le critère : la **logique** (code → destroy → rebuild → chrono) est identique ; seul le niveau de réalisme change.

### 2.4 Le « quand » : maintenant, puis trimestriellement

Ce projet se fait **une fois pour prouver**, puis **chaque trimestre pour maintenir** (Leçon 8) : la reconstruction qui prend 20 min aujourd'hui et 15 min dans trois mois, c'est ta DR qui s'améliore. Au-delà du bloc : le dépôt devient la **base** des blocs suivants (Docker au Bloc 9 s'ajoutera au rôle Ansible ; le CI/CD du Bloc 11 exécutera ces mêmes commandes en pipeline).

---

## 📖 Vocabulaire / Abréviations

| Terme | Définition (une ligne) | Où |
|-------|------------------------|-----|
| **Stack** | L'ensemble cohérent des briques d'un projet (infra + config + docs) | § 2.2 |
| **Runbook DR** | Le plan de reprise écrit, en étapes numérotées, avec les commandes exactes | § 2.2 |
| **RTO réel** | La durée **mesurée** de reconstruction (par opposition au RTO annoncé) | § 2.2 |
| **`jq`** | Un petit outil en ligne de commande qui lit et filtre du JSON | § 3.2 |
| **`terraform output -json`** | Affiche les outputs du projet au format JSON (pour l'automatiser) | § 3.2 |
| **Ressource fantôme** | Une ressource réelle que le state ne connaît plus (Leçon 3) | Exercice |

---

## 3. Exemples concrets

Le projet étant de l'**assemblage** de ce que tu sais déjà faire, les exemples ci-dessous montrent les **points de jonction** — là où les briques se raccordent, et où se cachent les erreurs.

### 3.1 Le plan de reprise, en version prête à exécuter

```markdown
# Plan de reprise (docs/plan-reprise.md)

## Prérequis
- [ ] Le code est commité (git status propre)
- [ ] Scénario B : aws configure OK, compartiment backend OK, budget OK

## Étape 1 — Recréer l'infrastructure (chrono : __)
    cd terraform && terraform init && terraform apply

## Étape 2 — Reconfigurer les machines (chrono : __)
    cd ../ansible && ansible-playbook site.yml -i inventory.ini

## Étape 3 — Restaurer les données (scénario B, chrono : __)
    (snapshot RDS / pg_restore — Bloc 7, Leçon 4)

## Étape 4 — Re-pointer les applications
    (nouvel endpoint/IP depuis terraform output → variables)

## Étape 5 — Vérifier
    curl http://<ip>   → page attendue

## Étape 6 — RTO total : __ min (objectif <= 30)
```

**Pourquoi ce fichier est central** : le jour d'un vrai sinistre, tu n'auras ni le temps ni le calme pour improviser. Le plan **dans Git**, relu, corrigé à chaque test, EST la DR. (Il est d'ailleurs souvent exécuté par un pipeline CI/CD au Bloc 11 — la suite logique.)

### 3.2 La jonction Terraform → Ansible (l'endroit n°1 des erreurs)

La reconstruction ne serait pas « reproductible » si l'IP du serveur reconstruit devait être retrouvée à la main. Le pont :

```bash
# 1. Terraform produit l'IP publique (output ip_publique, Leçon 5).
#    terraform output -json : les outputs au format JSON (Bloc 3, Leçon 4) ;
#    jq -r : affiche la valeur brute (-r = raw, sans guillemets).
cd terraform
terraform output -json ip_publique | jq -r .
# → 203.0.113.10
# (installe jq si besoin : sudo apt install -y jq)

# 2. Ansible consomme l'IP (inventaire — Leçon 7) :
#    [serv_web]
#    atelier-app ansible_host=203.0.113.10 ansible_user=ubuntu
#    ansible_ssh_private_key_file=~/.ssh/atelier.pem

# 3. La configuration joue (rôle serveur_web).
cd ../ansible
ansible-playbook site.yml -i inventory.ini
```

**Automatisable** (selon ton niveau) : une ligne de shell qui réécrit l'inventaire depuis les outputs (`terraform output -json … | jq … > inventory.ini`). À ce stade, la version manuelle documentée suffit — l'automatisation complète arrive avec le CI/CD (Bloc 11).

> 💡 **Rappels utiles pour la reconstruction** (chacun avec sa leçon) : le state, la mémoire du chantier (Leçon 3) ; le backend S3, l'endroit où elle vit (Leçons 3 et 5) ; le rôle Ansible, la configuration idempotente (Leçon 7) ; les chronos, le RTO réel (Leçon 8).

### 3.3 Ce que « reproductible » veut dire, vérifié point par point

| Test | Commande/action | Résultat attendu |
|------|-----------------|------------------|
| Reconstruction n°1 | le plan de reprise, du début à la fin | Service rendu |
| Reconstruction n°2 (le lendemain) | le MÊME plan, sans modification | Service rendu, `No changes`/`changed=0` en fin |
| Reconstruction par un tiers | un ami (ou le futur toi) lit le README | Il exécute sans te poser de question |
| Idempotence de la config | relancer `ansible-playbook` | `changed=0` : la config est une promesse d'état |

Si les 4 tests passent : le critère de la roadmap est **prouvé par la démonstration**, pas par la mémoire.

---

## 4. Bonnes pratiques modernes (2025-2026)

- **Le plan de reprise est du code** : versionné, relu (Bloc 4), corrigé à chaque test — jamais un document hors dépôt.
- **La preuve par le chrono** : un RTO annoncé sans mesure ne vaut rien ; le rapport de reprise **daté et commité** est le standard.
- **Reconstruction testée avant le sinistre** : le stack doit se reconstruire en état « normal » d'abord ; on ne teste pas un plan pour la première fois en incendie.
- **Chaque erreur de reconstruction devient une correction commitée** : c'est ainsi qu'un plan s'améliore — et la raison pour laquelle les erreurs gratuites d'aujourd'hui valent de l'or.
- **Le dépôt unique** (Terraform + Ansible + docs) : une seule adresse pour tout reconstruire — c'est ce que le CI/CD (Bloc 11) appellera en pipeline.

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi c'est dangereux/inefficace | ✅ Version correcte |
|-----------------|--------------------------------------|---------------------|
| Détruire avec du code **non commité** | La reconstruction utilise du code qui n'existe plus nulle part | `git status` propre + commit **avant** le destroy |
| Improviser pendant la reconstruction | Ce n'est plus « reproductible », c'est de l'artisanat | Suivre le plan ; corriger le plan si besoin, puis recommencer l'étape |
| Garder l'IP/endpoint en dur dans l'inventaire | Après reconstruction, les IP changent : le plan échoue au 2ᵉ essai | `terraform output` → inventaire/variables (§ 3.2) |
| Oublier la restauration des données (scénario B) | L'infra revient, la base est vide : service faux | L'étape 3 du plan (backup, Bloc 7) est obligatoire |
| Un rapport sans chronos ni erreurs | Aucune preuve, aucune amélioration | Chronos + corrections commitées |
| Détruire le compartiment backend « pour tout nettoyer » | Adieu le state… et la mémoire de toutes tes infra (Leçon 3) | Le backend survit aux projets (quasi gratuit) |

---

## 6. L'exercice pratique

> ⚠️ Le projet complet est décrit dans **`02-exercice.md`** (phases 0 à 3), la correction dans **`03-correction.md`**.

**Résumé** : assemble le dépôt `bibliotheque-iac/` (Terraform + Ansible + plan de reprise), committe, teste une reconstruction « normale », puis détruis tout et reconstruis **uniquement** en suivant le plan — en chronométrant. Termine par le rapport (`rapport-reprise.md`) avec le RTO réel et les corrections effectuées.

---

## 7. La correction détaillée

> La correction complète est dans **`03-correction.md`** : chronos attendus, les pièges du jour J et leurs solutions, et la rélecture du critère d'acquis.

---

## 8. Checklist de validation (et critère d'acquis du bloc)

- [ ] J'ai un dépôt unique et propre : Terraform + Ansible + plan de reprise + README, tout commité.
- [ ] J'ai détruit intégralement mon infrastructure et la reconstruite **en suivant uniquement le plan**.
- [ ] La configuration Ansible s'est réappliquée idempotente (`changed=0` au 2ᵉ passage).
- [ ] J'ai mesuré le **RTO réel** et écrit le rapport (chronos + erreurs + corrections).
- [ ] Je peux refaire l'exercice le lendemain avec le même plan, sans aide.
- [ ] ✋ **Le critère de la roadmap est coché** : « Tu peux supprimer ton infrastructure et la reconstruire de manière reproductible. »

---

🧭 **Pont vers le Bloc 9 (Docker et conteneurisation)** — Regarde le flux que tu viens de prouver : Terraform crée des machines, Ansible y installe nginx… **paquet par paquet, en tapant dans le système de la machine**. C'est le dernier vestige du monde « à la main ». Le Bloc 9 change le paradigme : au lieu de configurer des machines, on **emballera l'application** (avec ses dépendances) dans des **conteneurs** identiques, exécutables partout. Le flux devient : Terraform crée → Docker exécute. Ta machine Ansible reconfigurée dix fois et ton conteneur reconstruit cent fois n'auront plus rien à envier l'un à l'autre : les deux seront reproductibles **par construction**. Tu as posé la fondation ; Docker bâtira dessus.

---

*Prochaine étape :* Bloc 9 — **Docker et conteneurisation** (le plan `09. Docker et conteneurisation/`).