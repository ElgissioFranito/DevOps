# Référence rapide — Leçon 5 : Reverse Proxy & Load Balancing

> Bloc 5 · Leçon 5 — Aide-mémoire.

## Architecture cible
```
Internet
   ▼
 Nginx (proxy + TLS + LB)
   ├── sert ────────> Angular (fichiers statiques)
   └── /api ───────> Spring Boot (backend)
                         ▼
                     PostgreSQL (jamais exposée)
```

## Concepts
- **Reverse proxy** : point d'entrée unique qui renvoie la requête au bon service.
- **Load balancer** : répartit le trafic entre plusieurs serveurs.
- **Health check** : vérifier qu'un backend répond (`/health`).
- **Failover** : bascule vers un backend sain si un autre tombe.
- **Sticky session** : coller un client au même backend (si état).
- **L4** (transport/TCP) vs **L7** (application/HTTP).
- **Round-robin** : un tour chacun.

## Nginx — directives clés
| Directive | Rôle |
|-----------|------|
| `listen 80` | port écouté |
| `server_name` | domaine géré |
| `root ...` | dossier des fichiers statiques |
| `location /api/` | pour les chemins commençant par `/api/` |
| `proxy_pass http://localhost:8080` | renvoie vers le backend |
| `upstream { server ... ; server ...; }` | groupe de backends (LB) |
| `nginx -t` | tester la syntaxe avant reload |

## Réflexes
1. `sudo nginx -t` **puis** `sudo systemctl reload nginx`.
2. Backend sur `localhost`/réseau privé, jamais public.
3. TLS terminé au proxy (Leçon 4).
4. Health checks sur chaque backend.