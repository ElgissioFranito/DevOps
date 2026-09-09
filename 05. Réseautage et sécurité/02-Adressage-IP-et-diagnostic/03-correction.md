# Correction — Leçon 2 : Adressage IP et diagnostic réseau

> **Bloc 5 · Leçon 2** — Correction pas à pas.

---

## Étape 1 — Connaître sa machine

```bash
ip addr show
# 2: wlan0: ... inet 192.168.1.42/24 ...
ip route show
# default via 192.168.1.1 dev wlan0
```

**Explication** : ton IP `192.168.1.42` avec `/24` = masque `255.255.255.0`. La gateway `192.168.1.1` est la sortie par défaut (route par défaut = `default via`).

---

## Étape 2 — Test de liaison

```bash
ping -c 4 127.0.0.1   # tout fonctionne normalement même en panne réseau (boucle locale)
ping -c 4 192.168.1.1 # agrafe locale : ~1-2 ms
ping -c 4 8.8.8.8     # Internet : ~10-30 ms
```

**Explication** :
- `127.0.0.1` = loopback, ça répond **toujours** (même sans réseau). Bon point de départ pour valider la pile TCP/IP locale.
- Gateway responsive = micro-réseau OK.
- `8.8.8.8` responsive = sortie Internet OK (niveau routage IP).

---

## Étape 3 — Traceroute

```bash
traceroute -m 5 example.com
#  1  192.168.1.1 ...
#  2  ...
```

**Explication** : chaque ligne = un routeur (saut). S'il y a des `*`, c'est souvent parce que le routeur filtre ICMP, pas un signe de coupure. Combien de sauts : en local court (2-4), vers l'internet plus (parfois 10-15 en laissant plus de `-m`).

---

## Étape 4 — IP publique

```bash
curl -4 https://api.ipify.org
# ex. 203.0.113.7  (une IP publique → la tienne vue d'Internet)
```

**Explication** : ton IP privée (192.168.x) n'est **pas** routable depuis Internet. Le routeur / NAT traduit ton IP privée en IP **publique** pour sortir. D'où : IP privée ≠ IP publique, les deux sont bien distinctes.

---

## Étape 5 — Plages

1. `192.168.1.100` dans `192.168.1.0/24` → **oui**. Le `/24` couvre `192.168.1.0 → .255`.
2. `10.0.5.9` dans `10.0.0.0/8` → **oui**. `/8` = tout le `10.x.x.x`.
3. `172.16.0.1` dans `172.16.0.0/12` → **oui**. `/12` couvre `172.16.0.0 → 172.31.255.255`.
4. `8.8.8.8` dans `192.168.1.0/24` → **non**. Les premiers octets différent (8 vs 192) ; pas dans la plage.

---

## Checklist de validation (leçon 2)

- [ ] Je lis mon IP/masque/gateway avec `ip addr` / `ip route`.
- [ ] Je teste la liaison locale, la gateway, et Internet avec `ping -c`.
- [ ] Je trace un itinéraire avec `traceroute` et j'interprète les sauts.
- [ ] Je récupère mon IP publique.
- [ ] Je détermine si une adresse est dans une plage `/8`, `/12`, `/24`.

---

## 🧠 Conseils pour la suite

- **La cascade** : soi → gateway → Internet → DNS → service. Mémorise-la, tu la retrouveras à chaque incident.
- Les masques en `/X` te suivront dans tout le cloud (VPC, subnets AWS), ne stresse pas de les comprendre vraiment.
- **Ne scanne jamais** un réseau dont tu n'es pas propriétaire (Leçon 3 approfondit le pare-feu).