# Correction — Leçon 5 : Reverse Proxy et Load Balancing

> **Bloc 5 · Leçon 5** — Correction pas à pas.

---

## Étape 1 — Installation

```bash
sudo apt update
sudo apt install nginx -y   # -y = répondre "oui" automatiquement
sudo systemctl status nginx # vérifier que le service tourne
```

## Étape 2 — Frontend statique

```bash
mkdir -p /tmp/mon-site && echo "<h1>Bienvenue</h1>" > /tmp/mon-site/index.html
```

Dans `/etc/nginx/sites-available/mon-site` :
```nginx
server {
    listen 80;
    server_name _;                      # "_" = accepter tout nom non reconnu
    root /tmp/mon-site;                 # dossier des fichiers servis
    index index.html;
}
```

```bash
sudo nginx -t                 # teste la syntaxe
sudo systemctl reload nginx   # recharge sans couper
curl -I http://localhost/     # doit afficher HTTP/... 200
```

## Étape 3 — Reverse proxy `/api`

Lance le backend (2e terminal) :
```bash
python3 -m http.server 8080 --directory /tmp/mon-site
```

Ajoute le reverse proxy :
```nginx
server {
    listen 80;
    server_name _;
    root /tmp/mon-site;

    location /api/ {
        proxy_pass http://localhost:8080;   # renvoie à mon backend
    }
}
```
```bash
sudo nginx -t && sudo systemctl reload nginx
curl http://localhost/api/      # répond via le backend (le serveur python)
```

## Étape 4 — Load balancing

Deux backends (2 terminaux) :
```bash
python3 -m http.server 8080 --directory /tmp/mon-site
python3 -m http.server 8081 --directory /tmp/mon-site   # même dossier
```
```nginx
upstream backend {
    server localhost:8080;
    server localhost:8081;
}
server {
    listen 80;
    server_name _;
    root /tmp/mon-site;
    location /api/ { proxy_pass http://backend; }
}
```
```bash
sudo nginx -t && sudo systemctl reload nginx
for i in 1 2 3 4; do curl -s http://localhost/api/; echo; done
# la réponse alterne entre 8080 et 8081 (round-robin) même si ici le contenu est identique
```

## Étape 5 — Réflexion

**Pourquoi le proxy est-il le seul accès exposé ?**
> Principe de **moindre exposition** (Leçon 3) : on réduit la surface d'attaque à un seul point (80). Le backend reste inaccessible de l'extérieur, protégé par le proxy et le pare-feu.

**Que se passe-t-il si tu coupes un backend ?**
> Coupe le backend 8081 (Ctrl+C). Re-teste `curl http://localhost/api/` : Nginx **bascule automatiquement** sur l'autre backend sain (principle du **failover**). L'utilisateur ne voit pas la panne — c'est tout l'intérêt du LB.

---

## Checklist de validation (leçon 5)

- [ ] J'installe Nginx et je vérifie son état.
- [ ] Je sers un frontend statique (`root` / `index`).
- [ ] J'ajoute un `location /api/` en reverse proxy (proxy_pass).
- [ ] J'utilise `nginx -t` + `systemctl reload`.
- [ ] Je définis un `upstream` à 2 serveurs et je vois la répartition.
- [ ] J'explique le failover quand un backend tombe.

---

## 🧠 Conseils pour la suite

- En prod, ajoute le **TLS** (Leçon 4) dans ce même bloc `server` (listen 443 ssl…), le proxy est le point idéal.
- Surveille les health checks (Bloc 12, observabilité).
- Dans Docker/Kubernetes, ces « upstream » deviennent des services/Traefik/Ingress (Blocs 9-10).