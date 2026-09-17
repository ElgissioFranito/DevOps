# Leçon 5a — VPN : accéder à un réseau privé à distance (WireGuard)

> **Bloc 5 · Réseautage et sécurité** — Leçon 5a (le VPN, première moitié de la Leçon 5)
> 🧭 **Pont depuis la Leçon 4 (TLS)** : tu sais désormais chiffrer une session web grâce à une paire de clés (publique/privée) et à des certificats. Cette leçon réutilise **exactement** ces mécanismes — mêmes idées de chiffrement, mêmes réflexes sur les clés — mais pour un autre but : **relier ta machine (ou un réseau entier) à un réseau distant comme s'il y était physiquement branché**. C'est le **VPN**, un outil que tu utiliseras **tous les jours** (ordinateur, téléphone, serveurs) et qui servira aussi au Bloc 6 (cloud).
> 👉 **Fil rouge de la leçon** : monter ton **propre serveur VPN** avec **WireGuard**, y connecter ton ordinateur et ton téléphone, et ne rien exposer d'inutile (rappel pare-feu, Leçon 3).
> 🧭 **Où sont passés les tunnels SSH et Cloudflare Tunnel ?** Ils vivent dans la **Leçon 5b** (`05b-Tunnels-et-exposition-services/`) : on sépare volontairement le **VPN** (persistant, réseau entier) des **tunnels ponctuels** (une session, un service) — deux outils, deux moments d'apprentissage.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Expliquer** ce qu'est un VPN, ce qu'il protège, et citer des usages quotidiens (Wi-Fi public, administration à distance, accès à un réseau privé) — **et distinguer les deux grandes familles** : VPN d'**accès distant** (ta machine → un réseau) et VPN **site-à-site** (un réseau entier ↔ un autre réseau).
2. **Monter** un serveur VPN **WireGuard** complet : clés, config serveur, config client, service persistant.
3. **Connecter** plusieurs appareils (ordinateur, téléphone) et **vérifier** le tunnel avec `wg show`, `ping`, `curl`.
4. **Régler** le périmètre du tunnel avec **AllowedIPs** : split tunnel vs full tunnel.
5. **Situer** OpenVPN face à WireGuard, et **Tailscale/NetBird** face au WireGuard « brut », et savoir quand choisir l'un ou l'autre.

---

## 2. Explication simple

### 2.1 Le « pourquoi » : Internet est une place publique

Quand ton ordinateur parle sur Internet, ses paquets transitent par des machines dont tu ne maîtrises rien : box de ton fournisseur d'accès, routeurs des opérateurs, points d'accès Wi-Fi. Sur un **Wi-Fi public** (café, aéroport), n'importe qui sur le même réseau peut observer ce qui circule. Et dans l'autre sens : pour joindre depuis l'extérieur un service privé (ton NAS, une base de données), tu dois soit l'exposer sur Internet (dangereux — rappel Leçon 3), soit ne pas y accéder du tout.

**Le VPN résout ces deux problèmes à la fois.**

> 💡 **Analogie** : le **tunnel souterrain**. Internet, c'est la ville en surface : tout le monde peut circuler et te suivre. Un VPN, c'est un **tunnel secret creusé entre ton immeuble et celui de ton entreprise** : tu entres d'un côté, tu ressorts **à l'intérieur** du réseau distant, et personne en surface ne voit ce que tu transportes (tout est **chiffré**). Bonus : depuis le tunnel, tu accèdes aux **pièces privées** de l'immeuble (services internes) qui restent fermées à ceux qui restent en surface.

### 2.2 Le « comment » : une interface réseau virtuelle

Concrètement, le VPN crée sur ta machine une **interface réseau virtuelle** — par exemple `wg0` — en plus de ta vraie carte réseau (`wlan0`, `eth0`). Cette interface reçoit une **IP privée** (rappel Leçon 2 : `10.x.x.x`, `192.168.x.x`). Ensuite, la **table de routage** (rappel Leçon 2 : elle dit où envoyer les paquets) décide ce qui part dans le tunnel :

```
Trafic vers 10.66.66.x  → interface wg0   (tunnel chiffré)
Trafic vers le reste    → interface physique (sortie normale)
```

Le paquet qui entre dans le tunnel est **chiffré** avec les clés échangées à la connexion (même logique que TLS, Leçon 4), enveloppé dans un paquet **UDP** (port **51820** par défaut chez WireGuard), envoyé sur Internet, puis déchiffré à l'arrivée. Pour le réseau distant, tu es **une machine locale de plus** — avec une IP privée du réseau du tunnel.

> 🔎 **Zoom : le NAT, enfin clair** (tu l'as croisé en Leçon 2, et il revient tout le temps dans les discussions VPN — soyons précis).
> - **Quoi ?** **NAT** = *Network Address Translation* : la **traduction d'adresses** faite par ta **box**. Chez toi, tes machines ont des **IP privées** (`192.168.x.x` — rappel Leçon 2) ; elles partagent **une seule IP publique** face à Internet. Ta box se tient **entre les deux** et **traduit** : elle note dans une **table** « la requête du PC 192.168.1.10 vers le port 443 de ce site » et, à la réponse, elle sait **à quelle machine de la maison la renvoyer**.
> - **Analogie** : ta box est la **réception d'un immeuble**. Tout le courrier arrive à l'**adresse unique de l'immeuble** (IP publique) ; la réception regarde le **destinataire sur l'enveloppe** (appartement n°X) pour remettre le colis au bon locataire. Depuis l'extérieur, on ne connaît que l'immeuble — jamais les appartements.
> - **Comment ça nous concerne ici ?** Deux conséquences directes : 1) **de l'extérieur, on ne peut pas « sonner » chez toi spontanément** : ta box ne sait pas à quel appareil transmettre un paquet qui arrive sans demande préalable — c'est pour ça que ton **serveur VPN chez toi a besoin d'une règle de redirection de port** (*port forwarding*, à configurer dans la box) ; 2) **une connexion inactive finit par tomber** : la box « oublie » la ligne de sa table au bout d'un moment — d'où le **`PersistentKeepalive`** de WireGuard (§3.3) : un petit paquet régulier qui dit « je suis toujours là » pour que la box garde la ligne.
> - **Quand le croiseras-tu encore ?** Partout : derrière chaque box/routeur domestique, dans le **cloud** (la NAT Gateway du Bloc 6 fait exactement ce travail pour les sous-réseaux privés), et dans la section 3.7 ci-dessous : le `MASQUERADE` est **le même mécanisme côté serveur** — il réécrit l'adresse des paquets du tunnel avec l'IP du serveur pour qu'ils circulent dans son LAN.

### 2.3 Le « quand » : usages quotidiens

| Situation | Sans VPN | Avec VPN |
|---|---|---|
| Wi-Fi public | trafic observable par les autres clients | tout est chiffré de bout en bout |
| Administration à distance | il faut exposer SSH (22) publiquement | SSH reste fermé au public ; tu l'utilises via l'IP privée du tunnel |
| Homelab / services auto-hébergés | exposés publiquement ou inaccessibles dehors | tu y accèdes comme si tu étais à la maison |
| Cloud (Bloc 6) | base managée inaccessible ou exposée | tu te connectes à la base via le tunnel, sans l'exposer |
| Deux réseaux à relier (bureau ↔ datacenter, entreprise ↔ cloud) | impossible sans exposer des services de part et d'autre | VPN **site-à-site** : les deux réseaux se voient comme un seul (voir §2.6) |

> 🧭 **Lien avec la Leçon 3** : le VPN ne remplace pas le pare-feu — il le complète. Le principe reste le même : **n'exposer au public que le strict nécessaire**. Ici, c'est **un seul port UDP**.

### 2.4 Les outils : trois familles à connaître

| Outil | Créé en | Force | Limite | Quand l'utiliser |
|---|---|---|---|---|
| **WireGuard** | 2015 (intégré au noyau Linux depuis 2020) | très simple, très rapide, crypto moderne | pas de gestion fine des certificats | **choix par défaut** aujourd'hui (perso comme pro) |
| **OpenVPN** | 2001 | universel (Windows, routeurs, pare-feux d'entreprise), PKI à base de certificats (Leçon 4) | plus lourd, config plus longue | quand l'environnement l'exige ou pour la compatibilité |
| **tunnel SSH** | — | déjà installé partout, zéro installation | point-à-point (un port, une session), pas un « réseau » | besoin **ponctuel** |

> 📌 **Règle simple** : ponctuel (un port, une session) → **tunnels SSH — Leçon 5b** ; usage régulier (un réseau, plusieurs services) → **WireGuard** (cette leçon) ; obligation de compatibilité → **OpenVPN**.

### 2.5 Split tunnel vs full tunnel : les `AllowedIPs`

C'est **LA** ligne à comprendre dans une config WireGuard :

```
AllowedIPs = 10.66.66.0/24   → SPLIT TUNNEL : seul ce réseau passe par le tunnel
AllowedIPs = 0.0.0.0/0       → FULL TUNNEL  : TOUT ton trafic passe par le tunnel
```

- **Split tunnel** (défaut recommandé) : seul le trafic destiné au réseau privé emprunte le tunnel ; le reste (navigation web, streaming) sort par ta connexion normale → rapide, ne perturbe rien.
- **Full tunnel** : tout passe par ton serveur VPN — utile sur un **Wi-Fi public** non fiable, ou pour sortir sur Internet avec l'IP du serveur. Contrepartie : toute ta connexion dépend du serveur (lenteur, disponibilité).
### 2.6 Les deux familles de VPN : accès distant vs site-à-site

Jusqu'ici on a parlé d'**une machine** (ton PC, ton téléphone) qui rejoint **un réseau**. Mais un VPN peut aussi relier **deux réseaux entiers**. Ce sont les deux grandes familles à connaître :

| | **VPN d'accès distant** (client → réseau) | **VPN site-à-site** (réseau → réseau) |
|---|---|---|
| **Qui se connecte** | une machine isolée (ton PC, ton téléphone) | deux **passerelles** (routeurs/serveurs VPN), une par réseau |
| **Ce qui est relié** | ta machine au réseau distant | **tout** le réseau A à **tout** le réseau B |
| **Qui configure** | chaque utilisateur installe un client | un admin configure une fois, tout le monde en profite |
| **Exemple typique** | télétravailler depuis chez toi | le bureau de Paris et le datacenter de Lyon se parlent comme voisins |
| **Dans cette leçon** | ✅ c'est ce que tu montes avec WireGuard (sections 3.1 à 3.7) | aperçu en section 3.8, approfondi au Bloc 6 (cloud) |

> 💡 **Analogie** : le VPN d'accès distant, c'est un **tunnel privatif entre TON appartement et l'immeuble de l'entreprise** — toi seul l'empruntes. Le VPN site-à-site, c'est un **passage souterrain fermé entre les DEUX immeubles entiers** : tous les habitants du premier peuvent aller dans le second, sans même s'en apercevoir (pour eux, c'est le même bâtiment). Ta passerelle fait le travail : tu ne configures rien sur ton PC.

**Pourquoi c'est important en DevOps ?** C'est exactement ainsi qu'on relie **un réseau d'entreprise à un cloud privé** (VPC — Virtual Private Cloud, Bloc 6) : le serveur applicatif du bureau atteint la base de données du cloud avec son IP privée, comme si les deux étaient dans la même pièce. C'est aussi ce que faisait déjà ton admin d'entreprise avec **IPSec** (rappel Leçon 2 : suite de protocoles de chiffrement IP, très utilisée pour les VPN site-à-site d'entreprise) — WireGuard permet le même résultat avec une config bien plus simple.

> 🧩 **« VPN personnel », « VPN mobile », « Cloud VPN », « SSL VPN »… : que sont ces noms ?** En recherche, tu croiseras beaucoup d'étiquettes. **Ce ne sont pas de nouvelles architectures** : ce sont des **cas d'usage** ou des **technologies**, qui se ramènent toujours à **accès distant OU site-à-site** :
>
> | Étiquette vue en ligne | En réalité | Notre architecture |
> |---|---|---|
> | VPN « personnel » (NordVPN, ProtonVPN…) | accès distant vers un serveur qui fait sortir ton trafic sur Internet (vie privée) | **accès distant**, full tunnel (§2.5) |
> | VPN « d'accès à distance » / « mobile » | ton PC ou ton téléphone rejoint un réseau ; « mobile » = toujours actif malgré les bascules Wi-Fi/4G | **accès distant** |
> | **SSL VPN** (portails web d'entreprise) | accès distant qui utilise **TLS** (Leçon 4) et parfois un simple navigateur, sans client à installer | **accès distant**, protocole TLS |
> | **Cloud VPN** (AWS Site-to-Site VPN, Azure VPN Gateway…) | site-à-site ou accès distant, **managé par un cloud provider** | **site-à-site** (Bloc 6) |
>
> Retiens la règle : **l'architecture (2 types) + la technologie (WireGuard, OpenVPN, IPSec, TLS) + la finalité (vie privée, télétravail, relier des sites) = toutes les combinaisons que le marketing nomme différemment.**

> 🌐 **Zoom 2025-2026 : les VPN « mesh » — Tailscale & NetBird**. Avec du WireGuard brut, tu échanges les clés et écris les configs **à la main** pour chaque appareil — ingérable au-delà de quelques machines. **Tailscale** et **NetBird** ajoutent **au-dessus du moteur WireGuard** un **serveur de coordination** (un annuaire automatique) qui distribue clés et configs, une **authentification par identité** (tu te connectes avec ton compte Google/Microsoft/GitHub — pas de fichiers de clés à copier), et une **adresse IP privée stable** par machine, où qu'elle soit. Les tunnels se créent **automatiquement** entre les machines qui ont besoin de se parler, souvent en **direct** (peer-to-peer). L'appareil qui rejoint le réseau = accès distant ; un serveur qui le rejoint = un site de plus, relié automatiquement. C'est ce qu'on appelle un réseau **mesh** (chaque machine peut joindre les autres), lié à l'approche **Zero Trust** (Bloc 7 : on autorise par **identité**, pas par « être sur le réseau »). Différences : **Tailscale** = le plus simple, plan de contrôle en service (gratuit en usage perso) ; **NetBird** = **open source complet et auto-hébergeable** (tu héberges ton propre annuaire). En entreprise, tu verras ces outils **plus souvent que le WireGuard brut** — mais le moteur reste celui que tu viens de configurer à la main, et c'est bien de l'avoir fait une fois pour comprendre ce qu'ils automatisent.

---


## 📖 Vocabulaire / Abréviations

| Terme | Définition (une ligne) |
|---|---|
| **VPN** (Virtual Private Network) | réseau privé virtuel : tunnel chiffré entre deux machines ou deux réseaux |
| **Accès distant** | VPN reliant **une machine** à **un réseau** (ton PC → réseau de l'entreprise) |
| **Site-à-site** (site-to-site) | VPN reliant **deux réseaux entiers** via deux passerelles (bureau ↔ datacenter, bureau ↔ cloud) |
| **Passerelle** (gateway VPN) | machine/routeur qui héberge le tunnel pour tout son réseau local |
| **IPSec** | suite de protocoles de chiffrement/authentification IP, historique des VPN site-à-site d'entreprise |
| **Tunnel** | canal chiffré qui « emballe » (encapsule) ton trafic pour traverser Internet sans être lisible |
| **Interface virtuelle** | « carte réseau » logicielle créée par le VPN (`wg0`, `tun0`) |
| **WireGuard** | VPN moderne et minimal, intégré au noyau Linux depuis 2020 |
| **OpenVPN** | VPN historique (2001), basé sur TLS et une PKI |
| **PKI** (Public Key Infrastructure) | système de certificats signés par une autorité (Leçon 4) |
| **AllowedIPs** | les IP que ce peer envoie dans le tunnel (et qu'il accepte d'en recevoir) |
| **Split / full tunnel** | partie du trafic / tout le trafic dans le tunnel |
| **Endpoint** | l'adresse IP:port publique où joindre le serveur VPN |
| **Handshake** | « poignée de main » chiffrée qui établit la session |
| **Keepalive** | petit paquet régulier qui maintient le tunnel ouvert derrière une box/NAT |
| **NAT** (Network Address Translation) | traduction d'adresses par la box/routeur : plusieurs IP privées partagent une IP publique (voir le zoom §2.2) |
| **Port forwarding** | règle dans la box qui redirige un port entrant vers une machine précise du LAN |
| **VPN mesh** | VPN où les machines se trouvent et se connectent **automatiquement** entre elles (Tailscale, NetBird) |
| **Serveur de coordination** | annuaire central d'un VPN mesh : distribue clés, adresses et configs automatiquement |
| **Tailscale / NetBird** | deux VPN mesh populaires construits sur WireGuard ; NetBird est open source et auto-hébergeable |
| **Zero Trust** | modèle de sécurité qui autorise par **identité et contexte**, pas par appartenance à un réseau |
| **Fuite DNS** (DNS leak) | quand les requêtes de noms sortent en clair malgré le VPN |
| **SOCKS** | protocole de proxy générique (utilisé par `ssh -D`) |
| **MTU** (Maximum Transmission Unit) | taille maximale d'un paquet ; mal réglée dans un tunnel, elle casse des connexions |

---

> 🧭 **Transition** : la théorie est posée (pourquoi §2.1, comment §2.2, quand §2.3, quel outil §2.4, quel périmètre §2.5, quelles familles §2.6). Passons à la pratique : tout ce qui suit est **copiable-collable**. On commence par l'installation et les clés — la partie la plus importante, car sans clés cohérentes, rien ne se connectera.

## 3. Exemples concrets

### 3.1 Installer WireGuard et générer les clés (sur le serveur)

```bash
sudo apt update && sudo apt install -y wireguard wireguard-tools qrencode
# installe WireGuard, wg-quick (l'outil de config) et qrencode (QR code pour le téléphone)

cd /etc/wireguard
# dossier de config de WireGuard, réservé à root

wg genkey | sudo tee server.key | wg pubkey | sudo tee server.pub
# wg genkey      → génère une clé privée (aléatoire)
# tee server.key → l'écrit dans server.key ET le transmet à la commande suivante
# wg pubkey      → calcule la clé publique correspondante (lue depuis l'entrée standard)
# tee server.pub → sauvegarde la clé publique

sudo sh -c 'umask 077; wg genkey > client1.key; wg pubkey < client1.key > client1.pub'
# umask 077 → fichiers créés en permissions 600 (réflexe clés, Leçon 4)
# UNE paire PAR appareil : client1 (ton PC), client2 (ton téléphone)...
```

### 3.2 Config du serveur : `/etc/wireguard/wg0.conf`

```ini
[Interface]
# section « qui je suis » côté serveur
Address = 10.66.66.1/24
# l'IP PRIVÉE du serveur dans le tunnel (on choisit le réseau 10.66.66.0/24)
ListenPort = 51820
# le port UDP d'écoute (standard WireGuard)
PrivateKey = <contenu de server.key — affiché par : sudo cat /etc/wireguard/server.key>
# la clé privée du serveur : NE JAMAIS la partager

[Peer]
# un bloc [Peer] par client autorisé — ici client1
PublicKey = <contenu de client1.pub>
# la clé PUBLIQUE du client : elle peut circuler (rôle d'une clé publique, Leçon 4)
AllowedIPs = 10.66.66.2/32
# « ce peer portera l'IP 10.66.66.2 dans le tunnel » — une IP par peer
```

### 3.3 Config du client (ton PC) : `/etc/wireguard/wg0.conf`

```ini
[Interface]
Address = 10.66.66.2/24
# l'IP privée du client (cohérente avec l'AllowedIPs du serveur)
PrivateKey = <contenu de client1.key>
# la clé privée du client

[Peer]
PublicKey = <contenu de server.pub>
# la clé publique du serveur
Endpoint = 203.0.113.10:51820
# l'IP PUBLIQUE (ou domaine) de ton serveur + son port — remplace par la tienne
AllowedIPs = 10.66.66.0/24
# SPLIT TUNNEL (section 2.5) : seul le réseau du tunnel y passe
PersistentKeepalive = 25
# « je suis vivant » toutes les 25 s : maintient le tunnel derrière une box/NAT
```

### 3.4 Le pare-feu (Leçon 3 !) — n'exposer QUE le port VPN

```bash
sudo ufw allow 51820/udp
# un SEUL port ouvert, en UDP — le reste reste fermé par défaut
sudo ufw enable
```

### 3.5 Démarrer et vérifier

```bash
sudo wg-quick up wg0
# monte l'interface wg0 à partir de /etc/wireguard/wg0.conf

sudo systemctl enable wg-quick@wg0
# au redémarrage du serveur, le tunnel se remontera tout seul (systemd, Bloc 2)

sudo wg show
# côté serveur ET client : peers, trafic échangé, heure du dernier « handshake »

ping -c 3 10.66.66.1
# depuis le client : ping le serveur via son IP DE TUNNEL (pas son IP publique)
# -c 3 = seulement 3 paquets (au lieu d'un ping infini)
```

### 3.6 Le téléphone : QR code

```bash
nano ~/client1.conf
# recopie la config client (3.3) dans ce fichier, sur le serveur, pour générer le QR

sudo qrencode -t ansiutf8 < ~/client1.conf
# -t ansiutf8 → affiche le QR code directement dans le terminal
# app WireGuard (iOS/Android) → « + » → scanner → connecté
```

### 3.7 Bonus : atteindre le réseau LOCAL derrière le serveur (homelab)

Par défaut, le tunnel ne mène qu'au **serveur lui-même**. Pour accéder au LAN derrière lui (ex. `192.168.1.0/24`) :

```bash
echo 'net.ipv4.ip_forward=1' | sudo tee /etc/sysctl.d/99-wg.conf && sudo sysctl --system
# active le forwarding : le serveur accepte de ROUTER les paquets d'une interface à l'autre
# sysctl --system = relit les réglages du noyau
```

et ajoute dans `[Interface]` du serveur :

```ini
PostUp = iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
# quand un paquet du tunnel sort vers le LAN, il est réécrit avec l'IP du serveur (NAT, Bloc 6)
# eth0 = vérifie le vrai nom de ton interface publique avec : ip a
PostDown = iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
# retire la règle quand le tunnel descend
```

Puis côté **client**, ajoute le LAN aux `AllowedIPs` : `AllowedIPs = 10.66.66.0/24, 192.168.1.0/24`.

### 3.8 Variante site-à-site : relier deux réseaux entiers avec WireGuard

Rappel (section 2.6) : au lieu de relier **une machine**, on relie **deux réseaux** — chacun derrière sa passerelle WireGuard. Exemple : le **réseau A** (bureau, `192.168.10.0/24`) et le **réseau B** (datacenter, `192.168.20.0/24`), avec un réseau de tunnel `10.77.77.0/24`. Prérequis : **l'IP forwarding activé** sur les DEUX passerelles (technique de la section 3.7 — sans lui, la passerelle ne retransmet pas les paquets).

**Passerelle A** (bureau) — `/etc/wireguard/wg0.conf` :

```ini
[Interface]
Address = 10.77.77.1/24
# l'IP de la passerelle A dans le tunnel
PrivateKey = <clé privée A>
# la clé privée de la passerelle A

[Peer]
PublicKey = <clé publique B>
# la clé publique de la passerelle B (comme un client, mais c'est un réseau entier)
Endpoint = 198.51.100.20:51820
# l'IP PUBLIQUE de la passerelle B (au moins un côté doit connaître l'autre)
AllowedIPs = 10.77.77.0/24, 192.168.20.0/24
# LE point clé : on annonce que tout le RÉSEAU B (192.168.20.0/24)
# est joignable via ce peer — pas une seule IP comme en accès distant
PersistentKeepalive = 25
# maintient le tunnel si la passerelle B est derrière une box/NAT
```

**Passerelle B** (datacenter) — config miroir :

```ini
[Interface]
Address = 10.77.77.2/24
PrivateKey = <clé privée B>

[Peer]
PublicKey = <clé publique A>
Endpoint = 203.0.113.10:51820
# l'IP publique de la passerelle A
AllowedIPs = 10.77.77.0/24, 192.168.10.0/24
# miroir : tout le RÉSEAU A est joignable via ce peer
```

Après `wg-quick up wg0` des deux côtés, **n'importe quelle machine du bureau** peut joindre **n'importe quelle machine du datacenter** (et inversement) — à condition que les machines client aient une **route** vers le réseau distant via leur passerelle (sur la plupart des box/routeurs, c'est automatique : la passerelle VPN est la passerelle par défaut). Vérification :

```bash
ping -c 3 192.168.20.5
# depuis une machine du réseau A : elle atteint une machine du réseau B
# SANS avoir WireGuard installé — c'est sa passerelle qui fait tout le travail
```

> 📌 **Résumé mental** : en accès distant, les `AllowedIPs` côté serveur sont des **IP uniques** (`/32`, une par appareil). En site-à-site, elles sont des **sous-réseaux entiers** (`/24` et plus). Même outil, même syntaxe, périmètre différent. En entreprise, ce montage se fait souvent avec **IPSec** sur les routeurs — le concept est identique, seul l'outil change.

### 3.9 OpenVPN : le vétéran (aperçu)

WireGuard utilise des clés simples ; OpenVPN s'appuie sur une **PKI** (certificats signés par une CA — Leçon 4). Le principe :

```bash
sudo apt install -y openvpn easy-rsa
# easy-rsa = l'outil qui génère la PKI (CA, certificats serveur/client)

make-cadir ~/pki-openvpn && cd ~/pki-openvpn
# crée un dossier de travail pour la PKI

./easyrsa init-pki && ./easyrsa build-ca nopass
# init-pki → initialise ; build-ca → crée l'autorité (comme la CA auto-signée, Leçon 4)

./easyrsa build-server-full serveur nopass && ./easyrsa build-client-full client1 nopass
# certificats signés pour le serveur et un client (nopass = sans phrase de passe)

./easyrsa gen-dh
# paramètres d'échange de clés (Diffie-Hellman, ancêtre du mécanisme de TLS)
```

Ensuite, un fichier `/etc/openvpn/server/server.conf` (gabarit dans `04-commandes-references.md`) puis `sudo openvpn --config client.ovpn` côté client. WireGuard fait la même chose en ~15 lignes et sans CA : c'est pourquoi il est devenu le choix par défaut — mais OpenVPN reste indispensable quand l'environnement (pare-feu, routeur d'entreprise) ne connaît que lui.

> 🧭 **Et les tunnels SSH (`-L`, `-R`, `-D`), le reverse SSH tunnel et Cloudflare Tunnel ?** Ils méritent leur propre leçon, car ils répondent à un besoin **différent** du VPN : pas « relier un réseau », mais « atteindre **un** port ou exposer **un** service, le temps d'une session ». C'est toute la **Leçon 5b** — ton prochain arrêt.

---

## 4. Bonnes pratiques modernes (2025-2026)

- **WireGuard par défaut** pour tes usages perso ; OpenVPN si l'environnement l'exige.
- **Un appareil = une clé / un `[Peer]`** : pouvoir révoquer UNE clé sans couper les autres (moindre privilège, Bloc 2).
- **Split tunnel par défaut** ; full tunnel seulement sur un besoin réel (Wi-Fi public non fiable).
- **Permissions 600** sur les fichiers de clés — et **jamais dans Git** (Leçon 7 : secrets).
- **Pare-feu minimal** : seul le port UDP du VPN ouvert (Leçon 3) ; SSH idéalement **uniquement via le tunnel**.
- **Rotation immédiate** d'une clé compromise : supprimer le `[Peer]`, régénérer, redéployer.
- **Vérifie la fuite DNS** après branchement : en split tunnel, le DNS peut sortir en clair — teste avec `dig +short whoami.cloudflare @1.1.1.1` (la réponse « whoami » est l'IP vue par le résolveur).

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi c'est dangereux/inefficace | ✅ Version correcte |
|---|---|---|
| `AllowedIPs = 0.0.0.0/0` « pour être sûr » | tout ton trafic dépend du serveur ; lenteurs inutiles au quotidien | `AllowedIPs = 10.66.66.0/24` (split) sauf besoin réel |
| La même clé sur PC et téléphone | impossible de révoquer un appareil sans couper les autres | un `[Peer]`/une clé par appareil |
| Ouvrir SSH publiquement « en plus » du VPN | double la surface d'attaque (rappel Leçon 3) | SSH limité à l'interface du tunnel (`wg0`) |
| Clés commitées dans Git | vol de l'accès au réseau ; l'historique les garde pour toujours (Leçon 7) | fichiers `600` hors dépôt + rotation |
| Oublier `PersistentKeepalive` derrière une box | le tunnel « meurt » après quelques minutes d'inactivité | `PersistentKeepalive = 25` côté client |
| Aucun test après branchement | « ça marche » non vérifié = faux sentiment de sécurité | `wg show` + `ping` + `curl https://ifconfig.me` |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`**, la correction dans **`03-correction.md`**, l'aide-mémoire dans **`04-commandes-references.md`**.

**Énoncé court** : sur un VPS de test (ou deux VMs locales), monte un serveur WireGuard, connecte ton ordinateur, vérifie le tunnel (`wg show`, `ping`), ouvre SSH **seulement** via le tunnel, prouve la différence split/full tunnel avec `curl https://ifconfig.me`, et génère le QR code pour ton téléphone (optionnel).

---

## 7. Correction détaillée de l'exercice

> La correction complète pas-à-pas est dans **`03-correction.md`**. Le raisonnement :
> - clés générées avec permissions `600`, **une paire par appareil** (révocable individuellement) ;
> - pare-feu ouvert **uniquement** sur `51820/udp` — le port SSH public est ensuite fermé ;
> - validation par **observations** (`wg show`, ping sur l'IP de tunnel, IP de sortie), jamais par supposition.

---

## 8. Checklist de validation

- [ ] J'explique ce qu'est un VPN et je cite 3 usages quotidiens.
- [ ] Je **distingue VPN d'accès distant et VPN site-à-site**, et je sais que l'un relie une machine à un réseau, l'autre deux réseaux entiers.
- [ ] Je génère une paire de clés WireGuard et je sais qui possède quoi (privée = secrète, publique = partageable).
- [ ] J'écris une config serveur et une config client avec des `AllowedIPs` cohérents.
- [ ] Je démarre le tunnel, le rends persistant (`systemctl enable`) et le vérifie (`wg show`, `ping`).
- [ ] Je choisis split vs full tunnel selon le besoin et j'en connais la conséquence.
- [ ] J'explique la différence accès distant / site-à-site et je sais quels `AllowedIPs` écrire pour chacun.
- [ ] Je sais quand préférer un tunnel SSH (Leçon 5b) à un VPN — et quand c'est l'inverse.
- [ ] Je situe WireGuard vs OpenVPN (quand utiliser l'un ou l'autre).
- [ ] Aucune clé dans Git, permissions 600, pare-feu n'exposant que le port VPN.

---

🧭 **Pont vers la suite (Leçon 5b)** — Tu sais maintenant **relier durablement une machine ou un réseau entier** en privé (VPN). Mais il y a l'autre besoin, plus **ponctuel** : joindre **un seul port** (lire une base 5 minutes), montrer une app locale à un collègue, ou exposer un service **sans ouvrir de port** (NAT, rappel §2.2). Pour cela, pas besoin d'un VPN : des **tunnels** plus légers — SSH (`-L`, `-R`, `-D`), reverse SSH tunnel, Cloudflare Tunnel. C'est toute la Leçon 5b.

---

*Prochaine étape :* Leçon 5b — **Tunnels et exposition de services (SSH, reverse, Cloudflare Tunnel)** dans `05b-Tunnels-et-exposition-services/`.

