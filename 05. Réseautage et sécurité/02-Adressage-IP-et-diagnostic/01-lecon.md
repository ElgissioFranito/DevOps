# Leçon 2 — Adressage IP et diagnostic réseau

> **Bloc 5 · Réseautage et sécurité** — Leçon 2 sur 8
> 🧭 **Pont depuis la Leçon 1** : tu sais de quoi « se parlent » les machines (protocoles). Mais pour viser une machine précise, il faut une **adresse**. Cette leçon te donne les bases de l'**adressage IP** (IPv4, subnet, gateway, CIDR, IP privée/publique) et les outils **diagnostic** (`ping`, `traceroute`, `ip`) pour suivre un paquet jusqu'à sa cible.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Expliquer** ce qu'est une adresse IP, un subnet, un masque et une passerelle (gateway).
2. **Lire et écrire** une adresse en notation **CIDR** (ex. `192.168.1.10/24`) et en déduire la plage d'adresses.
3. **Distinguer** IP privée (LAN) et IP publique (Internet), et nommer les plages privées (10.x, 172.16-31.x, 192.168.x).
4. **Diagnostiquer** une panne avec `ping`, `ping6`, `traceroute`/`tracepath`, `ip addr`, `ip route`, et la chaîne complète vue au Bloc 1.
5. **Savoir** (sans creuser) définir **IPv4 vs IPv6** et **IPSec**.

---

## 2. Explication simple

### 2.1 Le « pourquoi » : pourquoi une adresse ?

Sans adresse, impossible de retrouver une machine dans le réseau — comme une maison sans numéro dans une ville. Chaque machine connectée a une **adresse IP** unique sur son réseau, qui sert de « numéro de rue » pour router les paquets.

> 💡 **Analogie** : l'adresse IP, c'est le **numéro de maison** ; la **passerelle/routeur**, c'est la **poste du quartier** qui sait quel trafic envoyer vers le monde extérieur ; le **DNS** c'est l'annuaire de noms.

### 2.2 IPv4 et IPv6 (le « quoi »)

- **IPv4** : adresses de 32 bits, écrites en 4 octets : `192.168.1.10`. C'est encore la norme majoritaire.
- **IPv6** : adresses de 128 bits en hexa, ex. `2001:db8::1`. Espace bien plus grand.

> 🔵 **À mentionner, définir, ne pas creuser — IPv6** : futur de l'adressage, mais la plupart des infra actuelles fonctionnent encore en IPv4. Bon à savoir reconnaître, pas urgent à maîtriser.

### 2.3 Subnet, masque et CIDR : le « comment »

Un **subnet** (sous-réseau) est un **groupe d'adresses** qui se « voient » directement entre elles, sans passer par un routeur. On le délimite par un **masque**, écrit en **CIDR** (Classless Inter-Domain Routing) :

```
192.168.1.0/24      → les 256 adresses 192.168.1.0 → 192.168.1.255
                       masque 255.255.255.0, la "maison du 192.168.1"
192.10.0.0/16       → plage beaucoup plus grande 192.10.0.0 → 192.10.255.255
```

Explication du `/24` : les **24 premiers bits** sont fixes (le réseau) ; les 8 derniers bits (2⁸ = 256) décrivent les machines du subnet.

> 💡 **Repère** : `/24` = maison dans le quartier 192.168.1 ; `/16` = toute la « ville » 192.10.x ; `/8` = tout le réseau 10.x. Plus le dénominateur est petit, plus le réseau est grand.

**La passerelle (gateway)** = l'adresse du routeur qui permet de sortir du subnet vers un autre réseau (ex. Internet). Classiquement l'adresse basse : `192.168.1.1`.

### IP privée / publique

| Type | Plage typique | Usage |
|------|---------------|-------|
| **Privée** | `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16` | Réseaux internes du LAN, non routables vers Internet directement |
| **Publique** | tout le reste attribué par un FAI | Machines accessibles depuis Internet |

Avec NAT/PAT, une machine à IP privée va sortir vers Internet grâce à l'IP **publique** de son routeur (voir mini-glossaire).

---

## 📖 Mini-glossaire (à consulter avant les exemples)

> Définitions d'une ligne pour ne jamais être perdu(e).

- **Bit** : plus petite unité d'information (0 ou 1). **Octet** = 8 bits.
- **IPv4** : format d'adresse de 32 bits, écrit en 4 nombres (ex. `192.168.1.10`).
- **IPv6** : format d'adresse de 128 bits, plus grand, écrit en hexadécimal (ex. `2001:db8::1`). À savoir reconnaître, pas à maîtriser.
- **IPSec** : suite de protocoles pour chiffrer/authentifier le trafic IP (ex. montage de VPN site-à-site). Nom à connaître, sans creuser.
- **Subnet / sous-réseau** : groupe d'adresses qui se voient entre elles sans passer par un routeur.
- **Masque** : nombre qui indique la taille du réseau (souvent écrit en CIDR `/24`).
- **CIDR** (Classless Inter-Domain Routing) : notation `adresse/nombre` pour dire la **plage du réseau**, ex. `192.168.1.0/24`.
- **Gateway / passerelle** : adresse du routeur qui permet de sortir du réseau local.
- **LAN** (Local Area Network) : ton réseau local (Wifi/maison/bureau).
- **FAI** (fournisseur d'accès Internet) : l'entreprise qui te connecte (Orange, Free, SFR…).
- **NAT** (Network Address Translation) : mécanisme du routeur qui traduit ton IP privée en IP publique pour sortir vers Internet.
- **ICMP** : protocole de diagnostic léger, utilisé par `ping` (il ne transporte pas de données applicatives).
- **Routeur** : machine qui aiguille les paquets d'un réseau à un autre.

---

### 🧪 À faire maintenant (5 min) — connaître et tester ton propre réseau

> Objectif : partir de TA machine et remonter étape par étape, comme dans un vrai diagnostic. Exécute :

```bash
ip addr show          # ton IP + masque (ex. 192.168.1.42/24)
ip route show         # ta gateway (ex. default via 192.168.1.1)
ping -c 4 8.8.8.8     # test Internet (4 paquets puis arrêt)
traceroute -m 5 example.com   # les routeurs traversés (5 sauts max)
```

**Ce que tu dois observer / écrire dans ta tête** :
- Ton `192.168.x.x/24` = ton adresse **privée** + la taille `/24` (256 adresses).
- `default via ...` = ta gateway (le routeur qui te fait sortir).
- `ping 8.8.8.8` répond → ta **sortie Internet** fonctionne.
- `traceroute` liste les **sauts** (routeurs) pour atteindre `example.com` — des `*` = routeur qui filtre, pas une panne forcément.

**Astuce** : si un problème réseau survient, refais ce parcours en notant la **première étape qui échoue** : c'est là que se situe la panne (cascade vue au Bloc 01).

---

## 📖 Vocabulaire / Abréviations

| Terme | Définition (une ligne) |
|---|---|
| **IPv4 / IPv6** | format d'adresse 32 bits (`192.168.1.10`) / 128 bits (hexadécimal) |
| **CIDR** | notation d'une plage d'adresses (`/24` = 256 adresses) |
| **Masque de sous-réseau** | l'ancienne écriture du CIDR (`255.255.255.0` = `/24`) |
| **Subnet (sous-réseau)** | groupe d'adresses qui se voient sans passer par un routeur |
| **Gateway (passerelle)** | le routeur par lequel sort le trafic vers d'autres réseaux |
| **IP privée / publique** | adresse interne au réseau / joignable depuis tout Internet |
| **Loopback (127.0.0.1, localhost)** | « moi-même » : adresse pour se joindre soi-même |
| **ICMP** | protocole de diagnostic utilisé par `ping` et `traceroute` |

---

## 3. Exemples concrets

### 3.1 Voir ton adressage

```bash
ip addr show              # IP, masque, interfaces (comme ifconfig)
ip route show             # gateway par défaut
hostname -I               # IP privées de la machine
```

### 3.2 Les deux commandes icône du diagnostic : `ping` et `traceroute`

```bash
# ICMP : la machine répond-elle ?
ping -c 4 8.8.8.8            # 4 échos (ctrl+C sinon infini)
ping -c 4 exemple.com

# Tracer : par quels routeurs passe la requête ?
traceroute exemple.com        # parfois il faut `traceroute` installé
tracepath exemple.com         # alternative sans privilège
```

> 💡 **Chronologie de la réponse**
> - `ping` : dit : « la cible répond » et mesure la latence.
> - `traceroute` : liste les routeurs intermédiaires (chaque ligne = un saut).

### 3.3 Un exemple simple de plage CIDR

```bash
# Le réseau 192.168.1.0/24 regroupe les adresses 192.168.1.0 à .255
# L'adresse 192.168.1.99 est dedans ; 192.168.2.1 n'en fait pas partie (réseau voisin).
```
### 3.4 Chercher ton IP publique

```bash
# Via un service en ligne (exemple)
curl -4 https://api.ipify.org
```
---

## 4. Bonnes pratiques modernes (2025-2026)

- **Toujours utiliser le CIDR, pas les vieux masques** : `ip addr` affiche `192.168.1.10/24`, pas `255.255.255.0` seul.
- **Privilégier `ip` plutôt que `ifconfig`** (obsolète sur de nombreuses distros récentes).
- **Diagnostiquer par étapes** : `ping` local (127.0.0.1) → IP du LAN → gateway → IP publique → DNS → service. C'est la cascade du Bloc 1.
- **Ne jamais lancer `ping`/`traceroute` en boucle infinie en production** sans `-c`.
- La **plage `/24`** reste le standard de base pour les petits LANs ; en cloud, on découpe souvent des subnets plus fins (`/26`, `/27`) par zone de sécurité.

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi c'est dangereux/inefficace | ✅ Version correcte |
|----------------|-------------------------------------|---------------------|
| `ping` infini sans `-c` | Occupe le terminal et fausse ton diagnostic. | `ping -c 4 host`. |
| Oublier la passerelle / le firewall | On est « connecté au LAN » mais pas d'Internet : on cherche à mauvais endroit. | Vérifier `ip route`, puis firewall (Leçon 3). |
| Confondre IP **privée** et **publique** | `ifconfig` donne une IP 192.168.1.X mais le serveur n'est pas joignable depuis Internet sans NAT/IP publique. | Savoir distinguer les plages, et quel port est exposé. |
| Se fier à `traceroute` bloqué pour dire « coupure » | Beaucoup de routeurs filtrent ICMP → des `*` ne signifient pas forcément une panne. | Interpréter à conjoint-avec `ping` final et `curl`. |
| Utiliser du `nmap` sans autorisation sur un réseau d'autrui | Illégal et intrusif | Ne scanner que chez soi/la machine de test (voir Leçon 3). |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`**, la correction dans **`03-correction.md`**.

**Énoncé court** : dans `notes-exercice-02.md`, recueille : ton adresse et son masque (`ip addr`, `ip route`), teste `ping -c 4` vers ta gateway et vers 8.8.8.8, lance `traceroute` vers `example.com` (5 sauts max), note ton IP publique via `curl https://api.ipify.org`, et détermine si une adresse donnée (ex. `10.0.0.1/8` vs `192.168.1.10/24`) est dans la plage souhaitée.

---

## 7. Correction détaillée de l'exercice

> La correction complète pas-à-pas est dans **`03-correction.md`**. Essentiel du raisonnement :
- on **part de soi** : `ip addr` → je connais mon masque et ma gateway ;
- on **monte en haut** : local → gateway  → Internet, défini où la rupture se situe ;
- on **croise** ping + traceroute + IP privée/publique pour confirmer le diagnostic.

---

## 8. Checklist de validation

- [ ] J'explique IP, subnet, CIDR et passerelle.
- [ ] J'écris une adresse en `/24` et je sais dire si une IP est dans la plage.
- [ ] Je distingue IP privée et publique et je cite les plages privées.
- [ ] Je diagnostique avec `ip addr`, `ip route`, `ping -c`, `traceroute`.
- [ ] Je sais chercher mon IP publique.
- [ ] (survol) Je définis IPv4 vs IPv6 et IPSec.

---

🧭 **Pont vers la suite** — Diagnostiquer suffit pour localiser un saut. Mais pour **bloquer ou autoriser** un flux (la découverte que ton gateway 168 n'était pas la seule), il faut un **pare-feu**. C'est la Leçon 3.

---

*Prochaine étape :* Leçon 3 — **Pare-feu et contrôle des flux** dans `03-Pare-feu`.