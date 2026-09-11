# Référence rapide — Leçon 5 : VPN & tunnels sécurisés

> Bloc 5 · Leçon 5 — Aide-mémoire.

---

## 🗝️ WireGuard en 6 commandes

```bash
wg genkey                          # génère une clé privée (aléatoire, 256 bits)
wg pubkey                          # lit une clé privée (stdin) → affiche la clé publique
sudo wg-quick up wg0               # monte l'interface wg0 (lit /etc/wireguard/wg0.conf)
sudo wg-quick down wg0             # démonte l'interface
sudo wg show                       # état : clés, peers, trafic, dernier handshake
sudo systemctl enable wg-quick@wg0 # persiste le tunnel au redémarrage (service systemd)
```

## ⚙️ gabarit `/etc/wireguard/wg0.conf` (serveur)

```ini
[Interface]
Address = 10.66.66.1/24        # IP privée du serveur DANS le tunnel
ListenPort = 51820             # port UDP d'écoute
PrivateKey = <server.key>      # clé privée du serveur (secret !)

[Peer]                         # un bloc par appareil
PublicKey = <client1.pub>      # clé publique du client
AllowedIPs = 10.66.66.2/32     # l'IP que ce peer porte dans le tunnel
```

## ⚙️ gabarit `/etc/wireguard/wg0.conf` (client)

```ini
[Interface]
Address = 10.66.66.2/24        # IP privée du client (même réseau que le serveur)
PrivateKey = <client1.key>     # clé privée du client (secret !)

[Peer]
PublicKey = <server.pub>       # clé publique du serveur
Endpoint = <IP-publique-serveur>:51820  # où le joindre sur Internet
AllowedIPs = 10.66.66.0/24     # SPLIT tunnel (0.0.0.0/0 = FULL tunnel)
PersistentKeepalive = 25       # maintient le tunnel derrière une box/NAT
```

## 🔥 Pare-feu (Leçon 3)

```bash
sudo ufw allow 51820/udp                     # n'ouvrir QUE le port du VPN
sudo ufw allow in on wg0 to any port 22 proto tcp  # SSH accessible SEULEMENT via le tunnel
sudo ufw status verbose                      # vérifier
```

## 🧰 Tunnels SSH (ponctuel)

```bash
ssh -L 5433:localhost:5432 toto@serveur  # -L local : mon 5433 → le 5432 vu depuis le serveur
ssh -R 8080:localhost:3000 toto@serveur  # -R remote : le 8080 du serveur → mon 3000
ssh -D 1080 toto@serveur                 # -D dynamic : proxy SOCKS local sur 1080
```

## 🔍 Diagnostic rapide

| Symptôme | Cause probable | Commande / action |
|---|---|---|
| Pas de handshake | port fermé / mauvais Endpoint | `sudo ufw status` ; vérifier IP publique du serveur |
| Handshake ok mais ping échoue | `AllowedIPs` incohérents | comparer serveur (`/32` du peer) et client (`/24`) |
| Le tunnel « meurt » après inactivité | pas de keepalive derrière NAT | `PersistentKeepalive = 25` côté client |
| Full tunnel : certains sites lents/cassés | MTU mal négociée | `MTU = 1280` dans `[Interface]` (client) |
| Clé refusée à la connexion | fichier de clé avec espaces/permissions | `sudo chmod 600 *.key` ; contenu exact via `sudo cat` |

## 🔐 Rappels sécurité

- Fichiers de clés : `chmod 600`, jamais dans Git (Leçon 7).
- Un appareil = une clé = un bloc `[Peer]` → révocable individuellement.
- Teste ta sortie : `curl https://ifconfig.me` (split = ta vraie IP ; full = l'IP du serveur).
## 🐘 OpenVPN (quand l'environnement l'exige)

```bash
sudo apt install -y openvpn easy-rsa
make-cadir ~/pki-openvpn && cd ~/pki-openvpn
# make-cadir crée un dossier de travail pour la PKI (Leçon 4)
./easyrsa init-pki && ./easyrsa build-ca nopass
# build-ca crée l'autorité de certification (comme la CA auto-signée, Leçon 4)
./easyrsa build-server-full serveur nopass && ./easyrsa build-client-full client1 nopass
# certificats signés pour le serveur et un client (nopass = sans phrase de passe)
./easyrsa gen-dh
# paramètres d'échange de clés (Diffie-Hellman, ancêtre du mécanisme de TLS)
```

Gabarit minimal `/etc/openvpn/server/server.conf` :

```
port 1194
proto udp
dev tun
ca ca.crt
cert serveur.crt
key serveur.key
dh dh.pem
server 10.8.0.0 255.255.255.0
keepalive 10 120
persist-key
persist-tun
verb 3
```

Client : un fichier `.ovpn` (ca + cert + key intégrés) puis :

```bash
sudo openvpn --config client.ovpn
```

> 📌 WireGuard fait la même chose en ~15 lignes sans CA → choix par défaut ; OpenVPN = compatibilité.

