# Leçon 6 — Ansible : bases, inventaire et premier playbook

> **Bloc 8 · Infrastructure as Code (IaC)** — Leçon 6 sur 9
> 🧭 **Pont depuis la Leçon 5** : Terraform vient de créer une machine Ubuntu **neuve et vide** — comme au Bloc 6, mais cette fois depuis le code. Une machine vide ne sert à rien : il faut **installer et configurer à l'intérieur** (Nginx, utilisateurs, application). C'est exactement le travail que tu as fait **à la main** aux Blocs 2 et 7 — et le rôle d'**Ansible**, annoncé depuis la Leçon 1 : le duo Terraform crée, **Ansible configure**. Cette leçon pose ses bases : comment Ansible parle aux machines (SSH, rappel du Bloc 2, Leçon 5), l'**inventaire** (la liste des machines), les **commandes ad-hoc** (une action rapide), et ton premier **playbook** idempotent — pratiqué d'abord sur `localhost`, sans risque.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Installer** Ansible et **expliquer** son architecture « sans agent » (SSH à la place d'un logiciel installé sur la cible).
2. **Créer et utiliser** un **inventaire** (la liste des machines, en groupes).
3. **Exécuter** des commandes **ad-hoc** (`ansible all -m ping`, `-m apt`) et savoir quand les utiliser (diagnostic) ou pas (répétable → playbook).
4. **Écrire** un **playbook** YAML (rappel : le format vu au Bloc 3, Leçon 4) avec des **tasks** et des **modules** dédiés.
5. **Vérifier l'idempotence** : relancer un playbook et observer `changed=0` — le lien direct avec la Leçon 1.

---

## 2. Explication simple

### 2.1 Le « pourquoi » : la moitié « configuration » du duo

Rappelle-toi le schéma de la Leçon 1 :

```
Terraform  →  crée le réseau, la VM, la base   (l'infrastructure)
    ↓
Ansible    →  installe/configure sur la VM      (la configuration)
```

Pourquoi deux outils ? Parce que ce sont **deux métiers différents** : Terraform parle aux **API du cloud** (« donne-moi une machine ») ; Ansible parle **aux machines elles-mêmes** (« installe Nginx, crée l'utilisateur deploy, démarre le service »). Au Bloc 7, tu as fait tout cela **en tapant les commandes une à une** sur le serveur : Ansible automatise ce travail, de façon **idempotente** (Leçon 1) et **reproductible** (défini dans un fichier versionné dans Git).

### 2.2 Le « comment » : l'architecture sans agent

Ansible fonctionne avec une **machine de contrôle** (ton poste) qui se connecte aux **machines cibles** (les serveurs à configurer) :

```
Ton poste (contrôle)                 Les cibles
┌────────────────────┐   SSH   ┌──────────────┐
│  Ansible           │ ───────▶│ serveur web  │
│  - inventaire      │         ├──────────────┤
│  - playbooks       │ ───────▶│ serveur BDD  │
│  - modules         │         └──────────────┘
└────────────────────┘
```

La particularité d'Ansible : **« sans agent »**. Aucun logiciel spécial n'est installé sur les cibles : Ansible utilise simplement **SSH** (rappel du Bloc 2, Leçon 5 : la connexion sécurisée à distance) et exécute des petits programmes Python envoyés à la volée — Python étant déjà présent sur toute Ubuntu (Bloc 3).

> 💡 **Analogie** : Ansible est un **majordome avec un carnet d'instructions** (le playbook). Il va de chambre en chambre (les machines, par le « téléphone interne » SSH) et vérifie chaque consigne : « la plante est-elle arrosée ? » — si oui, **il ne fait rien** ; si non, il l'arrose. C'est l'idempotence : la consigne décrit l'**état voulu**, pas le geste.

Le vocabulaire, avant de pratiquer :

| Terme | Définition | Analogie |
|-------|------------|----------|
| **Machine de contrôle** | Ton poste, où tu lances `ansible` | Le majordome |
| **Cible (host)** | La machine à configurer | La chambre |
| **Inventaire** | Le fichier listant les cibles, en groupes | La liste des chambres |
| **Playbook** | Le fichier YAML des consignes, dans l'ordre | Le carnet d'instructions |
| **Task** | Une consigne du playbook | Une ligne du carnet |
| **Module** | L'outil spécialisé d'une consigne (`apt`, `file`, `service`…) | L'ustensile de chaque tâche |
| **Ad-hoc** | Une commande directe, sans playbook | Un ordre de dernière minute |
| **Fact** | Une information collectée sur une cible (OS, IP…) | Le rapport d'inspection |

### 2.3 Le « quoi » : inventaire et modules

**L'inventaire** est un simple fichier texte, au format **INI** (le format « clé = valeur par sections » que tu as déjà croisé dans les fichiers de config du Bloc 2) :

```ini
[serv_web]
serveur-1 ansible_host=192.168.1.10   # une cible : alias + adresse

[serv_bdd]
serveur-2 ansible_host=10.0.2.5

[app:children]     # un groupe composé d'autres groupes
serv_web
serv_bdd
```

**Les modules** sont les « verbes » d'Ansible — et c'est là que se cache son secret d'idempotence : chaque module **sait vérifier l'état actuel** avant d'agir. `apt` (gérer les paquets), `file` (fichiers/dossiers), `user` (utilisateurs), `service` (services systemd, Bloc 2, Leçon 4), `copy` (déposer des fichiers), `command`/`shell` (commandes libres — **à éviter**, voir § 5)…

**Les commandes ad-hoc** pour tester vite :

```bash
ansible groupe -m ping          # "les cibles du groupe répondent-elles ?"
ansible groupe -a "uptime"      # une commande libre, ponctuelle
```

### 2.4 Le « quand » : ad-hoc vs playbook

- **Ad-hoc** : diagnostic, une action ponctuelle « à ne jamais refaire » (et si tu la referais, ce serait un playbook…).
- **Playbook** : tout ce qui doit être **reproductible** — en pratique, 95 % du travail Ansible. La règle du métier : *si tu l'écris deux fois en ad-hoc, écris-le en playbook*.

**Dans ce bloc, la pratique se fait sur `localhost`** (ta machine, traitée comme une cible spéciale avec `ansible_connection=local` — pas de SSH, actions locales). Pourquoi ? Zéro risque, zéro machine nécessaire ; et la logique (inventaire, playbook, modules, idempotence) est **identique** à celle d'un serveur distant. En Leçon 7, tu cibleras une vraie machine (EC2 de la Leçon 5 ou une VM locale).

---

## 📖 Vocabulaire / Abréviations

| Terme | Définition (une ligne) | Où |
|-------|------------------------|-----|
| **Ansible** | L'outil de configuration des machines, via SSH, sans agent | § 2.2 |
| **Machine de contrôle** | Ton poste, d'où on lance `ansible` | § 2.2 |
| **Cible (host)** | La machine à configurer | § 2.2 |
| **Inventaire** | Le fichier listant les cibles et leurs groupes | § 2.3 |
| **INI** | Format « section + clé = valeur » des fichiers de config | § 2.3 |
| **Playbook** | Le fichier YAML avec les consignes ordonnées | § 2.2 |
| **Play** | Un « chapitre » du playbook : cible + réglages + tasks | § 3.3 |
| **Task** | Une consigne unitaire | § 2.2 |
| **Module** | L'outil spécialisé d'une task (`apt`, `file`, `service`…) | § 2.3 |
| **Ad-hoc** | Commande directe sans playbook | § 2.4 |
| **Fact** | Une info collectée sur une cible (OS, IP, mémoire…) | § 2.2 |
| **`ansible_connection=local`** | Indique « exécute directement ici, sans SSH » | § 2.4 |
| **`become`** | Exécuter avec les droits root/sudo | § 3.3 |
| **`--check`** | Mode simulation : montre sans faire (l'équivalent du `plan`) | § 3.3 |
| **YAML** | Le format des playbooks (Bloc 3, Leçon 4) | § 3.3 |

---

## 3. Exemples concrets

La théorie est posée ; on installe et on pratique sur `localhost`, chaque commande commentée ligne par ligne.

### 3.1 Installer Ansible

```bash
# Installation recommandée en 2025 : via pip, pour ton compte.
# --user  = installe dans ton dossier (~/.local), pas dans le système.
# --upgrade = met à jour si une version existe déjà.
python3 -m pip install --user --upgrade ansible

# Vérifie : version + chemin de config.
ansible --version
```

Sortie attendue (extrait) : `ansible [core 2.17.x]` et un chemin de configuration. Si `ansible: command not found` → ajoute `~/.local/bin` dans le `PATH` (déjà vu au Bloc 6, Leçon 1).

> 📌 **Deux commandes à ne pas confondre** : `ansible` (une commande ad-hoc, une action) et `ansible-playbook` (exécuter un playbook entier). On les découvre toutes les deux juste après.

### 3.2 L'inventaire

```bash
# Crée le dossier de travail du bloc Ansible.
mkdir -p ~/ansible-debut && cd ~/ansible-debut

# Crée l'inventaire avec nano.
nano inventory.ini
```

```ini
[local]
localhost ansible_connection=local
```

**Explication ligne par ligne** : `[local]` crée un **groupe** nommé `local` ; `localhost` est l'**alias** de la cible ; `ansible_connection=local` dit « ne passe pas par SSH : agis directement sur cette machine » — indispensable pour pratiquer sur ton poste. (Sans cette ligne, Ansible tenterait un SSH vers lui-même.)

En production, ce même fichier listerait les serveurs réels :

```ini
[serv_web]
atelier-app ansible_host=203.0.113.10 ansible_user=admin   # une EC2 de la Leçon 5 !

[serv_bdd]
atelier-bdd ansible_host=10.0.2.5 ansible_user=admin
```

### 3.3 Le premier playbook — `01-nginx.yml`

```bash
# Crée le playbook avec nano.
nano 01-nginx.yml
```

```yaml
---
# YAML (Bloc 3, Leçon 4) : les "---" ouvrent le document ; l'indentation (2 espaces) est vitale.

# Un "play" : un chapitre = cible + réglages + liste de tasks.
- name: Preparer le serveur web (localhost)     # titre du play (lisibilité)
  hosts: local                                  # la cible : le groupe de l'inventaire
  become: true                                  # sudo pour installer des paquets

  tasks:                                        # la liste des consignes, DANS L'ORDRE
    # Task 1 : le module "apt" gère les paquets Debian/Ubuntu.
    - name: Installer nginx                     # titre de la task
      ansible.builtin.apt:                      # le module (les "built-in" = fournis d'origine)
        name: nginx                             # quel paquet
        state: present                          # L'ÉTAT VOULU : "présent" (déclaratif !)
        update_cache: true                      # rafraîchit la liste des paquets d'abord

    # Task 2 : le module "service" gère les services systemd (Bloc 2, Leçon 4).
    - name: Demarrer et activer nginx
      ansible.builtin.service:
        name: nginx
        state: started                          # "démarré" : le démarre s'il est arrêté
        enabled: true                           # "activé au boot" (comme systemctl enable)

    # Task 3 : le module "copy" dépose du contenu dans un fichier.
    - name: Deposer la page d'accueil
      ansible.builtin.copy:
        content: "Serveur prepare par Ansible - lecon 6\n"   # le contenu voulu
        dest: /var/www/html/index.html          # le chemin (la page par défaut de nginx)
```

**Lis ce playbook avec les yeux de la Leçon 1** : chaque task décrit un **état voulu** (« nginx présent », « service démarré », « cette page à cet endroit ») — jamais une suite de gestes. C'est le même esprit déclaratif que Terraform, appliqué à l'intérieur des machines.

### 3.4 Exécuter : simulation, puis pour de vrai

```bash
# SIMULATION : --check montre ce qui serait fait, sans rien toucher.
# C'est l'équivalent Ansible du "terraform plan" (Leçon 2) !
ansible-playbook 01-nginx.yml -i inventory.ini --check

# POUR DE VRAI : -i = quel inventaire utiliser.
ansible-playbook 01-nginx.yml -i inventory.ini
```

Sortie attendue (résumé final) :

```
localhost : ok=3  changed=3  failed=0
```

Lecture : 3 tasks **ok**, 3 **changed** (elles ont effectivement agi : nginx installé, service démarré, page déposée), 0 en échec. Vérifie le résultat réel : `curl http://localhost` (le Bloc 5 t'a appris cette requête HTTP) affiche la page déposée.

### 3.5 L'idempotence en action

```bash
# Relance le playbook SANS rien changer.
ansible-playbook 01-nginx.yml -i inventory.ini
```

Sortie : `ok=3 changed=0 failed=0` — **rien n'a été refait** : nginx était déjà présent, le service déjà démarré, la page déjà là. C'est exactement le `No changes` de Terraform (Leçon 2) : l'idempotence vérifiée par toi-même.

```bash
# Modifie le contenu de la page (Task 3) dans le playbook, puis relance :
ansible-playbook 01-nginx.yml -i inventory.ini
# → changed=1 : SEULE la Task 3 a agi. Mise à jour ciblée, pas un chaos.
```

### 3.6 Les ad-hoc, à leur place

```bash
# Diagnostic : la machine répond ? (le "ping" d'Ansible, pas celui du réseau)
ansible local -m ping -i inventory.ini

# Le rapport d'inspection : les facts (OS, RAM, IP…).
ansible local -m setup -i inventory.ini | head -30
```

**Explication des options** : `-m` choisit le **module** ad-hoc ; `-i` l'inventaire ; `| head -30` limite l'affichage (Bloc 2). Les facts servent au diagnostic — et en Leçon 7, tu les utilisera dans les templates.

---

## 4. Bonnes pratiques modernes (2025-2026)

- **Installation via `pip --user`** : méthode recommandée depuis que le paquet système d'Ubuntu a été retiré — toujours la version actuelle.
- **Tout le code Ansible dans Git** : inventaires et playbooks sont du code — dépôt, historique, revue (Bloc 4).
- **`--check` avant d'exécuter** sur une cible inconnue : c'est le filet de sécurité du `plan`, version Ansible.
- **Un module dédié plutôt qu'une commande libre** : `apt`, `service`, `copy`, `file`… savent vérifier l'état ; `command`/`shell` non (ils s'exécutent **à chaque fois** → l'idempotence est perdue).
- **Des `name` partout** : chaque play et chaque task porte un titre lisible — c'est ce que tu lis dans les logs.
- **Inventaire versionné, secrets non** : les adresses vont dans Git ; les mots de passe passent par des variables sensibles (Leçon 7) ou un gestionnaire de secrets (Bloc 5).

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi c'est dangereux/inefficace | ✅ Version correcte |
|-----------------|--------------------------------------|---------------------|
| Tout faire en `shell: apt install nginx` | Non idempotent : le module `shell` s'exécute à chaque passage, sans vérifier l'état | Module `apt` avec `state: present` |
| Une série de commandes ad-hoc recopiées dans tes notes | Non reproductible, non relu, non versionné | Un **playbook** versionné dans Git |
| Oublier `ansible_connection=local` sur localhost | Ansible tente SSH vers sa propre machine → erreurs obscures | La ligne magique dans l'inventaire |
| Exécuter en `become: true` par réflexe général | Des droits root sans raison = risque inutile | `become` seulement sur ce qui en a besoin |
| Modifier la cible à la main après le playbook | Drift (Leçon 1) : le prochain passage rectifie… avec surprises | Tout passe par le playbook |
| Lancer sans `--check` sur une machine de prod | Tu découvriras les changements… après | `--check` d'abord, toujours sur l'inconnu |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`**, la correction dans **`03-correction.md`**.

**Énoncé court** : installe Ansible via pip, crée un inventaire avec la cible `localhost` (`ansible_connection=local`), teste le contact ad-hoc (`ping`, `setup`), écris le playbook `01-nginx.yml` (install + service + page), exécute en `--check` puis pour de vrai, **vérifie l'idempotence** (second passage `changed=0`), modifie la page et observe la mise à jour ciblée.

---

## 7. Correction détaillée de l'exercice

> La correction complète est dans **`03-correction.md`** : commandes commentées, sorties attendues, et les chiffres d'idempotence expliqués.

---

## 8. Checklist de validation

- [ ] J'explique l'architecture « sans agent » et le rôle de SSH (rappel Bloc 2, Leçon 5).
- [ ] Je crée un inventaire (groupes, `ansible_host`, `ansible_connection=local`) et je sais à quoi il sert.
- [ ] Je fais la différence `ansible` (ad-hoc) vs `ansible-playbook`, et je sais quand utiliser l'un ou l'autre.
- [ ] J'écris un playbook YAML avec des modules dédiés (`apt`, `service`, `copy`) et des états voulus.
- [ ] J'ai **vérifié l'idempotence** (second passage : `changed=0`) et je sais pourquoi les modules dédiés la garantissent.
- [ ] Je connais le duo de sécurité : `--check` (simulation) avant d'exécuter sur une cible inconnue.

---

🧭 **Pont vers la suite** — Ton playbook fonctionne… en local, sur une seule cible, avec du texte en dur. Mais un playbook professionnel doit : cibler une **vraie machine** (l'EC2 de la Leçon 5, par SSH), être **lisible et réutilisable** (variables, templates Jinja2), **réagir aux changements** (handlers : « si la config a changé, redémarre le service ») et s'**organiser en rôles** — l'équivalent Ansible des modules Terraform de la Leçon 4. C'est la **Leçon 7**.

---

*Prochaine étape :* Leçon 7 — **Ansible : playbooks avancés, templates et rôles** dans `07-Ansible-playbooks-avances-et-roles/`.