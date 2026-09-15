# Correction — Leçon 6 : Ansible, bases, inventaire et premier playbook

> **Bloc 8 · Leçon 6** — Correction pas à pas.

---

## Étape 1 — Installer Ansible

```bash
python3 -m pip install --user --upgrade ansible
ansible --version
# ansible [core 2.17.x]
#   config file = None
#   ...
```

**Explication** : `pip --user` installe dans `~/.local/lib/...` et place les exécutables dans `~/.local/bin`. Si `ansible` n'est pas trouvé : `export PATH="$HOME/.local/bin:$PATH"` dans `~/.bashrc` (même remediation que l'AWS CLI, Bloc 6 Leçon 1).

**Pourquoi pip et pas `apt` ?** Le paquet `ansible` d'Ubuntu est un « paquet tombstone » (retiré volontairement, il affiche un message qui explique d'utiliser pip). `pip --user` garantit la version actuelle, pour ton compte, sans `sudo`.

## Étape 2 — L'inventaire

```ini
[local]
localhost ansible_connection=local
```

**Explication** :
- `[local]` : un **groupe** — on peut cibler un groupe (`ansible local …`) ou toutes les cibles (`ansible all …`).
- `localhost` : l'alias de la cible.
- `ansible_connection=local` : « exécute directement ici » — sans cette ligne, Ansible tenterait `ssh localhost` et te demanderait sans doute un mot de passe (Bloc 2, Leçon 5 : on utilise des clés, pas des mots de passe — mais ici, inutile de SSH du tout).

**Réponse à la question de l'étape 3** : `ansible_connection=local` signifie « pas de SSH, exécute les tasks directement sur la machine de contrôle ».

## Étape 3 — Les ad-hoc

```bash
ansible local -m ping -i inventory.ini
# localhost | SUCCESS => {
#     "changed": false,
#     "ping": "pong"
# }
```

**Explication** : le module `ping` d'Ansible n'est pas le `ping` réseau (Bloc 5, Leçon 2) : il teste le **chemin complet d'exécution** (contact + interpréteur Python + capacité à exécuter un module). Le `changed: false` est logique : un diagnostic ne change rien — idempotence dès le diagnostic.

```bash
ansible local -m setup -i inventory.ini | head -30
```

**Deux facts attendues** (parmi tant d'autres) : `ansible_distribution` (ex. `Ubuntu`), `ansible_memtotal_mb` (la RAM). Toute la machine est « rapportée » : ces informations deviennent des **variables** utilisables dans les templates (Leçon 7).

```bash
ansible local -a "uname -a" -i inventory.ini
```

**Explication** : sans `-m`, l'option `-a` s'applique au module par défaut `command` — une commande libre, **non idempotente** : c'est pour ça qu'on la réserve au diagnostic.

## Étape 4 — Le premier playbook

Les trois tasks, une à une :

| Task | Module | État voulu | Pourquoi idempotent |
|------|--------|------------|---------------------|
| Installer nginx | `ansible.builtin.apt` | `state: present` | « présent » : installe **s'il manque**, ne fait rien sinon |
| Démarrer et activer | `ansible.builtin.service` | `started` + `enabled: true` | démarre **s'il est arrêté** ; active **s'il n'est pas activé** |
| Page d'accueil | `ansible.builtin.copy` | `content` + `dest` | compare le contenu, réécrit **seulement si différent** |

**Réponses aux questions attendues** :
- `update_cache: true` = l'équivalent du `apt update` avant installation (raffraîchit la liste des paquets) — fait **une fois** par exécution, pas à chaque task.
- `become: true` = exécuter avec sudo, nécessaire car `/var/www/html` et l'installation de paquets demandent des droits root.

```bash
ansible-playbook 01-nginx.yml -i inventory.ini --check
# PLAY RECAP : localhost : ok=3  changed=3  failed=0   ← mais rien n'a été fait !
ansible-playbook 01-nginx.yml -i inventory.ini
# PLAY RECAP : localhost : ok=3  changed=3  failed=0
```

**Le `--check`** a montré le programme de travaux sans toucher — le filet de sécurité, équivalent exact du `terraform plan` (Leçon 2). Ce parallèle est l'un des fils du bloc : **prévisualiser, puis exécuter**.

## Étape 5 — L'idempotence vérifiée

```bash
ansible-playbook 01-nginx.yml -i inventory.ini
# PLAY RECAP : localhost : ok=3  changed=0  failed=0
```

**Explication attendue** : `changed=0` au second passage. Les modules dédiés (`apt`, `service`, `copy`) **vérifient l'état actuel avant d'agir** : rien à changer → rien ne change. C'est la définition de l'idempotence de la Leçon 1 (« assure-toi que… »), rendue concrète.

```bash
curl http://localhost
# Serveur prepare par Ansible - lecon 6
```

## Étape 6 — La mise à jour ciblée

Après modification du `content` de la Task 3 et relance :

```
PLAY RECAP : localhost : ok=3  changed=1  failed=0
```

**Explication** : seul le `copy` a détecté un écart (contenu différent) → `changed=1`. Les deux autres tasks confirment leur état (`ok`, `changed=0`). C'est la mise à jour ciblée de la Leçon 2, version Ansible : **le changement exact, rien de plus**.

## Étape 7 — Nettoyage (optionnel)

```bash
sudo apt remove -y nginx && sudo apt autoremove -y
```

**Explication** : `remove` désinstalle nginx ; `autoremove` nettoie les dépendances devenues inutiles. (Si tu comptes enchaîner directement sur la Leçon 7, tu peux garder nginx : le playbook avancé le réutilisera.)

---

## Checklist de validation (leçon 6)

- [ ] J'explique l'architecture « sans agent » et le rôle de SSH (rappel Bloc 2, Leçon 5).
- [ ] Je crée un inventaire (groupes, `ansible_host`, `ansible_connection=local`) et je sais à quoi il sert.
- [ ] Je fais la différence `ansible` (ad-hoc) vs `ansible-playbook`, et je sais quand utiliser l'un ou l'autre.
- [ ] J'écris un playbook YAML avec des modules dédiés (`apt`, `service`, `copy`) et des états voulus.
- [ ] J'ai **vérifié l'idempotence** (second passage : `changed=0`) et je sais pourquoi les modules dédiés la garantissent.
- [ ] Je connais le duo de sécurité : `--check` (simulation) avant d'exécuter sur une cible inconnue.

---

## 🧠 Conseils pour la suite

- **Grave le réflexe** : `changed` = « j'ai agi », `ok` avec `changed=0` = « l'état était déjà bon ». Si un playbook qui **devait** être idempotent affiche des `changed` à chaque passage, cherche le `shell`/`command` fautif.
- **Garde nginx** si tu enchaînes sur la Leçon 7 : le rôle « serveur web » la réutilisera avec variables, template et handler — tu verras la différence entre un playbook linéaire et un rôle structuré.
- **Le lien Leçon 5 → Leçon 7** : si tu as créé l'EC2 de la Leçon 5, garde l'IP publique et la clé SSH à portée : l'inventaire de la Leçon 7 ciblera cette machine réelle par SSH.
