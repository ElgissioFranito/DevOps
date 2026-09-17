# Référence rapide — Leçon 5b : tunnels & exposition de services

> Bloc 5 · Leçon 5b — Aide-mémoire. Les commandes **VPN (WireGuard/OpenVPN)** sont dans la fiche de la **Leçon 5a**.

---

## 🧰 Tunnels SSH (ponctuel)

```bash
ssh -L 5433:localhost:5432 toto@serveur  # -L local : mon 5433 → le 5432 vu depuis le serveur
ssh -R 8080:localhost:3000 toto@serveur  # -R remote (reverse) : le 8080 du serveur → mon 3000
ssh -D 1080 toto@serveur                 # -D dynamic : proxy SOCKS local sur 1080
```

| Option | Sens | Usage typique |
|---|---|---|
| `-L` | je vais **chercher** un port distant | lire une base privée, joindre un service interne |
| `-R` | je **pousse** un port local | montrer une app de dev (PC derrière le NAT) |
| `-D` | relais **générique** (SOCKS) | faire sortir un navigateur via le serveur |

## 🌩️ Cloudflare Tunnel / ngrok

```bash
python3 -m http.server 3000          # petit serveur local de test (port 3000)
cloudflared tunnel --url http://localhost:3000   # quick tunnel : URL https aléatoire, sans compte
sudo cloudflared service install     # usage durable : cloudflared en service systemd (il faut un compte/domaine)
ngrok http 3000                      # alternative générique (compte gratuit requis)
```

> ⚠️ `trycloudflare.com` / ngrok = **tests seulement** : URL aléatoire, publique, sans garantie. Production = domaine à soi + authentification devant.

## 🔍 Diagnostic rapide

| Symptôme | Cause probable | Action |
|---|---|---|
| « connection refused » sur le port local | tunnel coupé (Ctrl+C, réseau perdu) | relancer la commande `ssh -L/-R/-D` |
| `-R` : le port 8080 n'écoute pas sur le serveur | SSH n'écoute que `localhost` (défaut, sain) | normal ; accès public = Cloudflare Tunnel, pas `GatewayPorts` |
| Le tunnel « meurt » après inactivité | NAT/NAT timeout (Leçon 5a §2.2) | `ServerAliveInterval=60` dans `~/.ssh/config` (côté client) |
| URL trycloudflare ne répond plus | le processus `cloudflared` est arrêté | relancer ; pour du durable, passer en mode service |
| Un port `-R` traîne sur le serveur | tunnel oublié | `ss -tlnp` pour repérer, puis tuer la session SSH |

## 🔐 Rappels sécurité

- Un tunnel `-R` est un **trou dans le pare-feu que tu as toi-même creusé** : ferme-le après usage.
- N'expose **jamais** une base (5432, 3306) via une URL publique : `ssh -L` ou VPN.
- En entreprise : exposer un service interne via ngrok/Cloudflare **sans autorisation** est une faille — passe par le VPN (Leçon 5a).
