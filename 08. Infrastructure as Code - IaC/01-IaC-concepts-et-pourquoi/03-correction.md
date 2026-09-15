# Correction — Leçon 1 : IaC, pourquoi et concepts

> **Bloc 8 · Leçon 1** — Correction pas à pas.

---

## Étape 1 — Installer Terraform

```bash
sudo apt update && sudo apt install -y gnupg software-properties-common curl lsb-release
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install -y terraform
terraform -version
```

**Explication pas à pas** :
1. La 1ʳᵉ ligne installe les **outils prérequis** : `gnupg` (vérification de clés), `curl` (téléchargement), `lsb-release` (donne le nom de code d'Ubuntu), `software-properties-common`.
2. La 2ᵉ télécharge la **clé publique** de HashiCorp. C'est une clé de vérification : elle prouve que les paquets « Terraform » que tu installeras viennent bien de HashiCorp et pas d'un imitateur malveillant. `wget -O -` télécharge et envoie le contenu sur la sortie ; le `|` le transmet à `gpg --dearmor` (conversion de format), qui écrit dans le trousseau système.
3. La 3ᵉ **déclare le dépôt** : `$(lsb_release -cs)` est remplacé par ton nom de code Ubuntu (ex. `noble` pour la 24.04). `tee` écrit ce texte dans le fichier listé.
4. La 4ᵉ rafraîchit la liste des paquets, puis installe `terraform`.
5. `terraform -version` doit afficher par ex. `Terraform v1.13.x`.

**Si `terraform: command not found`** : le programme n'est pas trouvé dans le `PATH` (Bloc 2 : la liste des dossiers où le shell cherche les programmes). Vérifie `which terraform` ; en dernier recours, méthode binaire : télécharge l'archive sur releases.hashicorp.com, décompresse, place le binaire dans `~/.local/bin` (dossier qui doit être dans le `PATH`).

## Étape 2 — Premiers contacts

```bash
terraform -help
terraform plan -help | head -20
```

**Explication** : `-help` liste les commandes ; `plan -help` détaille une commande (les `| head -20` limitent l'affichage aux 20 premières lignes — `head` vu au Bloc 2, Leçon 2). Une commande typique repérée : `-out=FILE` (le plan peut être sauvegardé dans un fichier — utile plus tard).

## Étape 3 — Réflexion

### 1. Définitions + analogies

| Terme | Définition | Analogie |
|-------|------------|----------|
| **IaC** | Décrire l'infrastructure dans des fichiers texte qu'un outil exécute | La **recette écrite** de ton infrastructure |
| **Déclaratif** | Décrire le résultat voulu ; l'outil calcule les gestes | Le **plan d'architecte** |
| **Impératif** | Décrire la liste des gestes, dans l'ordre | La **recette de cuisine** pas à pas |
| **Idempotence** | Répéter l'opération donne le même résultat | « Assure-toi que la casserole contient 1 litre » |
| **Drift** | Écart entre le code et la réalité | Le mur repeint en bleu **sans toucher au plan** |

### 2. Classification impératif / déclaratif

- **a) Script Bash** `apt install nginx` + `systemctl enable` → **impératif** : une liste de gestes, dans l'ordre. (Bonus : le 2ᵉ `systemctl enable` est idempotent par hasard, mais le script dans son ensemble ne dit rien du résultat voulu.)
- **b) Fichier « je veux une VM 2 CPU / 4 Go »** → **déclaratif** : le résultat final, sans les gestes.
- **c) `mkdir -p projet/donnees`** → **déclaratif** (cas subtil !) : l'option `-p` signifie « crée s'il n'existe pas, ne se plaint pas s'il existe ». La commande décrit un **état voulu** (« ce dossier existe ») et est **idempotente**. C'est l'exception impérative-dans-la-syntaxe qui se comporte en déclaratif.

### 3. Terraform ou Ansible ?

- **a) Créer un réseau et 2 VM chez AWS** → **Terraform** : on parle **au cloud** pour créer des ressources qui n'existent pas encore. C'est du provisionnement d'infrastructure.
- **b) Installer Nginx et créer un utilisateur sur une machine existante** → **Ansible** : on parle **à une machine** pour configurer son intérieur. Terraform ne sait pas « installer Nginx dans » une machine ; c'est le rôle de la gestion de configuration.

### 4. Question piège : le drift

C'est du **drift** : la réalité (pare-feu modifié à la main) s'écarte du code (qui ne décrit pas cette règle). **Risque concret** : à la prochaine exécution de ton code Terraform, il verra « ce que je gère » selon son registre — s'il ne connaît pas cette règle, il peut la **laisser** (trou de sécurité invisible pour l'équipe), ou au contraire la **supprimer** si la ressource tombe dans sa zone de gestion, **cassant le collègue**. Dans les deux cas, personne ne sait que la règle existe. Remède : **tout changement passe par le code**, la console ne sert qu'à observer.

---

## Checklist de validation (leçon 1)

- [ ] J'explique les 4 problèmes de la configuration manuelle (lenteur, erreur, non-reproductibilité, pas d'audit).
- [ ] Je définis l'IaC et ses 3 bénéfices (reproductible, révisable, automatisable).
- [ ] Je distingue déclaratif et impératif avec une analogie.
- [ ] Je définis l'idempotence et le drift, et je sais comment éviter le drift.
- [ ] J'explique qui fait quoi entre Terraform et Ansible, et je définis CloudFormation/Pulumi/Puppet/Chef/Salt en une phrase.
- [ ] Terraform est installé sur ma machine (`terraform -version` répond).

---

## 🧠 Conseils pour la suite

- Garde cette analogie filée : **plan d'architecte** (déclaratif) vs **recette** (impératif) — elle sera réutilisée dans chaque leçon du bloc.
- La question du drift t'a montré un trou : « comment Terraform sait-il ce qu'il a déjà créé ? ». La réponse — le **state** — est la Leçon 3, mais tu vas déjà la **rencontrer** en Leçon 2 : le fichier `terraform.tfstate` apparaîtra tout seul après ton premier `apply`.
- En Leçon 2, on pratique **en local, gratuitement** : le même code HCL servira au cloud en Leçon 5.