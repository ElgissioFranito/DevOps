# Référence rapide — Leçon 7 : Ansible avancé

> Bloc 8 · Leçon 7 — Aide-mémoire.

## Inventaire d'une vraie machine (SSH)

```ini
[serv_web]
atelier-app ansible_host=IP.PUBLIQUE ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/atelier.pem
```

## Priorité des variables (à gros traits)

```
extra vars (-e)   >   vars du play   >   defaults du rôle
(les plus proches de l'exécution gagnent)
```

## Template Jinja2 (.j2)

```jinja
<h1>{{ nom_site }}</h1>                          # variable du rôle/play
<p>{{ ansible_hostname }}</p>                    # fact de la cible
worker_processes {{ ansible_processor_vcpus | default(1) }};  # garde-fou
```

Module `template` (rendu) ≠ module `copy` (copie brute).

## Handlers

```yaml
tasks:
  - name: Deposer la config
    ansible.builtin.template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    notify: Redemarrer nginx      # sonne SI changé

handlers:
  - name: Redemarrer nginx
    ansible.builtin.service:
      name: nginx
      state: restarted            # une seule fois, à la fin du play
```

- Pas de handler si rien n'a changé → idempotence préservée.
- `state: restarted` en task directe = redémarrage à chaque passage (jamais).

## Rôle (arborescence standard)

```
roles/serveur_web/
├── defaults/main.yml   # valeurs par défaut (surcharges faciles)
├── tasks/main.yml      # le cœur
├── handlers/main.yml   # les réactions
└── templates/*.j2      # les modèles
```

```yaml
# site.yml — le playbook d'entrée
- hosts: serv_web
  become: true
  roles:
    - role: serveur_web
```

## Commandes

```bash
ansible-playbook site.yml -i inventory.ini --check            # simulation
ansible-playbook site.yml -i inventory.ini                    # exécution
ansible-playbook site.yml -i inventory.ini -e "nom_site=X"    # surcharge
ansible-galaxy install NOM-DU-ROLE                            # catalogue Galaxy (mention)
```

## Règles d'or

- defaults pour tout ce qui est réglable ; secrets via `-e` ou Vault.
- handler pour les redémarrages ; `| default(...)` dans les templates.
- une seule source, N cibles : étendre = une ligne d'inventaire.