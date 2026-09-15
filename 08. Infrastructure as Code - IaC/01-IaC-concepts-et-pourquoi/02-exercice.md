# Exercice — Leçon 1 : IaC, pourquoi et concepts

> **Bloc 8 · Leçon 1** — Exercice à faire en autonomie, **100 % local** : installation d'un outil + réflexion. Aucun compte cloud, aucun coût.

---

## Contexte

Avant d'écrire du code d'infrastructure, il faut **installer l'outil** (`terraform`) et **ancrer les concepts** (déclaratif, impératif, idempotence, drift, répartition Terraform/Ansible) : ils reviendront dans chaque leçon du bloc, et l'exercice vérifie qu'ils sont bien compris.

---

## Énoncé

> 📌 **Rappels d'options** : `apt update` rafraîchit la liste des paquets disponibles ; `apt install -y` installe un paquet (le `-y` répond « oui » d'avance aux questions) ; `--version` affiche la version d'un programme.

### Étape 1 — Installer Terraform (méthode officielle HashiCorp)

Terraform est édité par la société **HashiCorp**. Sous Ubuntu, on ajoute le dépôt officiel de HashiCorp, puis on installe comme n'importe quel paquet.

```bash
# Installe les utilitaires nécessaires (clés GPG, certificats, lsb_release).
sudo apt update && sudo apt install -y gnupg software-properties-common curl lsb-release

# Télécharge la clé publique GPG de HashiCorp et l'installe dans le trousseau système.
# (GPG sert à vérifier que les paquets viennent vraiment de HashiCorp.)
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

# Déclare le dépôt HashiCorp pour ta version d'Ubuntu (lsb_release -cs renvoie son nom de code).
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

# Rafraîchit la liste des paquets, puis installe Terraform.
sudo apt update && sudo apt install -y terraform

# Vérifie l'installation : un numéro de version doit s'afficher.
terraform -version
```

> 💡 **Si `sudo` n'est pas disponible** (ou si tu préfères sans dépôt) : méthode alternative, télécharge l'archive officielle et place le binaire `terraform` dans un dossier de ton `PATH` (rappel du Bloc 2 : le `PATH` est la liste des dossiers où le shell cherche les programmes).

### Étape 2 — Premiers contacts

```bash
# Liste les commandes disponibles et l'aide générale.
terraform -help

# Demande l'aide de la commande "plan" (tu verras de quoi il s'agit à la Leçon 2).
terraform plan -help | head -20
```

Note dans `notes-exercice-01.md` la version installée et **une commande** repérée dans l'aide de `terraform plan`.

### Étape 3 — Réflexion (dans `notes-exercice-01.md`)

1. Définis en **une phrase + une analogie** chacun : **IaC**, **déclaratif**, **impératif**, **idempotence**, **drift**.
2. Classifie ces situations : **impératif** ou **déclaratif** ? Justifie en une ligne.
   - a) Un script Bash qui fait `apt install nginx` puis `systemctl enable nginx`.
   - b) Un fichier qui dit : « je veux une VM Ubuntu avec 2 CPU et 4 Go de RAM ». *(CPU = le processeur, le « cerveau » qui calcule ; RAM = la mémoire vive, l'espace de travail de la machine.)*
   - c) Une commande du bloc 2 : `mkdir -p projet/donnees` (crée le dossier s'il n'existe pas, ne fait rien sinon).
3. **Terraform ou Ansible ?** Pour chaque besoin, dis quel outil tu utilises et pourquoi :
   - a) Créer un réseau privé et 2 machines virtuelles chez AWS.
   - b) Installer Nginx et créer un utilisateur `deploy` sur une machine déjà existante.
4. Question piège : un collègue a modifié une règle de pare-feu **directement dans la console web** de ton cloud, alors que ton code IaC n'en parle pas. Comment s'appelle cette situation, et quel risque cela crée-t-il pour la prochaine exécution de ton code ?

---

## Livrable

`notes-exercice-01.md` avec : la version de Terraform, une commande repérée dans l'aide, les 5 définitions + analogies, la classification a/b/c, le duo Terraform/Ansible, et la réponse à la question piège.

Correction détaillée dans **`03-correction.md`**.