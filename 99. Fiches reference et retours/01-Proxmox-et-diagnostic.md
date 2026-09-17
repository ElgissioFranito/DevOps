# Fiche de référence 01 — Serveur Proxmox et diagnostic réseau

> 🧭 **Pont depuis les leçons** — Cette fiche réutilise ce que tu connais déjà :
> - **Bloc 2 (Linux)** : fichiers, utilisateurs, services (`systemctl`/`journalctl`), SSH, umask ;
> - **Bloc 5 (Réseau)** : TCP/UDP, adressage IP et CIDR (Leçon 2), pare-feu (Leçon 3), SSH par clé (Leçon 5 du bloc 2), VPN (Leçon 5a), reverse proxy (Leçon 6).
> **À quoi sert cette fiche ?** C'est une **référence de dépannage**, pas une leçon : reviens-y quand ton PC n'arrive pas à joindre un serveur Proxmox (ou une VM qui tourne dessus). Chaque section répond à une question « où suis-je bloqué ? ».

---

## 1. Le scénario de départ (pour situer le contexte)

**Situation réelle** : un PC veut joindre un **serveur Proxmox** (et les machines virtuelles qui tournent dessus) sur le réseau local, et ça ne marche pas. Ce document regroupe : le vocabulaire pour comprendre, la méthode pour diagnostiquer, les commandes pour agir.

**Proxmox VE** = un système complet pour transformer un serveur physique en plateforme de machines virtuelles. Concrètement : Debian (une distribution Linux) + **KVM** (le composant du noyau Linux qui fait tourner des VM) + **LXC** (conteneurs système) + une **interface web** (port 8006) + la gestion du stockage, du réseau, des sauvegardes.

## 2. 📖 Vocabulaire / Abréviations

| Terme | Définition (1 ligne) |
|---|---|
| **Hyperviseur type 1** | logiciel qui s'installe **directement sur le matériel** (Proxmox, ESXi, Hyper-V) |
| **Hyperviseur type 2** | logiciel qui s'installe **dans un OS déjà présent** (VirtualBox, VMware Workstation) |
| **KVM** | composant du noyau Linux qui fait tourner des VM (Kernel-based Virtual Machine) |
| **VM** | machine virtuelle : un « ordinateur » simulé, avec son propre OS complet |
| **LXC** | conteneur **système** : un Linux quasi complet, mais partageant le noyau de l'hôte |
| **Conteneur applicatif** | un seul programme emballé avec ses dépendances (c'est **Docker**, vu au Bloc 9) |
| **Hôte (host)** | le serveur physique qui fait tourner Proxmox |
| **Invité (guest / CT)** | une VM ou un conteneur créé à l'intérieur de Proxmox |
| **Bridge (vmbr0)** | « commutateur virtuel » : relie les VM au réseau physique (voir §7) |
| **NIC** | carte réseau (Network Interface Card) |
| **MAC** | adresse physique gravée d'une carte réseau |
| **ARP** | protocole qui traduit une IP en adresse MAC (« qui a cette IP ? ») |
| **VLAN** | segmentation logique d'un réseau physique en plusieurs réseaux isolés |
| **VLAN tag** | numéro (1-4094) inséré dans les paquets pour indiquer leur VLAN |
| **Bonding** | union de plusieurs cartes réseau en une seule (redondance ou débit) |
| **DHCP** | protocole qui distribue automatiquement les configurations IP |
| **Métrique (route)** | priorité d'une route : plus petite valeur = plus prioritaire |
| **`NO-CARRIER`** | « aucun signal électrique » : problème physique (câble/port) |
| **Snapshot** | instantané de l'état d'une VM — **≠ sauvegarde** (voir §9, cas 10) |
| **VMID** | numéro identifiant une VM/CT dans Proxmox (ex. 100, 101) |
| **PBS** | Proxmox Backup Server : l'outil de sauvegarde dédié de Proxmox |
| **DMZ** | zone isolée entre un réseau interne et Internet |

## 4. La méthode : diagnostiquer de bas en haut

> 🧭 C'est la **cascade** que tu connais du Bloc 5 (Leçon 1 : DNS → IP → port → service), appliquée au plus près du matériel. **Ne jamais sauter de couche** : chaque couche ne peut marcher que si celle du dessous marche.

| Couche | Question à se poser | Commandes |
|---|---|---|
| **1 – Physique** | Le câble est-il branché, le port vivant ? | yeux (LEDs), `ethtool nic0`, `ip -br link show` |
| **2 – Liaison** | L'interface est-elle UP ? Le bridge a-t-il un port actif ? | `ip -br link show`, `bridge link show` |
| **3 – Réseau** | IP correcte ? Masque ? Gateway ? Routes ? | `ip a`, `ip route` |
| **4 – Transport** | Le port écoute-t-il ? Le pare-feu bloque-t-il ? | `ss -tlnp`, `iptables -L`, `nft list ruleset`, `ufw status` |
| **7 – Application** | Le service tourne-t-il et répond-il ? | `systemctl status <svc>`, `journalctl -u <svc>`, `curl` |

**Règle d'or : ~90 % des problèmes sont en couche 1 ou 3** (câble, IP, masque, gateway). Le réflexe humain naturel (« c'est l'application qui bug ! ») est presque toujours le mauvais réflexe.

---

## 5. Adressage IP : les fondamentaux appliqués

> 🧭 Renvoi direct au **Bloc 5, Leçon 2** : IP, CIDR, sous-réseau, gateway. Cette section en fait un plan de diagnostic.

- Une IP seule ne veut rien dire : il faut **IP + masque**. `10.10.10.36/24` → réseau `10.10.10.0/24` (254 adresses utilisables).
- **Deux machines dans des sous-réseaux différents ne communiquent pas sans routeur** — même si elles sont branchées sur le même switch (le switch travaille en couche 2/MAC ; le routage est couche 3/IP). *C'est LA cause classique du « même câblage, ça ne marche pas ».*
- **Gateway** = le routeur de sortie du réseau : tout ce qui n'est pas local lui est envoyé (`default via 10.10.10.1 dev enp43s0`).

```bash
ip route                 # la table de routage : où envoyer quoi
# default via 10.10.10.1 dev enp43s0     ← tout ce qui n'est pas local → la gateway
# 10.10.10.0/24 dev enp43s0 proto kernel scope link
#   scope link : réseau joignable DIRECTEMENT (sans routeur)
#   proto kernel : route créée automatiquement par le noyau
#   proto dhcp : route apprise via DHCP ; metric : priorité (plus petit = gagne)
```

---

## 6. Traduire les messages d'erreur

> 🧭 Renvoi au **Bloc 2, Leçon 4** (`ping`, `ss`) : chaque message pointe vers une couche.

| Message vu | Traduction en clair | Cause typique | Couche |
|---|---|---|---|
| `Destination Host Unreachable` | « Impossible d'envoyer sur le réseau local » | ARP sans réponse, interface down | 2-3 |
| `Request timed out` | « Parti, jamais de réponse » | Pare-feu, service arrêté | 4-7 |
| `NO-CARRIER` | « Aucun signal électrique » | Câble débranché/défectueux, mauvais port | 1 |
| `linkdown` | « L'interface est down pour le noyau » | Bridge sans port actif | 2 |
| `DOWN` | « Interface désactivée » (volontairement ou faute de lien) | `ip link set down`, ou `NO-CARRIER` | 1-2 |

---

## 7. Le bridge : le « switch virtuel » de Proxmox

**Analogie** : le bridge est une **multiprise réseau virtuelle**. Le câble physique (nic0) y est branché, et les VM s'y branchent aussi : elles se retrouvent « sur le même switch » que le réseau local.

Config type de `/etc/network/interfaces` (Debian/Proxmox), **commentée ligne par ligne** :

```text
auto lo                      # l'interface de bouclage locale (127.0.0.1), toujours
iface lo inet loopback

auto nic0                    # la carte réseau PHYSIQUE
iface nic0 inet manual       # « manual » = pas d'IP ici : elle n'est qu'un port du bridge

auto vmbr0                   # le bridge (le switch virtuel)
iface vmbr0 inet static      # c'est LUI qui porte l'IP du serveur
    address 10.10.10.214/24  # l'IP de l'hôte Proxmox
    gateway 10.10.10.1       # la sortie vers le reste du réseau
    bridge-ports nic0        # ce que le bridge connecte : la carte physique
    bridge-stp off           # protocole anti-boucle désactivé (lab simple)
    bridge-fd 0              # délai de « forwarding » à 0
```

**Les 4 points critiques** :
1. Interface physique en `manual` (sans IP) ;
2. `bridge-ports` = nom **exact** de l'interface (vérifié avec `ip -br link show`) ;
3. L'IP va **sur le bridge**, pas sur l'interface physique ;
4. Un bridge sans port actif = interface `DOWN` (message `linkdown`).

**Identifier quel port physique correspond à quel nom** (serveurs à plusieurs NICs) :

| Méthode | Fiabilité | Commande |
|---|---|---|
| Faire **clignoter la LED** du port | ⭐⭐⭐⭐⭐ | `ethtool -p nic0 10` (10 s) |
| Débrancher/rebrancher un câble en observant | ⭐⭐⭐⭐⭐ | `watch -n 1 'ip -br link show'` |
| Documentation constructeur | ⭐⭐⭐⭐⭐ | manuel du serveur |
| Menu BIOS/UEFI | ⭐⭐⭐⭐ | au redémarrage |
| Bus PCI | ⭐⭐⭐ | `ls -l /sys/class/net/` |
| Logs du noyau | ⭐⭐⭐ | `dmesg | grep -i eth` |

## 8. Boîte à commandes (organisée par question)

> 🧭 Plutôt qu'une liste brute : chaque bloc répond à une **question**. Les bases (`ip`, `ss`, `systemctl`, `journalctl`) sont celles des Blocs 2 et 5.

**« Quelles interfaces, quelles IP ? »**
```bash
ip a                        # interfaces et IP (Bloc 5, Leçon 2)
ip -br link show            # état court : UP / DOWN / NO-CARRIER par interface
ip neigh                    # cache ARP : qui a répondu « c'est moi » à quelle MAC
sudo ip neigh flush all     # vide le cache (après un changement de machine/câble)
```

**« Ma carte réseau est-elle physiquement vivante ? »**
```bash
ethtool nic0                # « Link detected: yes/no »
ethtool -p nic0 10          # fait clignoter la LED du port 10 secondes
ethtool -i nic0             # driver et emplacement (bus PCI)
lspci | grep -i ethernet    # lister les cartes réseau installées
dmesg | grep -i eth         # ce que le noyau a détecté au démarrage
```

**« Quels ports écoutent ? Qui bloque ? »**
```bash
ss -tlnp                    # ports TCP en écoute + processus (Bloc 2, Leçon 4)
iptables -L                 # règles du pare-feu (moteur historique)
nft list ruleset            # règles du pare-feu (moteur moderne)
ufw status                  # pare-feu UFW (Bloc 5, Leçon 3)
```

**« Configurer et appliquer le réseau (Debian/Proxmox) »**
```bash
cat /etc/network/interfaces     # le fichier de config réseau de Debian
ip link set nic0 up             # activer une interface
ifreload -a                     # réappliquer la config (commande Proxmox)
systemctl restart networking    # redémarrer le service réseau
```

**« Côté Proxmox : que tourne-t-il ? »**
```bash
pveversion                  # version de Proxmox
qm list                     # les VM
pct list                    # les conteneurs LXC
pvesm status                # les stockages
pvecm status                # l'état du cluster
systemctl status pveproxy   # le serveur web de l'interface (port 8006)
```

**« L'état des disques »** (rappel Bloc 2, Leçon 4)
```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT,MODEL   # arbre des disques
df -h                       # espace occupé/libre
cat /sys/block/sdX/queue/rotational   # 0 = SSD, 1 = HDD
smartctl -H /dev/sdX        # santé du disque (paquet smartmontools)
pvs / vgs / lvs             # LVM : volumes physiques / groupes / logiques (encart L4)
zpool status && zfs list    # si stockage ZFS
```

**« SSH dans une VM Debian »** (rappel Bloc 2, Leçon 5)
```bash
sudo apt install -y openssh-server   # installe le serveur SSH
sudo systemctl enable --now ssh      # active au boot + démarre maintenant
sudo systemctl status ssh            # vérifie qu'il tourne
sudo nano /etc/ssh/sshd_config       # config ; PermitRootLogin yes : lab UNIQUEMENT
sudo systemctl restart ssh           # recharger la config
```

## 9. Recettes pas à pas

### Cas 1 — Connecter une VM au réseau local (le plus fréquent)
1. Dans Proxmox, crée la VM avec sa carte réseau sur `vmbr0`.
2. Dans la VM, configure une IP du **même sous-réseau** que le LAN (`10.10.10.0/24`), par DHCP ou statique.
3. Depuis le PC : `ping <IP_VM>`.
4. En cas d'échec : vérifier **l'hôte** d'abord (`bridge link show`, `ip -br link show`), puis la config IP de la VM. Cascade, jamais l'inverse.

### Cas 2 — Un réseau interne isolé pour les VM (lab)
```text
auto vmbr1
iface vmbr1 inet static
    address 192.168.100.1/24
    bridge-ports none        # « none » = switch virtuel NON relié au câble physique
    bridge-stp off
    bridge-fd 0
```
Les VM branchées sur `vmbr1` communiquent **entre elles seulement** (idéal : lab Kubernetes, DMZ virtuelle). Pour qu'elles sortent sur Internet, l'hôte doit faire du **NAT** (voir cas 6).

### Cas 3 — VLAN sur un bridge (avancé — à définir)
```text
auto vmbr0.10
iface vmbr0.10 inet static
    address 10.10.10.214/24
    vlan-raw-device vmbr0    # « sous-interface » du bridge, taguée VLAN 10
```
> 🔵 **VLAN** = segmentation logique d'un réseau : on regroupe des machines « comme si » elles étaient sur un câble séparé. Utile en infra segmentée — à approfondir seulement quand l'entreprise l'utilise.

### Cas 4 — Un seul bridge pour tous les VLANs (avancé — à définir)
```text
auto vmbr0
iface vmbr0 inet manual
    bridge-ports nic0
    bridge-vlan-aware yes    # le bridge comprend les tags VLAN
    bridge-vids 2-4094       # plages de VLANs autorisées à le traverser
```
Le tag VLAN se choisit alors **dans la config de chaque VM** : un seul bridge gère tout.

### Cas 5 — Bonding : deux cartes en une (avancé — à définir)
```text
auto bond0
iface bond0 inet manual
    bond-slaves nic0 nic1    # les deux cartes font « équipe »
    bond-mode active-backup  # une active, l'autre en secours
    bond-miimon 100          # vérification d'état toutes les 100 ms

auto vmbr0
iface vmbr0 inet static
    address 10.10.10.214/24
    gateway 10.10.10.1
    bridge-ports bond0       # le bridge s'appuie sur le bond
```
> 🔵 **Bonding** = agréger plusieurs cartes réseau : **redondance** (un câble peut mourir sans coupure) ou **débit**. Modes courants : `active-backup` (secours), `802.3ad`/LACP (négociation avec le switch), `balance-rr` (alternance).

### Cas 6 — Une VM ping la gateway mais pas 8.8.8.8 (sans Internet)
1. `ip route` **dans la VM** : IP et gateway correctes ?
2. `bridge link show` **sur l'hôte** : le bridge a-t-il son port actif ?
3. **Sur l'hôte**, le « routage » est-il activé ? `sysctl net.ipv4.ip_forward` → si `0`, activer :
   ```bash
   echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
   sudo sysctl -p              # réapplique immédiatement
   ```
4. Si le réseau VM est **interne** (cas 2), il faut le **NAT** : `sudo iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -o vmbr0 -j MASQUERADE` (MASQUERADE = « traduis leur IP privée en celle de vmbr0 en sortie » — le même principe que le NAT du Bloc 5, Leçon 2).

### Cas 7 — SSH par clé (bonne pratique — rappel Bloc 2, Leçon 5)
```bash
ssh-keygen -t ed25519 -C "elgissio@pc"     # génère la paire de clés
ssh-copy-id user@10.10.10.214              # installe la clé PUBLIQUE sur le serveur
ssh user@10.10.10.214                      # connexion sans mot de passe
```
Puis, dans `/etc/ssh/sshd_config` du serveur : `PasswordAuthentication no` + `sudo systemctl restart ssh`. ⚠️ Toujours tester la connexion par clé **avant** de désactiver le mot de passe (règle d'or du Bloc 2).

### Cas 8 — Joindre Proxmox depuis l'extérieur : par VPN, jamais en direct
**Ne jamais exposer le port 8006 sur Internet** (faille critique — rappel Bloc 5 : moindre exposition). Options : **WireGuard** (Leçon 5a), **OpenVPN**, ou **Tailscale** (Leçon 5a, encart mesh ; voir la fiche 02).
```bash
curl -fsSL https://tailscale.com/install.sh | sh   # installe Tailscale
tailscale up                                        # connecte la machine au réseau VPN
# puis : https://<IP-Tailscale-100.x.x.x>:8006
```

### Cas 9 — Cloner une VM pour un lab
```bash
qm clone 100 101 --name debian-test --full  # 101 = copie complète de la VM 100
```
⚠️ Après clonage, **changer l'IP et le hostname** de la copie (sinon conflit : deux machines se déclarent avec la même identité).

### Cas 10 — Snapshot avant un test risqué
```bash
qm snapshot 100 avant-maj        # fige l'état (point de restauration)
# ... test ...
qm rollback 100 avant-maj        # revient à l'état figé
```
⚠️ **Snapshot ≠ sauvegarde** : le snapshot vit sur le même disque. Si le disque meurt, snapshots et VM meurent ensemble. Les vraies sauvegardes passent par PBS (un autre support).

---

## 10. Erreurs classiques et leur correction

| Erreur | Conséquence | Solution |
|---|---|---|
| Câble branché sur le mauvais port | `NO-CARRIER` | `ethtool -p nic0 10` pour identifier |
| `bridge-ports` mal orthographié | Bridge `DOWN` | `ip -br link show` pour vérifier le nom |
| VM dans un autre sous-réseau | Pas de communication | Même sous-réseau (ou routeur) |
| Gateway erronée | Pas d'Internet | `ip route` |
| Confondre hôte et VM | On configure la mauvaise machine | `hostname` + `ip a` AVANT toute commande |
| Cache ARP obsolète | Erreurs fantômes après un changement | `ip neigh flush all` |
| Commencer par l'application | Heures perdues | Toujours commencer couche 1 |
| Croire que snapshot = sauvegarde | Perte totale si le disque meurt | Vraies sauvegardes (PBS) |
| Port 8006 exposé sur Internet | Faille critique | VPN uniquement |

---

## 11. Les réflexes à mémoriser

1. **Diagnostiquer de bas en haut** : physique → liaison → réseau → transport → application. Ne jamais sauter de couche.
2. `hostname` + `ip a` : savoir **où** on est (hôte ou VM) avant d'agir.
3. `ip -br link show` puis `ip route` : les deux premières vérifications quasi automatiques.
4. `ping` la **gateway** : premier test « au-delà de soi ».
5. `ss -tlnp` + pare-feu : la couche 4 vient **après** la couche 3.
6. **Tester après chaque changement**, et **documenter** ce qu'on découvre (tout ce qui n'est pas écrit se redécouvre à chaque incident).
7. **Même switch ≠ même réseau** : c'est le sous-réseau IP qui compte.
8. Snapshots ≠ sauvegardes ; port 8006 jamais exposé sans VPN.

---

🧭 **Fiche liée** : `02-Debian-13-Tailscale-et-Fortinet.md` — le cas concret où le VPN (Tailscale) est lui-même bloqué par un pare-feu d'entreprise.

*Fin de la fiche 01.*




