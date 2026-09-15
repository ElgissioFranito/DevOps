# Exercice — Leçon 7 : Ansible avancé — variables, templates, handlers et rôles

> **Bloc 8 · Leçon 7** — Exercice en autonomie. **Deux options de cible** : `localhost` (comme en Leçon 6, sans risque) ou une **vraie machine** (l'EC2 de la Leçon 5, ou une VM locale) par SSH. Les deux fonctionnent avec le même code.

---

## Contexte

Tu transformes le playbook linéaire de la Leçon 6 en **rôle professionnel** : variables, template Jinja2, handler — la structure que tu utiliseras dans le projet final (Leçon 9) pour configurer plusieurs serveurs.

---

## Énoncé

> 📌 **Rappels d'options** : `-e "var=valeur"` = passer une variable en ligne de commande (priorité maximale) ; `-i FICHIER` = l'inventaire ; `--limit CIBLE` = restreindre l'exécution à une seule cible.

### Étape 1 — L'inventaire (choisis ton option)

**Option A — localhost** :

```ini
[serv_web]
localhost ansible_connection=local
```

**Option B — EC2 de la Leçon 5** (recrée-la : `terraform apply` si tu l'as détruite, et récupère `terraform output ip_publique`) :

```ini
[serv_web]
atelier-app ansible_host=TON.IP.PUBLIQUE ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/atelier.pem
```

Test de contact dans les deux cas :

```bash
ansible serv_web -m ping -i inventory.ini
```

### Étape 2 — Construire le rôle

```bash
mkdir -p ~/ansible-pro/roles/serveur_web/{tasks,handlers,templates,defaults}
cd ~/ansible-pro
nano inventory.ini    # le contenu de l'étape 1
```

Crée les 4 fichiers du rôle (contenus dans la leçon, § 3) :

1. `roles/serveur_web/defaults/main.yml` : `nom_site` (défaut `Site par defaut`), `port_nginx` (défaut `80`).
2. `roles/serveur_web/tasks/main.yml` : les 3 tasks de la Leçon 6 (install, config, page), **la config et la page utilisent des templates avec `notify`**.
3. `roles/serveur_web/handlers/main.yml` : le handler `Redemarrer nginx`.
4. `roles/serveur_web/templates/index.html.j2` : la page qui injecte `nom_site`, `port_nginx` et le fact `ansible_hostname`.

### Étape 3 — Le playbook qui appelle le rôle

```bash
nano site.yml
```

```yaml
- name: Deployer le serveur web
  hosts: serv_web
  become: true
  roles:
    - role: serveur_web
```

Exécute :

```bash
ansible-playbook site.yml -i inventory.ini --check    # simulation d'abord
ansible-playbook site.yml -i inventory.ini            # puis pour de vrai
```

### Étape 4 — Tester les variables

```bash
# Surcharge par extra var (priorité maximale) :
ansible-playbook site.yml -i inventory.ini -e "nom_site=Atelier-Teste"

# Vérifie le rendu réel :
curl http://localhost
```

Puis **relance sans `-e`** : la page revient à la valeur des `defaults`. Note le comportement dans tes notes : **quel niveau de variable a gagné, à chaque fois ?**

### Étape 5 — Tester les handlers

```bash
# 1. Relance sans rien changer : le handler NE DOIT PAS tourner.
ansible-playbook site.yml -i inventory.ini

# 2. Modifie le template (ajoute une ligne, ex. un footer), relance :
ansible-playbook site.yml -i inventory.ini
# → observe : la task "Deposer la configuration" change → le handler tourne UNE fois.
```

Note : combien de `changed` à chaque cas ? Le handler a-t-il tourné ?

### Étape 6 — Le test de l'échelle

Preuve que le rôle réutilisable fait son travail : ajoute une **deuxième cible** dans l'inventaire (un deuxième alias pointant vers le même localhost, avec un nom différent ex. `web-2` + `ansible_connection=local`), relance le playbook, et vérifie que les **deux cibles** ont reçu la même configuration. Une phrase : **qu'est-ce que cela a coûté en lignes de code ?**

---

## Livrable

- Le dossier `~/ansible-pro/` : inventaire + rôle complet + `site.yml`.
- `notes-exercice-07.md` : le pong, la priorité des variables observée (étape 4), les comportements des handlers (étape 5), et la réponse à l'étape 6.

Correction détaillée dans **`03-correction.md`**.