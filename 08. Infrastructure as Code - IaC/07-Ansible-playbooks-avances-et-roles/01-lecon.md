# Leçon 7 — Ansible avancé : variables, templates, handlers et rôles

> **Bloc 8 · Infrastructure as Code (IaC)** — Leçon 7 sur 9
> 🧭 **Pont depuis la Leçon 6** : ton premier playbook fonctionne — mais il est **linéaire** : une seule cible (`localhost`), du texte en dur (« Serveur prepare par Ansible - lecon 6 »), aucune réaction au changement. Un playbook professionnel doit : cibler une **vraie machine par SSH** (l'EC2 de la Leçon 5, ou une VM locale), **paramétrer** au lieu de coder en dur (variables), **générer** des fichiers dynamiques (templates Jinja2), **réagir** (« si la config a changé, redémarre le service » — handlers) et s'**organiser en rôles** — l'équivalent Ansible des modules Terraform de la Leçon 4.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Cibler une machine réelle** par SSH (inventaire avec `ansible_host`, `ansible_user`, clé privée) et vérifier le contact.
2. **Utiliser des variables** Ansible (dans le play, via `-e`, via des fichiers) et connaître leur priorité à gros traits.
3. **Écrire un template Jinja2** (`.j2`) qui injecte variables et facts dans un fichier généré.
4. **Ajouter des handlers** (`notify`) : une action qui ne s'exécute que si une task a changé quelque chose.
5. **Structurer un rôle** (arborescence standard `tasks/`, `handlers/`, `templates/`, `defaults/`) et l'appeler depuis un playbook `site.yml`.

---

## 2. Explication simple

### 2.1 Le « pourquoi » : du script linéaire à l'organisation professionnelle

Regarde ton playbook de la Leçon 6 : si tu veux préparer **cinq** serveurs web avec **cinq** pages différentes, que fais-tu ? Copier le fichier cinq fois et modifier le texte en dur ? C'est le **copier-coller** que la Leçon 4 (Terraform) t'a déjà fait abandonner. Les réponses d'Ansible aux mêmes problèmes :

| Problème | Réponse Terraform (Leçon 4) | Réponse Ansible (cette leçon) |
|----------|------------------------------|-------------------------------|
| Valeurs en dur | `variable` + `.tfvars` | **Variables** (`vars`, `-e`) |
| Fichiers générés selon les valeurs | interpolation `${...}` | **Templates Jinja2** (`.j2`) |
| Réaction à un changement | mise à jour ciblée (le plan) | **Handlers** (`notify`) |
| Réutilisation de la logique | **modules** | **rôles (roles)** |

Le vocabulaire change, la philosophie est la même : **paramétrer, réutiliser, ne jamais dupliquer**.

### 2.2 Le « comment » : cibler une vraie machine par SSH

La Leçon 6 utilisait `ansible_connection=local` (exécuter ici). Pour une vraie cible, on **revient à SSH** — et tu as déjà tout ce qu'il faut (Bloc 2, Leçon 5 : connexion par **clé**, jamais par mot de passe) :

```ini
[serv_web]
atelier-app ansible_host=203.0.113.10 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/atelier.pem
```

Explication ligne par ligne : `ansible_host` = l'adresse IP (celle de l'output `ip_publique` de la Leçon 5) ; `ansible_user=ubuntu` = l'utilisateur par défaut des AMI Ubuntu chez AWS ; `ansible_ssh_private_key_file` = la clé privée correspondant à la clé fournie à la création de l'EC2 (Bloc 6, Leçon 3).

> ⚠️ **Un prérequis du côté AWS** : le security group de la Leçon 5 doit autoriser SSH (port 22) **depuis ton IP** — c'était déjà le cas dans le code. Si tu utilises une **VM locale** (VirtualBox/Vagrant) à la place, même logique : IP locale + clé SSH partagée.

### 2.3 Les variables : paramétrer le playbook

Une **variable** Ansible se déclare dans le play (`vars:`) ou passe **en ligne de commande** avec `-e` (*extra vars* — la plus haute priorité, pratique pour les tests) :

```yaml
- name: Serveur web
  hosts: serv_web
  vars:
    nom_site: "Bibliotheque"
```

```bash
ansible-playbook site.yml -e "nom_site=Atelier"   # -e = écrase la valeur du play
```

**La priorité des variables** (à gros traits, suffisant pour ce bloc) : les `-e` (extra vars) **gagnent** sur les `vars` du play, qui gagnent sur les `defaults` du rôle (§ 2.6). Autrement dit : plus la valeur est « proche de l'exécution », plus elle gagne.

> 🟡 **À mentionner, définir, ne pas creuser — Ansible Vault** : l'outil intégré qui **chiffre** les fichiers de variables contenant des secrets (mots de passe, clés), protégés par un mot de passe maître. Pour ce bloc, on passe les secrets par `-e` en ligne de commande (comme les `TF_VAR_` de la Leçon 5) ; Vault est le standard professionnel à connaître d'un mot.

### 2.4 Les templates Jinja2 : des fichiers générés, pas copiés

Un **template** est un fichier modèle, en général avec l'extension **`.j2`** (pour Jinja2, le moteur de modèles que Python — rappel du Bloc 3 — utilise partout). Il contient du texte + des **espaces de substitution** :

```jinja
<h1>Bienvenue sur {{ nom_site }}</h1>       {{ ... }} = "insère la valeur de"
<p>Hébergé par la machine {{ ansible_hostname }}.</p>
```

`ansible_hostname` est un **fact** (Leçon 6 : l'info collectée sur la cible) : le même template, déployé sur cinq serveurs, produit **cinq pages différentes, personnalisées par machine** — sans cinq fichiers. Là où le Bloc 7 (PostgreSQL) te faisait éditer `postgresql.conf` à la main, le template fait ce travail : **une source, N cibles, N rendus**.

### 2.5 Les handlers : réagir, une seule fois

Un **handler** est une task spéciale qui ne s'exécute **que si une autre task l'a « notifié »** (`notify:`) **et** que cette task a effectivement changé quelque chose. De plus, si **plusieurs** tasks notifient le même handler, il ne s'exécute qu'**une fois**, à la fin du play.

```yaml
tasks:
  - name: Deposer la configuration
    template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    notify: Redemarrer nginx          # "préviens ce handler si tu as changé qqch"

handlers:
  - name: Redemarrer nginx            # ne tourne QUE si notifié et si changé
    service:
      name: nginx
      state: restarted
```

> 💡 **Analogie** : le handler est une **sonnette**. La task sonne **seulement si elle a modifié quelque chose** ; et si trois tasks sonnent le même bouton, la porte s'ouvre **une fois**, pas trois. Résultat : pas de redémarrage de nginx à chaque exécution du playbook (qui serait un `changed` à chaque passage — l'anti-idempotence).

### 2.6 Les rôles : la structure professionnelle

Un **rôle** est un dossier qui regroupe tout ce qu'il faut pour une brique de configuration, selon l'**arborescence standard** :

```
roles/
└── serveur_web/
    ├── defaults/        ← les valeurs PAR DÉFAUT (surchargeables)
    │   └── main.yml
    ├── tasks/           ← les tasks (le cœur)
    │   └── main.yml
    ├── handlers/        ← les handlers (redémarrages…)
    │   └── main.yml
    ├── templates/       ← les .j2
    │   └── index.html.j2
    └── meta/            ← les métadonnées (dépendances du rôle)
        └── main.yml
```

Le playbook devient minuscule — il **appelle** le rôle (comme le projet racine Terraform appelle ses modules, Leçon 4) :

```yaml
- name: Deployer le serveur web
  hosts: serv_web
  become: true
  roles:
    - role: serveur_web          # la logique vit dans le rôle
      vars:
        nom_site: "Bibliotheque" # éventuelles surcharges ici
```

> 🟡 **À mentionner, définir, ne pas creuser — Ansible Galaxy** : le catalogue public de rôles partagés par la communauté (l'équivalent du Registry Terraform, Leçon 4). `ansible-galaxy install nom-du-role` télécharge un rôle éprouvé. Pour apprendre, on écrit les nôtres.

---

## 📖 Vocabulaire / Abréviations

| Terme | Définition (une ligne) | Où |
|-------|------------------------|-----|
| **Variable** | Une valeur paramétrable (`vars`, `-e`, defaults du rôle) | § 2.3 |
| **Extra vars (`-e`)** | Variables passées en ligne de commande, priorité maximale | § 2.3 |
| **Ansible Vault** | L'outil qui chiffre les fichiers de variables à secrets (mention) | § 2.3 |
| **Template (`.j2`)** | Un modèle de fichier avec des espaces `{{ ... }}` remplis au déploiement | § 2.4 |
| **Jinja2** | Le moteur de modèles (syntaxe `{{ }}`) utilisé par Ansible | § 2.4 |
| **Fact** | Une info collectée sur la cible, utilisable comme variable | § 2.4 |
| **Handler** | Une task exécutée uniquement si notifiée par un changement | § 2.5 |
| **`notify`** | L'instruction qui « sonne » un handler | § 2.5 |
| **Rôle (role)** | Un dossier structurant une brique de configuration réutilisable | § 2.6 |
| **`defaults/`** | Les valeurs par défaut du rôle, les plus faciles à surcharger | § 2.6 |
| **Ansible Galaxy** | Le catalogue public de rôles (mention) | § 2.6 |
| **`site.yml`** | Le playbook d'entrée qui appelle les rôles | § 3.4 |

---

## 3. Exemples concrets

On construit maintenant le rôle complet. Toutes les commandes sont commentées ligne par ligne.

### 3.1 Le projet et l'inventaire

```bash
# Crée l'arborescence : -p crée tous les dossiers parents au besoin.
# {a,b,c} = "brace expansion" : crée les trois dossiers d'un coup (Bloc 2).
mkdir -p ~/ansible-pro/roles/serveur_web/{tasks,handlers,templates,defaults}
cd ~/ansible-pro

# L'inventaire : choisis ton option (A : localhost, ou B : EC2/SSH — § 2.2).
nano inventory.ini
```

Option A (localhost, sans risque) :

```ini
[serv_web]
localhost ansible_connection=local
```

Option B (l'EC2 de la Leçon 5) :

```ini
[serv_web]
atelier-app ansible_host=TON.IP.PUBLIQUE ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/atelier.pem
```

### 3.2 Les defaults du rôle

```bash
nano roles/serveur_web/defaults/main.yml
```

```yaml
---
# Les valeurs PAR DÉFAUT : les plus faciles à surcharger (priorité la plus basse).
nom_site: "Site par defaut"
port_nginx: 80
```

**Explication** : la convention du rôle — **tout ce qui est réglable** passe par `defaults`. Le rôle fonctionne sans que l'appelant ne fournisse quoi que ce soit (qui peut toujours surcharger). C'est l'équivalent des `default` des `variable` Terraform (Leçon 4).

### 3.3 Les tasks, les templates et le handler

```bash
nano roles/serveur_web/tasks/main.yml
```

```yaml
---
# Le cœur du rôle : les tasks, DANS L'ORDRE. Pas de "hosts:" ici :
# c'est le playbook appelant qui choisit les cibles (§ 2.6).

# Task 1 : le paquet installé (état voulu : présent).
- name: Installer nginx
  ansible.builtin.apt:
    name: nginx
    state: present
    update_cache: true

# Task 2 : la CONFIGURATION via template (pas de texte en dur).
- name: Deposer la configuration nginx
  ansible.builtin.template:                     # template, pas copy : le fichier est RENDU
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    mode: "0644"                                # permissions classiques d'un fichier de config
  notify: Redemarrer nginx                      # sonne si le fichier a changé

# Task 3 : la page d'accueil, générée par template.
- name: Deposer la page d'accueil
  ansible.builtin.template:
    src: index.html.j2
    dest: /var/www/html/index.html
    mode: "0644"
  notify: Redemarrer nginx
```

```bash
nano roles/serveur_web/handlers/main.yml
```

```yaml
---
# Le handler : exécuté UNE fois si (et seulement si) une task notifiée a changé qqch.
- name: Redemarrer nginx
  ansible.builtin.service:
    name: nginx
    state: restarted
```

```bash
nano roles/serveur_web/templates/index.html.j2
```

```html
<!DOCTYPE html>
<!-- Template Jinja2 : les {{ }} seront remplis au déploiement. -->
<html>
<head><title>{{ nom_site }}</title></head>
<body>
  <h1>Bienvenue sur {{ nom_site }}</h1>
  <p>Serveur : {{ ansible_hostname }} — port : {{ port_nginx }}.</p>
  <p>Configure par Ansible (Bloc 8, Lecon 7).</p>
</body>
</html>
```

```bash
nano roles/serveur_web/templates/nginx.conf.j2
```

```ini
# Template de configuration nginx : minimal mais réel.
# worker_processes : le nombre de "bras" du service (le fact rend le choix automatique).
worker_processes {{ ansible_processor_vcpus | default(1) }};

events { }

http {
    include       /etc/nginx/mime.types;
    access_log    /var/log/nginx/access.log;
    # Le port d'écoute vient de la variable du rôle (defaults/main.yml).
    server {
        listen {{ port_nginx }};
        location / {
            return 200 "Configuration generee pour {{ nom_site }}\n";
        }
    }
}
```

**Lecture des templates** : `{{ ansible_processor_vcpus | default(1) }}` = « le nombre de processeurs du fact ; **si absent, 1** » (le `| default(1)` est le garde-fou des templates — jamais de page blanche parce qu'un fact manque). Ce genre de `nginx.conf` paramétré est exactement ce que le Bloc 7 t'a fait éditer à la main — ici, il est **généré**.

### 3.4 Le playbook d'entrée — `site.yml`

```bash
nano site.yml
```

```yaml
---
# Le playbook d'entrée : MINUSCULE, car toute la logique vit dans le rôle.
# C'est ce fichier que l'équipe exécute — jamais les tasks directement.
- name: Deployer le serveur web
  hosts: serv_web                # la/les cibles (une ou dix, c'est pareil)
  become: true                   # installation de paquets + /etc
  roles:
    - role: serveur_web          # le rôle fait le travail
```

### 3.5 Exécuter, surcharger, vérifier

```bash
# Simulation d'abord (le filet de sécurité, Leçon 6) :
ansible-playbook site.yml -i inventory.ini --check

# Pour de vrai :
ansible-playbook site.yml -i inventory.ini
# → ok=3 changed=3 failed=0 (1er passage : install + config + page)

# Surcharge par extra var (priorité maximale, § 2.3) :
ansible-playbook site.yml -i inventory.ini -e "nom_site=Atelier-Teste"
# → seule la page change (et le handler redémarre nginx : la config cite nom_site)

# Le rendu réel (le port 80, HTTP — Bloc 5, Leçon 6) :
curl http://localhost
```

```bash
# Relance sans rien changer : l'idempotence, encore et toujours.
ansible-playbook site.yml -i inventory.ini
# → ok=3 changed=0 : le handler n'a PAS tourné (pas de sonnette).
```

**Test du handler** : modifie une ligne du template `index.html.j2`, relance :

```
RUNNING HANDLER [serveur_web : Redemarrer nginx]
PLAY RECAP : localhost : ok=4 changed=2 failed=0
```

Lecture : la page a changé (`changed`), le **handler s'est exécuté une fois** (la 4ᵉ ligne `ok` en plus = le handler lui-même). Si tu avais changé **les deux** templates, le handler n'aurait **toujours tourné qu'une fois** — c'est la promesse du § 2.5.

### 3.6 Le test de l'échelle : deux cibles, zéro ligne de logique en plus

```ini
# inventory.ini — deux cibles, le même groupe :
[serv_web]
web-1 ansible_connection=local
web-2 ansible_connection=local
```

```bash
ansible-playbook site.yml -i inventory.ini
# → les deux cibles sont configurées, avec leur hostname PERSONNALISÉ
#   dans la page (le fact {{ ansible_hostname }} — § 2.4).
```

> 🔑 **La preuve du concept** : une seule source (le rôle), N cibles, N rendus personnalisés. Étendre l'infra d'une machine, c'est **une ligne d'inventaire** — pas une copie de playbook.

---

## 4. Bonnes pratiques modernes (2025-2026)

- **Rôles pour tout ce qui est réutilisable** : dès qu'un playbook dépasse ~3 tasks ou servira deux fois, il devient un rôle (l'équivalent des modules Terraform, Leçon 4).
- **`defaults/` d'abord** : toute valeur réglable part dans `defaults/main.yml` — l'appelant surcharge, le rôle garde des valeurs sûres.
- **`template` plutôt que `copy`** dès qu'un fichier dépend d'une variable, d'un fact ou de la machine.
- **Handler pour les redémarrages** : jamais `state: restarted` en task directe (il tournerait à chaque passage — anti-idempotence).
- **`| default(...)` dans les templates** : garde-fou contre les facts/variables absents.
- **Les secrets via `-e` (ici) ou Ansible Vault (le standard)** : jamais dans `defaults/`, jamais dans Git.

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi c'est dangereux/inefficace | ✅ Version correcte |
|-----------------|--------------------------------------|---------------------|
| `state: restarted` en task directe | Redémarre le service **à chaque exécution** : coupure inutile, `changed` permanent | Task idempotente + **handler** `restarted` |
| Coder le nom du site / du port en dur | Un changement = éditer N playbooks | `defaults/` + variables (`vars`, `-e`) |
| `copy` avec un contenu dépendant de la machine | Le fichier est identique partout : pas de personnalisation | `template` `.j2` avec `{{ }}` |
| Secrets dans `defaults/main.yml` (versionné) | Publié dans Git pour toujours (Bloc 5 — secrets) | `-e` ponctuel, Vault en standard |
| Handler laissé dans le playbook quand la logique est dans le rôle | Confus, fragile | Le handler vit dans le **rôle** qui notifie |
| Oublier `--check` sur une cible réelle | Changements découverts après coup | `--check` d'abord, toujours sur l'inconnu |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`**, la correction dans **`03-correction.md`**.

**Énoncé court** : crée le rôle `serveur_web` (defaults, tasks, handlers, templates `index.html.j2` et `nginx.conf.j2`), le playbook `site.yml`, teste les variables (`-e` vs defaults), le handler (relance sans changement → rien ; changement du template → un redémarrage), et la montée en échelle (2 cibles, zéro ligne de logique en plus).

---

## 7. Correction détaillée de l'exercice

> La correction complète est dans **`03-correction.md`** : arborescence finale, sorties attendues (y compris le moment exact où le handler tourne), et la priorité des variables observée.

---

## 8. Checklist de validation

- [ ] Je cible une vraie machine par SSH (inventaire complet : host, user, clé).
- [ ] J'explique la priorité des variables (extra vars > vars du play > defaults du rôle).
- [ ] J'écris un template `.j2` avec variables, facts et garde-fou `| default(...)`.
- [ ] J'explique les handlers (`notify` : sonner une fois, seulement si changement).
- [ ] Je structure un rôle (tasks/handlers/templates/defaults) et je l'appelle depuis `site.yml`.
- [ ] J'ai vérifié l'idempotence **avec handler** : relance sans changement → zéro redémarrage.

---

🧭 **Pont vers la suite** — Tu sais maintenant créer l'infrastructure **en code** (Terraform, Leçons 2-5) et la configurer **en code** (Ansible, Leçons 6-7). Une question reste ouverte, et c'est le cœur de la roadmap pour ce bloc : **pourquoi** tout ce travail ? Pour concevoir une infrastructure qui **supporte la charge, survit aux pannes et se restaure** : scalabilité, haute disponibilité, RTO/RPO, disaster recovery. C'est la **Leçon 8** — la partie « Architecture système » de la roadmap.

---

*Prochaine étape :* Leçon 8 — **Scalabilité, haute disponibilité et disaster recovery** dans `08-Haute-disponibilite-et-DR/`.