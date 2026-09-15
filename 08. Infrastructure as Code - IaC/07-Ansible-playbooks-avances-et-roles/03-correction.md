# Correction — Leçon 7 : Ansible avancé — variables, templates, handlers et rôles

> **Bloc 8 · Leçon 7** — Correction pas à pas.

---

## Étape 1 — L'inventaire et le contact

```bash
ansible serv_web -m ping -i inventory.ini
# localhost | SUCCESS => { "changed": false, "ping": "pong" }
# atelier-app | SUCCESS => { ... }   (option B : la vraie EC2 répond via SSH)
```

**Explication (option B)** : `ansible_host` = l'IP publique de l'output de la Leçon 5 ; `ansible_user=ubuntu` = l'utilisateur des AMI Ubuntu AWS ; `ansible_ssh_private_key_file` = la clé privée correspondante. Si le contact échoue (option B), 3 causes classiques : le security group n'ouvre pas le port 22 à ton IP (Leçon 5) ; la clé n'a pas les permissions `600` (Bloc 2, Leçon 5) ; mauvais utilisateur (`ubuntu` pour Ubuntu, pas `root`).

## Étape 2 — L'arborescence du rôle

```
~/ansible-pro/
├── inventory.ini
├── site.yml
└── roles/serveur_web/
    ├── defaults/main.yml     # nom_site, port_nginx (défauts surchargeables)
    ├── tasks/main.yml        # install + template config + template page
    ├── handlers/main.yml     # Redemarrer nginx
    └── templates/
        ├── index.html.j2
        └── nginx.conf.j2
```

**Explication** : chaque dossier du rôle a un rôle précis (au sens propre) — Ansible sait **tout seul** où chercher les templates des tasks (`templates/`), les handlers (`handlers/main.yml`), etc. C'est cette convention qui rend le rôle « branchable » partout.

## Étape 3 — Les fichiers du rôle

**`defaults/main.yml`** :

```yaml
---
nom_site: "Site par defaut"   # surchargeable, priorité la plus basse
port_nginx: 80
```

**`tasks/main.yml`** — les 3 tasks (Leçon 6 + template + notify) :

```yaml
---
- name: Installer nginx
  ansible.builtin.apt:
    name: nginx
    state: present
    update_cache: true

- name: Deposer la configuration nginx
  ansible.builtin.template:        # RENDU du .j2, pas simple copie
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    mode: "0644"
  notify: Redemarrer nginx         # sonne SI (et seulement si) le fichier a changé

- name: Deposer la page d'accueil
  ansible.builtin.template:
    src: index.html.j2
    dest: /var/www/html/index.html
    mode: "0644"
  notify: Redemarrer nginx
```

**`handlers/main.yml`** :

```yaml
---
- name: Redemarrer nginx
  ansible.builtin.service:
    name: nginx
    state: restarted     # ici, "restarted" est CORRECT : c'est un handler,
                         # exécuté une seule fois, uniquement après un vrai changement
```

**`templates/index.html.j2`** :

```html
<html>
<head><title>{{ nom_site }}</title></head>
<body>
  <h1>Bienvenue sur {{ nom_site }}</h1>
  <p>Serveur : {{ ansible_hostname }} — port : {{ port_nginx }}.</p>
</body>
</html>
```

**`templates/nginx.conf.j2`** :

```ini
worker_processes {{ ansible_processor_vcpus | default(1) }};
events { }
http {
    include       /etc/nginx/mime.types;
    access_log    /var/log/nginx/access.log;
    server {
        listen {{ port_nginx }};
        location / {
            return 200 "Configuration generee pour {{ nom_site }}\n";
        }
    }
}
```

## Étape 3 — Le playbook `site.yml`

```yaml
---
- name: Deployer le serveur web
  hosts: serv_web
  become: true
  roles:
    - role: serveur_web
```

**Explication** : le playbook ne contient **aucune logique** — il dit « qui » (`hosts`) et « avec quoi » (`roles`). Corriger le comportement = éditer le **rôle** (une fois) ; changer le réglage = surcharger la **variable** (un argument). C'est la séparation du rôle et de l'appel, identique à celle des modules Terraform (Leçon 4).

## Étape 4 — La priorité des variables, observée

```bash
ansible-playbook site.yml -i inventory.ini -e "nom_site=Atelier-Teste"
curl http://localhost
# → "Atelier-Teste" : l'extra var a GAGNÉ sur les defaults.
```

Relance **sans** `-e` :

```bash
ansible-playbook site.yml -i inventory.ini
curl http://localhost
# → "Site par defaut" : retombe sur les defaults du rôle.
```

**Réponse attendue** : l'extra var (`-e`) gagne toujours (elle est la plus « proche de l'exécution ») ; sans elle, le `defaults/` du rôle prend le relais. Entre les deux : les `vars` du play. C'est la règle « extra vars > vars du play > defaults », observée par toi-même.

## Étape 5 — Le handler, observé

**Cas 1 — relance sans changement** :

```
PLAY RECAP : localhost : ok=3 changed=0 failed=0
```

→ Le handler **n'a pas tourné** : aucune task notifiante n'a changé quelque chose. Pas de redémarrage = coupure évitée.

**Cas 2 — template modifié puis relance** :

```
RUNNING HANDLER [serveur_web : Redemarrer nginx]
PLAY RECAP : localhost : ok=4 changed=2 failed=0
```

→ Le handler tourne **une fois**, après les tasks. Et si les **deux** templates changent dans la même exécution, il ne tourne **toujours qu'une fois** : les deux `notify` pointent le même handler, qui est exécuté une fois par play. C'est le comportement « sonnette » du § 2.5.

**Pourquoi `restarted` est correct ICI** : en task directe, `restarted` redémarrerait à chaque passage (anti-idempotence, piège du § 5). Dans un **handler**, il ne s'exécute qu'après un vrai changement — c'est exactement son rôle.

## Étape 6 — Le test de l'échelle

```ini
[serv_web]
web-1 ansible_connection=local
web-2 ansible_connection=local
```

```bash
ansible-playbook site.yml -i inventory.ini
# → chaque cible : ok=3 changed=3, la page de chacune cite SON hostname.
```

**Réponse attendue (étape 6)** : le coût en lignes de code pour doubler l'infrastructure = **une ligne d'inventaire**. Zéro nouvelle task, zéro template copié. C'est la traduction Ansible du « count » de Terraform — et le pont direct avec la Leçon 8 : quand la charge monte, on ajoute des cibles, pas du code.

---

## Checklist de validation (leçon 7)

- [ ] Je cible une vraie machine par SSH (inventaire complet : host, user, clé).
- [ ] J'explique la priorité des variables (extra vars > vars du play > defaults du rôle).
- [ ] J'écris un template `.j2` avec variables, facts et garde-fou `| default(...)`.
- [ ] J'explique les handlers (`notify` : sonner une fois, seulement si changement).
- [ ] Je structure un rôle (tasks/handlers/templates/defaults) et je l'appelle depuis `site.yml`.
- [ ] J'ai vérifié l'idempotence **avec handler** : relance sans changement → zéro redémarrage.

---

## 🧠 Conseils pour la suite

- **Le réflexe « rôle »** : au projet final (Leçon 9), tu appelleras ce rôle pour configurer les machines créées par Terraform — le flux exact de la roadmap (Terraform → Ansible → application).
- **La frontière Terraform/Ansible** se voit clairement ici : le rôle configure l'intérieur ; si tu changes le **nombre** de machines, c'est Terraform (infra) qui le fait — Ansible suit via l'inventaire.
- **Leçon 8** : plus de code, de la **conception** — le contenu « Architecture système » de la roadmap (scalabilité, HA, DR, RTO/RPO). Tu vas relier tout ce que le bloc t'a fait construire à la question « pourquoi ».
