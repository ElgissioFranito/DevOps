# Exercice — Leçon 6 : Ansible, bases, inventaire et premier playbook

> **Bloc 8 · Leçon 6** — Exercice en autonomie, **100 % local** : les cibles se limitent à `localhost` (ta machine, via `ansible_connection=local`). La seule installation sur ta machine est **nginx** (réversible : une commande de désinstallation est fournie à la fin).

---

## Contexte

Ansible s'apprend en **pratiquant** : un inventaire, des commandes ad-hoc pour prendre ses marques, puis un playbook idempotent — le même genre de configuration que celle que tu as faite à la main aux Blocs 2 et 7.

---

## Énoncé

> 📌 **Rappels d'options** : `-m MODULE` = choisir le module ad-hoc ; `-a "ARGUMENTS"` = les arguments du module ; `-i FICHIER` = quel inventaire utiliser ; `--check` = **simulation** (montre ce qui serait fait, ne fait rien) ; `become: true` = exécuter avec sudo.

### Étape 1 — Installer Ansible

```bash
# Installe via pip (l'installation recommandée en 2025 ; --user = pour ton compte).
python3 -m pip install --user --upgrade ansible

# Vérifie : version + config.
ansible --version
```

> 💡 Si la commande `ansible` n'est pas trouvée : ajoute `~/.local/bin` dans le `PATH` (comme pour l'AWS CLI au Bloc 6, Leçon 1).

### Étape 2 — Créer le dossier de projet et l'inventaire

```bash
mkdir -p ~/ansible-debut && cd ~/ansible-debut

# Crée l'inventaire.
nano inventory.ini
```

Contenu :

```ini
[local]
localhost ansible_connection=local   # la cible spéciale : ta machine, sans SSH
```

### Étape 3 — Premiers pas ad-hoc

```bash
# 1. Test de contact : le module "ping" d'Ansible (pas le ping réseau du Bloc 5 !).
ansible local -m ping -i inventory.ini

# 2. Le rapport d'inspection : le module "setup" collecte les facts.
ansible local -m setup -i inventory.ini | head -30

# 3. Une commande libre : ces infos collectées suffisent déjà à fabriquer un rapport.
ansible local -a "uname -a" -i inventory.ini

# 4. Version d'Ansible depuis Ansible lui-même (utile pour un rapport d'inventaire).
ansible local -a "ansible --version" -i inventory.ini | head -5
```

Note dans `notes-exercice-06.md` : la sortie du ping (le mot-clé `SUCCESS`/`pong`), **2 facts** intéressantes repérées dans `setup` (ex. distribution, mémoire), et ce que fait `ansible_connection=local` (une phrase).

### Étape 4 — Ton premier playbook

```bash
nano 01-nginx.yml
```

Contenu (l'explication complète est dans la leçon § 3.3) :

```yaml
---
# La cible et les réglages du play.
- name: Preparer le serveur web (localhost)
  hosts: local
  become: true          # sudo nécessaire pour installer des paquets

  tasks:
    # Task 1 : le paquet nginx installé (l'état voulu, pas le geste).
    - name: Installer nginx
      ansible.builtin.apt:
        name: nginx
        state: present    # "présent" : installe s'il manque, RIEN sinon
        update_cache: true

    # Task 2 : le service démarré ET activé au démarrage.
    - name: Demarrer et activer nginx
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true

    # Task 3 : la page d'accueil personnalisée.
    - name: Deposer la page d'accueil
      ansible.builtin.copy:
        content: "Serveur prepare par Ansible - lecon 6\n"
        dest: /var/www/html/index.html
```

Exécute-le **en simulation d'abord** :

```bash
# --check : montre ce qui serait fait, sans rien faire (le "plan" d'Ansible !).
ansible-playbook 01-nginx.yml -i inventory.ini --check

# Puis pour de vrai.
ansible-playbook 01-nginx.yml -i inventory.ini
```

### Étape 5 — Vérifier l'idempotence (le cœur de la leçon)

```bash
# Relance le playbook SANS rien changer.
ansible-playbook 01-nginx.yml -i inventory.ini

# Vérifie le résultat réel.
curl http://localhost
```

Note : combien de `changed=1` vs `changed=0` au second passage ? Comment s'appelle ce comportement (Leçon 1 !) ?

### Étape 6 — Modifier la page, observer la mise à jour ciblée

```bash
# Change le contenu dans le playbook (la Task 3), puis relance.
ansible-playbook 01-nginx.yml -i inventory.ini
# → seul nginx/config est mis à jour : observe "changed=1" SUR LA Task 3 uniquement.
```

### Étape 7 — Nettoyage (optionnel)

```bash
# Si tu veux retirer nginx de ta machine :
sudo apt remove -y nginx && sudo apt autoremove -y
```

---

## Livrable

- Le dossier `~/ansible-debut/` avec `inventory.ini` et `01-nginx.yml`.
- `notes-exercice-06.md` : le pong, 2 facts, l'explication de `ansible_connection=local`, les chiffres d'idempotence du second passage, et la Task qui a changé à l'étape 6.

Correction détaillée dans **`03-correction.md`**.