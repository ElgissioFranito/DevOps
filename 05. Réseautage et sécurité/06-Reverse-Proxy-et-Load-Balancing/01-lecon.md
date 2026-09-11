# Leçon 6 — Reverse Proxy et Load Balancing

> **Bloc 5 · Réseautage et sécurité** — Leçon 6 sur 8
> 🧭 **Pont depuis les Leçons 1-5** : tu sais faire circuler les données (protocoles), adresser (IP), filtrer (pare-feu), chiffrer (TLS) et **joindre des réseaux distants en privé via un tunnel (VPN, Leçon 5)**. Mais dans une app web réelle (frontend **Angular**, backend **Spring Boot**, base **PostgreSQL**), on n'expose **jamais** chaque port au public. On place **un seul point d'entrée** devant : le **reverse proxy**, capable aussi de **répartir la charge** (load balancing). C'est ce qu'on construit ici avec **Nginx**.
> 👉 C'est l'architecture cible de la roadmap : `Internet → Nginx → Angular (front) / Spring Boot (backend) → PostgreSQL`.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Expliquer** ce qu'est un reverse proxy, pourquoi on le place devant une application, et le différencier du simple serveur web.
2. **Comprendre** ce qu'est un load balancer, et les notions de **health check**, **failover**, **session**, **L4 / L7**.
3. **Citer** les outils du marché (Nginx, Traefik, HAProxy, Apache mod_proxy) et savoir lequel choisir selon le besoin.
4. **Écrire** une configuration Nginx simple : servir un frontend statique et faire un reverse proxy vers un backend `/api`.
5. **Écrire** un bloc `upstream` de base pour répartir le trafic entre deux serveurs et vérifier le résultat avec `curl`.

---

## 2. Explication simple

### 2.1 Le « pourquoi » : un seul point d'entrée

Sans reverse proxy, tu devrais exposer au public le port de chaque service : le front (ex. 4200), le backend (ex. 8080), etc. C'est fragile (plus de ports ouverts = plus d'attaques) et illisible pour l'utilisateur (plusieurs adresses).

> 💡 **Analogie** : le reverse proxy est la **réception d'un immeuble** ou le **standard d'un centre d'appels**. Le visiteur ne téléphone pas directement à chaque bureau : il passe par l'accueil, qui sait « à qui » et « vers quel bureau » orienter l'appel. Le visiteur voit **une seule porte** (l'accueil).

De plus, le proxy **cache la topologie interne** : l'extérieur ne connaît que l'entrée, pas les serveurs applicatifs derrière (meilleure sécurité, le TLS étant aussi géré à l'accueil, comme vu à la Leçon 4).

### 2.2 Serveur web vs reverse proxy (le « quoi »)

- Un **serveur web** (Nginx en mode « static », Apache) **sert des fichiers** (le frontend compilé : HTML, CSS, JS).
- Un **reverse proxy** **retransmet la requête** à un autre serveur (le backend Spring Boot) et renvoie sa réponse au client.

Dans une stack Angular + Spring Boot, **Nginx fait souvent les deux** : il sert le frontend **et** renvoie les appels `/api/*` vers le backend.

```
Internet
   │
   ▼
 Nginx  ──── sert ─────────────>  Angular (fichiers statiques compilés)
   │
   └──── reverse proxy /api ──>  Spring Boot (backend)
                                    │
                                    ▼
                                PostgreSQL (base, jamais exposée)
```

> ℹ️ **Rappel Bloc 1/2** : « Angular est un frontend statique » = ses fichiers compilés sont de simples fichiers servis par Nginx (rien n'est « exécuté » côté serveur), contrairement à Node/Next.js.

### 2.3 Le load balancer (le « comment » et le « quand »)

Quand le trafic augmente, une seule instance backend peut saturer. On en déploie plusieurs et on place un **répartiteur de charge** devant.

```
                ┌── Backend 1 (Spring Boot)
Internet ── LB ── Backend 2
                └── Backend 3
```

Concepts clés :
- **Distribution du trafic** : le LB répartit les requêtes (par exemple en « round-robin », un tour chacun).
- **Health check** : le LB vérifie en permanence que chaque backend répond (`GET /health`). S'il ne répond plus, il est écarté.
- **Failover** : bascule automatique vers un backend sain en cas de panne d'un autre.
- **Session** : si l'app garde « à qui » le client a parlé (session/état), il faut coller le client au **même** backend (« sticky session »), sinon l'état se perd.
- **L4 vs L7** : L4 équilibre au niveau transport/TCP (IP/port), L7 au niveau applicatif/HTTP (ex. router `/api` distinct de `/`).

> 🔵 **À mentionner, définir, ne pas creuser** :
> - **Traefik** : reverse proxy moderne très utilisé avec Docker/Kubernetes (auto-découverte, certificats auto).
> - **HAProxy** : répartiteur ultra-performant, souvent utilisé en simple LB.
> - **Apache (mod_proxy)** : peut faire reverse proxy mais plus lourd ; surtout pour du legacy PHP.
> - **Tomcat** : serveur d'application Java ; Spring Boot embarque un Tomcat **interne** — sans lien direct avec un reverse proxy séparé.

---

## 📖 Mini-glossaire (à consulter avant les exemples)

> Définitions d'une ligne pour ne jamais être perdu(e).

- **Reverse proxy** : serveur intermédiaire placé **devant** des applications ; il reçoit la requête, la transmet au bon service, renvoie la réponse au client.
- **Load balancer (LB)** : répartiteur de charge qui distribue le trafic entre plusieurs serveurs.
- **Health check** : vérification automatique qu'un serveur/backend répond (ex. `GET /health`).
- **Failover** : basculement automatique vers une instance saine en cas de panne.
- **Sticky session** : fait de « coller » un client au même backend pour conserver son état.
- **Round-robin** : méthode de distribution où chaque backend reçoit une requête à tour de rôle.
- **L4 / L7** : niveaux de la pile réseau. L4 = transport/TCP (IP + port) ; L7 = application/HTTP (URL, headers…).
- **upstream** : en conf Nginx, le bloc qui **groupe les serveurs backend** à répartir.
- **`proxy_pass`** : directive Nginx qui fait suivre la requête vers le backend.
- **`server_name`** : le nom de domaine géré par un bloc `server`.
- **location `/api/`** : dire « pour toutes les requêtes dont le chemin commence par `/api/` ».
- **Fichier statique** : contenu fixe (HTML/CSS/JS) servi tel quel, sans calcul serveur.
- **Spring Boot / Angular / PostgreSQL** : ton backend Java, ton frontend JavaScript, ta base (rappel Bloc 1/3).
- **Docker / Kubernetes** : conteneurs et orchestration (blocs 9-10) où ces notions reviennent avec Traefik/Ingress.

---

### 🧪 À faire maintenant (10 min) — un reverse proxy qui tourne pour de vrai

> Objectif : faire tourner Nginx et le voir servir + faire reverse proxy. **Machine de test** (VM/WSL).

```bash
# 1) installe Nginx
sudo apt update && sudo apt install nginx -y

# 2) vérifie qu'il tourne et qu'il répond
sudo systemctl status nginx --no-pager | head -5
curl -s -I http://localhost/ | head -1    # attendu : HTTP/1.1 200 OK

# 3) un petit backend de test sur le port 8080 (dans un autre terminal)
mkdir -p /tmp/site && echo "Je suis le backend" > /tmp/site/index.html
python3 -m http.server 8080 --directory /tmp/site
```

Maintenant, configure Nginx pour **renvoyer `/api` vers ce backend**. Pied à pied dans `/etc/nginx/sites-available/mon-site` (voir la section suivante), puis :
```bash
sudo nginx -t && sudo systemctl reload nginx
curl http://localhost/api/          # doit afficher "Je suis le backend" (via le proxy)
```

**Ce que tu dois observer / écrire dans ta tête** :
- `/` sert un fichier (Nginx comme serveur web), `/api` est **renvoyé** au backend (reverse proxy).
- `nginx -t` valide la config **avant** de recharger : ça évite de casser le site.

---

## 3. Exemples concrets

### 3.1 Servir le frontend + reverse proxy vers le backend (Nginx)

Fichier `/etc/nginx/sites-available/mon-app` (ou `nginx.conf` selon ta distro) :

```nginx
# Le bloc "server" définit un site (= un port + un server_name)
server {
    listen 80;                        # Nginx écoute sur le port 80 (HTTP)
    server_name mon-site.local;       # le nom de domaine géré

    # 1) On sert le frontend Angular (fichiers compilés)
    location / {
        root /var/www/mon-app/front/dist;   # dossier des fichiers Angular compilés
        try_files $uri /index.html;         # si fichier inconnu, servir l'app Angular
    }

    # 2) Pour le chemin /api → on renvoie vers le backend Spring Boot
    location /api/ {
        proxy_pass http://localhost:8080;   # le backend Spring Boot
        proxy_set_header Host $host;        # transmet le nom du site au backend
    }
}
```

### 3.2 Vérifier la config et recharger

```bash
sudo nginx -t                              # teste la syntaxe de la config
sudo systemctl reload nginx                # recharge Nginx sans redémarrer
curl -I http://mon-site.local/             # test du frontend
curl http://localhost:8080/api/health      # test direct du backend
```

> 💡 **À quoi sert `nginx -t` ?** Il valide la syntaxe **avant** de recharger, pour ne jamais casser un site en production avec une faute de frappe.

### 3.3 Ajouter un load balancer (plusieurs backends)

```nginx
# upstream = le groupe de serveurs backend entre lesquels Nginx répartit
upstream backend_spring {
    server localhost:8081;   # backend 1
    server localhost:8082;   # backend 2
}

server {
    listen 80;
    server_name mon-site.local;

    location /api/ {
        proxy_pass http://backend_spring;   # on envoie vers le GROUPE de backends
    }
}
```

**Vérifier la répartition** : si les deux backends répondent `Hello from 8081` / `Hello from 8082`, en appelant plusieurs fois l'API tu verras les deux réponses alterner (round-robin) :

```bash
for i in 1 2 3 4; do curl -s http://localhost/api/who; echo; done
# Hello from 8081
# Hello from 8082
# Hello from 8081
# Hello from 8082
```

> 💡 **Le « quand utiliser round-robin »** : parfait pour des backends identiques et sans état. Sinon, on utilisera les techniques de session vue plus haut.

---

## 4. Bonnes pratiques modernes (2025-2026)

- **Placer Nginx/Traefik devant chaque application web** : un seul point d'entrée, moins d'attaques, TLS centralisé.
- **Terminer le TLS au reverse proxy** : le chiffrement (Leçon 4) se gère à l'accueil ; le backend peut rester en HTTP interne.
- **Déclarer des health checks** (`/health`) sur chaque backend et les surveiller (bloc 12, observabilité).
- **Limiter la surface exposée** : seul le proxy est accessible de l'extérieur ; le backend écoute sur `localhost` ou un réseau privé, jamais en public.
- **Utiliser `upstream` + round-robin pour la montée en charge**, et penser à la session si l'app a un état.
- **Tester la config (`nginx -t`) avant de recharger** pour éviter une coupure.
- **En conteneurs** (bloc 9-10), Traefik se révèle très pratique pour gérer les proxy + certificats automatiquement.

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi | ✅ Version correcte |
|----------------|----------|---------------------|
| Exposer le backend (8080) directement au public | Surface d'attaque, contournement du proxy | Backend sur `localhost`/réseau privé, seul le proxy est public |
| Oublier `nginx -t` avant reload | Un reload avec une faute casse le site | `sudo nginx -t` puis reload |
| Backends avec état mais round-robin simple | Les sessions se perdent (l'utilisateur « change de serveur ») | Sticky session OU stocker l'état hors des backends (Redis) |
| Ne pas définir de health check | Un backend mort reçoit encore du trafic | Déclarer `/health` et monitorer |
| `proxy_pass` sans transmettre les headers | Le backend ne connaît pas le vrai client / le nom du site | `proxy_set_header Host $host; X-Forwarded-For ...` |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`**, la correction dans **`03-correction.md`**.

**Énoncé court** : sur une machine de test, installe Nginx, sers un simple fichier HTML (frontend fictif) sur le port 80, puis ajoute un bloc `location /api/` qui fait reverse proxy vers un service local (ex. `python3 -m http.server` sur 8080). Enfin, définis un `upstream` avec deux ports et vérifie la répartition avec `curl`.

---

## 7. Correction détaillée de l'exercice

> La correction complète pas-à-pas est dans **`03-correction.md`**. Le raisonnement principal :
> - on **installe et vérifie Nginx** (`nginx -t`, `systemctl`),
> - on **sépare** ce que le proxy sert (`/` = fichiers statiques) de ce qu'il renvoie (`/api` = backend),
> - on **confirme** chaque comportement avec `curl` (statique vs proxy vs répartition).

---

## 8. Checklist de validation

- [ ] J'explique ce qu'est un reverse proxy et pourquoi on le place devant une app.
- [ ] Je distingue serveur web / reverse proxy / load balancer.
- [ ] Je définis health check, failover, session, L4/L7.
- [ ] J'écris une config Nginx : `location /` (statique) + `location /api/` (reverse).
- [ ] J'écris un bloc `upstream` et je vérifie la répartition avec `curl`.
- [ ] Je cite les outils (Nginx, Traefik, HAProxy, Apache mod_proxy) et leur usage.

---

🧭 **Pont vers la suite** — Nous savons maintenant **exposer** une app proprement (proxy, LB) et contrôler qui peut la joindre. Mais savoir *qui a le droit de faire quoi* dans un système (utilisateurs, rôles, clés) et **ne pas fuiter de secrets** est un autre pilier : c'est la Leçon 7, **contrôle d'accès et secrets**.

---

*Prochaine étape :* Leçon 7 — **Contrôle d'accès et secrets** dans `07-Controle-acces-et-secrets`.