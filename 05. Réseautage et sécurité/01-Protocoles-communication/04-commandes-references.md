# Référence rapide — Leçon 1 : Protocoles

> Bloc 5 · Leçon 1 — Aide-mémoire à imprimer mentalement.

## Niveaux à garder en tête
```
Nom (api.example.com)
   │  DNS
   ▼
IP (192.168.x.x)
   │  route / firewall
   ▼
Port (443)
   │  TCP/UDP
   ▼
Service / application (HTTP, SSH…)
```

## Commandes

| Besoin | Commande | Note |
|--------|----------|------|
| Headers d'une API | `curl -I URL` | HEAD |
| Corps d'une API | `curl -i URL` | inclut headers + corps |
| Envoyer du JSON | `curl -X POST URL -H "Content-Type: application/json" -d '{...}'` | `-X` = méthode, `-H` = en-tête, `-d` = données |
| Résoudre un nom | `dig +short example.com` | IP courte |
| Résolution verbeuse | `nslookup example.com` | |
| Tester un port | `nc -zv host port` | `-z` = pas de données |
| Se connecter manuellement | `telnet host port` | en clair, à éviter pour sensible |
| Ports en écoute | `ss -tulpn` | ou `netstat -tulpn` |

## Ports courants

| Port | Service | |
|------|---------|--|
| 22 | SSH | |
| 80 | HTTP | |
| 443 | HTTPS ! | |
| 5432 | PostgreSQL | |
| 6379 | Redis (base mémoire) | |
| 27017 | MongoDB | |

## Réflexe diagnostic

1. `dig +short host` → DNS
2. `nc -zv host 443` → port
3. `curl -I https://host/path` → application
4. Si tout échoue → route / firewall (Leçon 3).