# Correction — Leçon 5 : VPN et tunnels sécurisés

> **Bloc 5 · Leçon 5** — Correction pas à pas.
> 🧭 **Articulation** : compare ta réalisation avec celle-ci ; si un test a échoué, va directement à la table de **diagnostic** (section 2), puis reviens à la checklist (section 3).

---

## 1. Correction pas à pas

### Étape 1 — Les clés (serveur)

```bash
sudo apt update && sudo apt install -y wireguard wireguard-tools qrencode
# installe WireGuard, wg-quick (l'outil de config) et qrencode (QR code)

cd /etc/wireguard
# dossier de config de WireGuard, réservé à root (droits déjà restrictifs)

wg genkey | sudo tee server.key | wg pubkey | sudo tee server.pub
# wg genkey      → génère une clé privée (aléatoire) sur la sortie standard
# tee server.key → l'écrit dans server.key ET le laisse passer à la commande suivante
# wg pubkey      → calcule la clé publique correspondante (lue depuis l'entrée standard)
# tee server.pub → sauvegarde la clé publique

sudo sh -c 'umask 077; wg genkey > client1.key; wg pubkey < client1.key > client1.pub'
# umask 077 → les fichiers créés auront des permissions 600 (réflexe Leçon 4 sur les clés)
# une paire client1 = un appareil. Pour le téléphone, tu feras client2.
```

❓ **Réponse à la question de l'étape 1** : une clé par appareil permet de **révoquer un seul appareil** (supprimer son bloc `[Peer]` côté serveur) sans couper les autres — c'est le moindre privilège (Bloc 2) appliqué au réseau.

### Étape 2 — Les configs

Côté **serveur**, `/etc/wireguard/wg0.conf` :

```ini
[Interface]
Address = 10.66.66.1/24
# l'IP PRIVÉE du serveur dans le tunnel (on a choisi le réseau 10.66.66.0/24)
ListenPort = 51820
# le port UDP d'écoute (standard WireGuard)
PrivateKey = <contenu de server.key — affiché par : sudo cat /etc/wireguard/server.key>
# la clé privée du serveur : NE JAMAIS la partager ni la committer

[Peer]
PublicKey = <contenu de client1.pub>
# la clé PUBLIQUE du client : elle, peut circuler (c'est son rôle, Leçon 4)
AllowedIPs = 10.66.66.2/32
# « ce peer aura l'IP 10.66.66.2 dans le tunnel » — une IP par peer
```

Côté **client**, `/etc/wireguard/wg0.conf` :

```ini
[Interface]
Address = 10.66.66.2/24
# l'IP privée du client (cohérente avec l'AllowedIPs annoncé au serveur)
PrivateKey = <contenu de client1.key>
# la clé privée du client

[Peer]
PublicKey = <contenu de server.pub>
# la clé publique du serveur
Endpoint = 203.0.113.10:51820
# l'IP PUBLIQUE (ou le domaine) de ton serveur + son port — remplace par la tienne
AllowedIPs = 10.66.66.0/24
# SPLIT TUNNEL : seul le réseau du tunnel y passe, le reste sort normalement
PersistentKeepalive = 25
# un petit « je suis vivant » toutes les 25 s : maintient le tunnel derrière une box/NAT
```

**Pare-feu du serveur** (Leçon 3) :

```bash
sudo ufw allow 51820/udp
# un SEUL port ouvert : celui du VPN, en UDP
sudo ufw enable
```
### Étape 3 — Montage et vérifications

```bash
sudo wg-quick up wg0
# à faire des DEUX côtés ; en cas d'erreur, le message indique la ligne fautive

sudo systemctl enable wg-quick@wg0
# persiste le tunnel côté serveur (au reboot, il se remonte tout seul)

sudo wg show
# côté serveur : client1 (10.66.66.2), trafic rx/tx, heure du dernier handshake
# côté client : peer serveur (10.66.66.1), trafic, handshake
# PAS de handshake ? → table de diagnostic (section 2) ci-dessous

ping -c 3 10.66.66.1
# depuis le client : une réponse = le tunnel fonctionne
```

**SSH accessible seulement via le tunnel** (le but : fermer SSH au public) :

```bash
sudo ufw allow in on wg0 to any port 22 proto tcp
# SSH accepté UNIQUEMENT si le paquet arrive par l'interface du tunnel
# ⚠️ Teste d'abord la connexion via le tunnel, AVANT de fermer le SSH public,
# et garde l'accès console du VPS en secours

sudo ufw deny 22/tcp
# puis ferme le SSH public
```

Test final : depuis le client, `ssh toto@10.66.66.1` **fonctionne** (via le tunnel), tandis que `ssh toto@203.0.113.10` (IP publique) est **refusé**.

### Étape 4 — La preuve du périmètre

```bash
curl https://ifconfig.me
# SPLIT tunnel (AllowedIPs = 10.66.66.0/24) → ta vraie IP (le web sort en clair)
# FULL tunnel (AllowedIPs = 0.0.0.0/0)     → l'IP du serveur (tout passe dans le tunnel)
```

❓ **Réponse (étape 4 de l'exercice)** : avantage du full tunnel sur un Wi-Fi public → **tout** est chiffré de bout en bout, y compris la navigation web ; inconvénient au quotidien → toute ta connexion dépend du serveur (lenteur, indisponibilité, certains services se voient géo-restreints). D'où le **split par défaut**.

---

## 2. Diagnostic (si un test a échoué)

| Symptôme | Cause probable | Action |
|---|---|---|
| Pas de handshake des deux côtés | port fermé au pare-feu | `sudo ufw status` : 51820/udp ouvert ? |
| Handshake côté client, pas côté serveur | `PublicKey`/`Endpoint` croisés | le client doit porter `server.pub` ; le serveur, `client1.pub` |
| Handshake ok mais `ping` échoue | `AllowedIPs` incohérents | serveur : `/32` du peer ; client : `10.66.66.0/24` |
| Le tunnel « meurt » après inactivité | pas de keepalive | `PersistentKeepalive = 25` côté client, puis redémarre le tunnel |
| Full tunnel : sites lents/cassés | MTU mal négociée | `MTU = 1280` dans `[Interface]` du client |

---

## 3. Checklist de validation

- [ ] J'ai une paire de clés par appareil, permissions 600.
- [ ] Config serveur + client avec `AllowedIPs` cohérents.
- [ ] Pare-feu : un seul port ouvert (51820/udp).
- [ ] `wg show` montre handshake + trafic ; `ping 10.66.66.1` répond.
- [ ] `systemctl enable wg-quick@wg0` côté serveur.
- [ ] Je prouve split vs full avec `curl https://ifconfig.me`.
- [ ] SSH joignable via le tunnel, fermé au public.

## 🧠 Conseils pour la suite

- Au **Bloc 6 (VPC)**, tu utiliseras exactement cette mécanique pour joindre une base managée (RDS) sans l'exposer : ton poste dans le tunnel, la base joignable depuis l'IP privée du tunnel.
- Tu découvriras aussi les **tunnels managés du cloud** (Site-to-Site VPN, Client VPN) : le concept est identique, seul le service change.
- Quand tu auras 3-4 appareils, scripte la création de clients (une paire de clés + un bloc `[Peer]`) — et retiens : la **Leçon 6 (reverse proxy)** peut aussi se placer derrière un VPN quand les services ne sont pas destinés au public.

