# Référence rapide — Leçon 6 : Ansible, bases

> Bloc 8 · Leçon 6 — Aide-mémoire.

## Installation

```bash
python3 -m pip install --user --upgrade ansible
ansible --version
# si "command not found" : export PATH="$HOME/.local/bin:$PATH"
```

## Architecture (sans agent)

```
Machine de contrôle (ton poste)  ── SSH ──▶  Cibles
   - inventaire (qui ?)
   - playbooks (quoi ?)
   - modules (comment ?)
```

## Inventaire

```ini
[serv_web]
atelier-app ansible_host=203.0.113.10 ansible_user=admin

[local]
localhost ansible_connection=local   # exécuter ici, sans SSH
```

## Ad-hoc

```bash
ansible GROUPE -m ping -i inventory.ini      # test de contact
ansible GROUPE -m setup -i inventory.ini     # les facts
ansible GROUPE -a "uptime" -i inventory.ini  # commande libre (diagnostic)
```

## Playbook

```yaml
---
- name: Titre du play
  hosts: groupe
  become: true                     # sudo
  tasks:
    - name: Titre de la task
      ansible.builtin.apt:
        name: nginx
        state: present             # L'ÉTAT VOULU (idempotent)
        update_cache: true
    - name: Service
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true
    - name: Fichier
      ansible.builtin.copy:
        content: "texte\n"
        dest: /chemin/fichier
```

## Commandes

```bash
ansible-playbook playbook.yml -i inventory.ini --check   # simulation (le "plan")
ansible-playbook playbook.yml -i inventory.ini           # exécution
```

## Lecture du résumé

```
localhost : ok=3  changed=3  failed=0     # 1er passage : 3 changements
localhost : ok=3  changed=0  failed=0     # 2e passage : idempotence ✔
```

## Règles d'or

- Module dédié > `command`/`shell` (les modules vérifient l'état ; `shell` non).
- Ad-hoc = diagnostic ; répétition = playbook.
- `--check` avant d'agir sur l'inconnu.
- Inventaire dans Git ; secrets jamais.