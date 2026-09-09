# Référence rapide — Leçon 2 : Adressage IP & diagnostic

> Bloc 5 · Leçon 2 — Aide-mémoire.

## Concepts
- **IPv4** : `192.168.1.10` (32 bits).
- **CIDR** : `192.168.1.0/24` = 256 adresses (masque `/24`).
  - `/8` = 16 M d'adresses, `/16` = 65 536, `/24` = 256, `/32` = 1 adresse.
- **Gateway** = routeur de sortie du subnet (`default via ...`).
- IP **privée** : `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`.
- IP **publique** : routable sur Internet, attribuée par un FAI.

## Commandes

| Besoin | Commande |
|--------|----------|
| Adresse + masque | `ip addr show` |
| Gateway / routes | `ip route show` |
| Test de base | `ping -c 4 host` |
| Trace de l'itinéraire | `traceroute host` / `tracepath host` |
| IP privée de la machine | `hostname -I` | Affiche les IP de ta machine |
| IP publique | `curl -4 https://api.ipify.org` | IP vue d'Internet |

## Cascade diagnostic

```
1. 127.0.0.1 (loopback)      → pile locale OK ?
2. IP du LAN (ex. 192.168.1.1)→ réseau local ?
3. Gateway                    → sortie locale ?
4. IP publique (8.8.8.8)      → Internet ?
5. DNS (dig example.com)      → name ?
6. Service (curl / nc port)   → app OK ?
```

## IPv6 & IPSec (survol)
- IPv6 : adresses en hexa `2001:db8::1`, plus grand espace, pas urgent.
- IPSec : suite de protocoles pour chiffrer/authentifier le trafic IP (VPN site-à-site).