# Leçon 1 — IaC : pourquoi décrire son infrastructure en code

> **Bloc 8 · Infrastructure as Code (IaC)** — Leçon 1 sur 9
> 🧭 **Pont depuis le Bloc 7 (Bases de données)** : au Bloc 7, tu as mis une base PostgreSQL en production **à la main** — commandes tapées une par une, fichiers de configuration édités, sauvegardes planifiées, le tout consigné dans un **runbook** (ton « carnet de conduite » qui décrit chaque étape). Tout fonctionne… mais si le serveur disparaît demain, il faut tout **refaire à la main** en suivant ton carnet, avec le risque d'oublier ou de se tromper. Ce Bloc 8 répond exactement à ce problème : transformer ton carnet en **code exécutable**. Cette première leçon pose les fondations : qu'est-ce que l'Infrastructure as Code, pourquoi elle a révolutionné le métier DevOps, quels outils existent, et l'installation de **Terraform** qui servira dans toutes les leçons suivantes.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Expliquer** pourquoi configurer une infrastructure « à la main » pose problème, et ce que l'**IaC** apporte.
2. **Distinguer** les deux styles d'automatisation : **déclaratif** et **impératif**, avec une analogie.
3. **Définir** l'**idempotence** et le **drift** — deux mots que tu entendras partout en IaC.
4. **Situer** les outils du marché : le duo **Terraform + Ansible**, et les mentions à connaître (CloudFormation, Pulumi, Puppet, Chef, Salt).
5. **Installer et vérifier** l'outil `terraform` sur ta machine, pour les leçons suivantes.

---

## 2. Explication simple

### 2.1 Le « pourquoi » : construire à la main ne marche qu'une fois

Reprenons le fil. Au **Bloc 6**, tu as créé (ou simulé) un réseau cloud, des machines virtuelles, du stockage. Au **Bloc 7**, tu as installé et sécurisé une base PostgreSQL **en tapant des commandes**. Chaque fois, tu as suivi une suite d'actions manuelles.

Cette façon de travailler a **quatre problèmes**, bien réels :

1. **La lenteur** : refaire une installation complète prend des heures ; un code informatique prend des minutes.
2. **L'erreur humaine** : à la main, on oublie une étape, on tape une mauvaise option. Ton runbook du Bloc 7 réduit ce risque, mais il reste **lu par un humain, exécuté par un humain**.
3. **La non-reproductibilité** : deux serveurs configurés « à la main » ne sont **jamais** rigoureusement identiques (« ça marche sur le serveur A mais pas sur le B » — tu as peut-être déjà vécu ça en développement).
4. **La piste d'audit perdue** : qui a changé quoi, et quand ? Avec des commandes manuelles, personne ne sait.

> 💡 **Analogie** : c'est un **cuisinier qui refait un plat de mémoire**. La première fois, c'est bon. La dixième fois, dans une autre cuisine, avec d'autres ingrédients, le résultat sera différent. La solution du restaurant : une **recette écrite** — précise, reproductible, transmissible. L'IaC, c'est **la recette écrite de ton infrastructure**.

**À quel moment a-t-on besoin de l'IaC ?** Dès que l'infrastructure doit être **recréée** (panne, nouveau serveur), **partagée** (l'équipe doit pouvoir la comprendre) ou **dupliquée** (un environnement de test identique à la production). En pratique : dès qu'on fait du sérieux — c'est pourquoi la roadmap la place ici, juste après avoir tout fait à la main.

### 2.2 Le « comment » : écrire l'infrastructure dans des fichiers texte

L'**Infrastructure as Code (IaC)** consiste à décrire ton infrastructure — machines, réseau, bases de données, règles de pare-feu — dans des **fichiers texte**, comme du code de programmation. Un **outil spécialisé** lit ces fichiers et **crée ou modifie réellement** l'infrastructure pour qu'elle corresponde à ce que tu as écrit.

```
Ton code IaC (fichiers .tf)   →   L'outil IaC (Terraform)   →   Infrastructure réelle
« je veux 1 VM, 1 base,           lit le code, compare avec      le cloud crée vraiment
1 pare-feu »                      ce qui existe déjà             ces ressources
```

Trois bénéfices immédiats, et ils découlent logiquement de la définition :

- **Reproductible** : le même fichier crée la même infrastructure, dix fois de suite.
- **Révisable** : les fichiers sont dans **Git** (Bloc 4) — chaque changement est historisé, relu, validé par l'équipe.
- **Automatisable** : un pipeline CI/CD (Bloc 11) pourra appliquer ton code sans intervention humaine.

### 2.3 Le « quoi » : déclaratif vs impératif

Il y a **deux styles** pour écrire de l'automatisation, et la distinction est fondamentale :

| Style | Ce que tu écris | Analogie | Exemple |
|-------|-----------------|----------|---------|
| **Impératif** | La **liste des gestes** à faire, dans l'ordre | Une **recette de cuisine** : « épluche, puis coupe, puis cuit 20 min » | Un script Bash : `apt install…`, puis `cp…`, puis `systemctl start…` |
| **Déclaratif** | Le **résultat final** voulu | Le **plan d'un architecte** : « maison avec 3 pièces, 2 fenêtres au sud » — pas une phrase sur *comment* construire | Un fichier Terraform : « je veux une VM de type X avec ce disque » |

> 💡 **Analogie** : avec un **plan d'architecte**, tu ne dis jamais « creuse les fondations, puis monte le mur ». Tu décris la maison finie ; le constructeur se débrouille des étapes. Avec une **recette**, si tu interromps puis relances, tu recommences depuis le début — et tu peux te retrouver avec deux fois le même ingrédient.

**Pourquoi le déclaratif est-il le standard de l'IaC ?** Parce que l'outil peut **comparer** ce que dit le **code** et ce qui **existe réellement**. De cette comparaison, il déduit tout seul la liste des gestes : créer, modifier, ou supprimer. Toi, tu ne décris que la cible.

### 2.4 L'idempotence : appliquer deux fois, c'est le même résultat

L'**idempotence** est la propriété d'une opération qui, répétée, donne **le même résultat** qu'une seule fois.

> 💡 **Analogie** : « assure-toi que la casserole contient 1 litre d'eau ». Si elle en contient déjà 1 litre : **rien à faire**. Si elle en contient 0,5 : complète. C'est idempotent. À l'inverse, « ajoute 1 litre d'eau » est **non idempotent** : répété, tu débordes la casserole.

En IaC, c'est précieux : tu peux relancer ton code le soir « pour être sûr », sans risquer de créer une deuxième base de données par accident. Le code déclaratif (Terraform) est **idempotent par conception** ; les scripts Bash (Bloc 3) ne le sont généralement **pas** (relancer `apt install nginx` deux fois est bénin, mais relancer `useradd toto` deux fois échoue).

### 2.5 Le drift : quand le réel s'écarte du code

Le **drift** (dérive, en français) désigne l'écart qui se creuse entre **ce que ton code décrit** et **ce qui tourne réellement**. Cause n°1 : quelqu'un a modifié la console web du cloud à la main, sans passer par le code.

> 💡 **Analogie** : le **plan de ta maison** dit mur blanc… mais un occupant a repeint le mur en bleu sans toucher au plan. Le plan et la réalité divergent : la prochaine fois que quelqu'un suit le plan, il aura une surprise.

Le remède est simple : **tout changement passe par le code**, la console web ne servant qu'à **observer**.

### 2.6 Les outils : le duo Terraform + Ansible, et les mentions à connaître

Il existe plusieurs outils IaC, mais en 2025-2026, le marché se résume à un **duo** aux rôles complémentaires :

- **Terraform** décrit et crée l'**infrastructure** : le réseau, les machines virtuelles, les bases managées, les règles de pare-feu. Il parle **au cloud** (« crée-moi une VM »).
- **Ansible** configure **l'intérieur des machines** : installer Nginx (nginx : le serveur web installé aux Blocs 2 et 6), créer des utilisateurs, déployer une application. Il parle **aux machines** (par SSH — la Leçon 5 du Bloc 2, déjà !).

Le flux complet que ce bloc va te faire vivre :

```
Terraform  →  crée le réseau, la VM, la base   (l'infrastructure)
    ↓
Ansible    →  installe/configure sur la VM      (la configuration)
    ↓
Docker     →  exécute l'application             (Bloc 9)
```

Les autres outils : tu dois savoir les **définir en une phrase**, rien de plus.

- **AWS CloudFormation** : l'équivalent de Terraform mais **propriétaire AWS** (ne marche que chez AWS, écrit en JSON/YAML — deux formats de fichiers texte vus au Bloc 3, Leçon 4).
- **Pulumi** : alternative à Terraform où tu écris l'infra dans un **vrai langage de programmation** (Python, TypeScript) au lieu d'un langage dédié.
- **Puppet, Chef, Salt** : anciens concurrents d'Ansible en **gestion de configuration** — encore présents dans de vieilles infrastructures, en déclin.

> 🔑 **À retenir** : ce bloc apprend **Terraform en profondeur** (le standard du marché, multi-cloud) puis **Ansible**. Les autres sont des mentions.

---

## 📖 Vocabulaire / Abréviations

| Terme | Définition (une ligne) | Où |
|-------|------------------------|-----|
| **IaC** | *Infrastructure as Code* : décrire l'infrastructure dans des fichiers texte qu'un outil exécute | § 2.2 |
| **Runbook** | Ton carnet de conduite du Bloc 7 : le document qui décrit chaque étape manuelle | Pont |
| **Déclaratif** | On décrit le résultat final voulu, l'outil calcule les gestes | § 2.3 |
| **Impératif** | On décrit la liste des gestes, dans l'ordre | § 2.3 |
| **Idempotence** | Propriété d'une opération qui, répétée, donne le même résultat | § 2.4 |
| **Drift** | Écart entre l'infrastructure décrite dans le code et la réalité | § 2.5 |
| **Terraform** | L'outil IaC déclaratif qui crée l'infrastructure chez le cloud | § 2.6 |
| **Ansible** | L'outil qui configure l'intérieur des machines, via SSH | § 2.6 |
| **HCL** | *HashiCorp Configuration Language* : le langage des fichiers Terraform (vu en détail en Leçon 2) | — |
| **State** | Le « registre » de Terraform : ce qu'il a déjà créé (approfondi en Leçon 3) | — |
| **GPG** | Système de clés publiques pour vérifier l'origine de fichiers/paquets | Exercice |
| **OpenTofu** | Le « jumeau » open source de Terraform, compatible (encart § 3.4) | § 3.4 |

*(Rappel des acquis cités dans cette leçon : SSH = connexion sécurisée à distance, Bloc 2 Leçon 5 · JSON/YAML = formats de données texte, Bloc 3 Leçon 4 · Git = gestion de versions, Bloc 4.)*

---

## 3. Exemples concrets

Maintenant que les concepts sont posés, on passe à la pratique : l'installation de Terraform, puis un premier coup d'œil sur un fichier de code IaC.

### 3.1 Installer Terraform (Ubuntu)

```bash
# Installe les utilitaires requis : gnupg (vérification des paquets), curl (téléchargement), lsb_release (nom de code Ubuntu).
sudo apt update && sudo apt install -y gnupg software-properties-common curl lsb-release

# Télécharge la clé publique de HashiCorp et la place dans le trousseau système.
# |  = envoi du résultat au programme suivant ; gpg --dearmor = conversion en format binaire du trousseau.
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

# Ajoute le dépôt HashiCorp à la liste des sources de paquets.
# $(lsb_release -cs) = insère le nom de code de ta version d'Ubuntu (ex. "noble").
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

# Rafraîchit la liste, puis installe Terraform (-y = répond "oui" automatiquement).
sudo apt update && sudo apt install -y terraform

# Vérifie : affiche la version installée.
terraform -version
```

### 3.2 Un premier aperçu de code IaC (HCL)

Voilà à quoi ressemble un fichier Terraform (langage **HCL**, expliqué en Leçon 2 — ici, lis juste les commentaires pour saisir l'esprit **déclaratif**) :

```hcl
# "resource" = "je veux cette brique d'infrastructure".
# "local_file" = le type de brique : un fichier sur disque. "salutations" = son nom dans le code.
resource "local_file" "salutations" {
  # Ce que le fichier DOIT contenir : le résultat voulu, pas les gestes.
  content  = "Bonjour, mon infrastructure est du code !"
  # Le chemin du fichier voulu.
  filename = "salutations.txt"
}
```

Ce code ne dit **jamais** « ouvre un fichier, écris dedans, ferme-le » (impératif) : il dit « ce fichier doit exister avec ce contenu » (déclaratif). Terraform se charge du comment.

### 3.3 Vérifier l'installation

```bash
# Affiche la version de Terraform et confirme que le programme est trouvable dans le PATH.
terraform -version

# Affiche l'aide générale : la liste des commandes disponibles.
terraform -help
```

### 3.4 Encart — Terraform et OpenTofu (à connaître en 2025-2026)

En 2023, Terraform a changé de licence (elle n'est plus entièrement libre). La communauté a créé **OpenTofu** : un « jumeau » **open source** et **compatible** (mêmes fichiers, mêmes commandes). En 2025-2026, les deux coexistent : on apprend **Terraform** (le plus répandu) et tout ce que tu écriras fonctionne aussi avec `tofu` à la place de `terraform`. Pas besoin d'aller plus loin pour ce bloc.

---

## 4. Bonnes pratiques modernes (2025-2026)

- **Tout le code IaC dans Git** : l'infrastructure est du code, elle a donc le même traitement — dépôt, historique, revues par les pairs. Un changement d'infra se voit dans les commits (Bloc 4).
- **La console web pour observer, jamais pour modifier** : tout changement passe par le code. C'est le seul moyen d'éviter le drift (§ 2.5).
- **Déclaratif plutôt qu'impératif** : pour l'infrastructure, préfère Terraform aux scripts Bash maison — l'idempotence est déjà gérée pour toi.
- **Épingler les versions** : indiquer la version de l'outil et des extensions utilisées, pour que le code reste reproductible dans le temps (tu verras le mécanisme `required_providers` en Leçon 2).
- **Connaître OpenTofu** : la communauté open source s'y réfuge ; le savoir « définit en une phrase » suffit à ce stade.

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi c'est dangereux/inefficace | ✅ Version correcte |
|-----------------|--------------------------------------|---------------------|
| Modifier le cloud dans la console web, en production | Crée du **drift** : ton code ne reflète plus la réalité, la prochaine exécution peut casser ce que tu avais fait à la main | Changer le **code**, puis appliquer ; la console sert seulement à observer |
| Réécrire l'installation d'un serveur en gros script Bash impératif | Pas idempotent, se casse à la 2ᵉ exécution, aucun suivi des changements | Décrire le résultat en **code déclaratif** (Terraform) |
| Garder un simple runbook comme unique documentation | Lu et exécuté par un humain : lent, sujet aux oublis, jamais à jour | Le **code IaC devient la documentation vivante** ; le runbook reste pour les procédures d'urgence |
| Apprendre tous les outils IaC en même temps (Terraform + Pulumi + Ansible + …) | Surcharge, aucune maîtrise | Le duo **Terraform + Ansible** en profondeur, les autres en une phrase |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`**, la correction dans **`03-correction.md`**.

**Énoncé court** : installe Terraform (dépôt officiel HashiCorp), vérifie avec `terraform -version` et explore l'aide. Puis, dans `notes-exercice-01.md` : les 5 définitions (IaC, déclaratif, impératif, idempotence, drift) avec une analogie chacune, une classification impératif/déclaratif de 3 situations, le duo Terraform/Ansible appliqué à 2 besoins, et une question piège sur le drift.

---

## 7. Correction détaillée de l'exercice

> La correction complète est dans **`03-correction.md`**. On y vérifie l'installation, puis on corrige la réflexion — notamment la question piège sur le drift, qui prépare la Leçon 3 (le state, là où Terraform note ce qu'il a créé).

---

## 8. Checklist de validation

- [ ] J'explique les 4 problèmes de la configuration manuelle (lenteur, erreur, non-reproductibilité, pas d'audit).
- [ ] Je définis l'IaC et ses 3 bénéfices (reproductible, révisable, automatisable).
- [ ] Je distingue déclaratif et impératif avec une analogie.
- [ ] Je définis l'idempotence et le drift, et je sais comment éviter le drift.
- [ ] J'explique qui fait quoi entre Terraform et Ansible, et je définis CloudFormation/Pulumi/Puppet/Chef/Salt en une phrase.
- [ ] Terraform est installé sur ma machine (`terraform -version` répond).

---

🧭 **Pont vers la suite** — Tu sais **pourquoi** l'IaC existe et tu as l'outil. Mais concrètement, à quoi ressemble un **projet Terraform** complet, et que font les commandes `init`, `plan`, `apply`, `destroy` ? C'est la **Leçon 2** — et pour pratiquer gratuitement, tu vas créer ta toute première « infrastructure »… des simples fichiers sur ton disque. Le code est identique à celui du cloud, seul le « traducteur » change.

---

*Prochaine étape :* Leçon 2 — **Terraform : ton premier projet** dans `02-Terraform-bases-et-workflow/`.