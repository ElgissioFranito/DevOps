# Leçon 3 — Pare-feu et contrôle des flux

> **Bloc 5 · Réseautage et sécurité** — Leçon 3 sur 8
> 🧭 **Pont depuis la Leçon 2** : tu sais adresser et diagnostiquer. Mais « joignable » n'est pas « autorisé » : entre Internet et ton application, il y a un **gardien** — le **pare-feu** (firewall). Cette leçon te montre son rôle et comment le configurer en pratique avec **UFW** (simple) et comprendre le principe de **iptables/nftables** (solide).

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Expliquer** ce qu'est un pare-feu et la différence entre pare-feu réseau (L3/L4) et pare-feu applicatif (L7).
2. **Comprendre** les notions de port ouvert/filtré, `allow`/`deny`, et l'ordre des règles.
3. **Configurer un pare-feu simple avec UFW** sur Ubuntu : activer, autoriser/bloquer des ports, autoriser par IP.
4. **Lire** l'état du pare-feu et les connexions (`ufw status`, `ss`).
5. **Appliquer le principe de moindre exposition** (n'exposer que l'indispensable).
6. (survol) Lister les points communs avec **iptables/nftables** et les **security groups cloud**.

---

## 2. Explication simple

### 2.1 Le « pourquoi » : un gardien à la porte

Sans pare-feu, chaque port de ta machine est a priori accessible. Avec un pare-feu, tu décides des règles : « accepter le trafic entrant sur le port 443, refuser le reste ». C'est une **liste de contrôle d'accès réseau**.

> 💡 **Analogie** : le pare-feu, c'est le **portier** d'un immeuble. À chaque visiteur (paquet), il consulte sa liste : si le nom (port/IP) est admis → il entre ; sinon → refus. Bonne pratique : **refuser par défaut** tout ce qui n'est pas explicitement autorisé (défaut-deny).

### 2.2 Le « comment » : comment les règles se décident

Un pare-feu évalue chaque paquet selon **sa direction** et **l'état** de connexion :

- **Entrant** (inbound/ingress) : vers ta machine.
- **Sortant** (outbound/egress) : depuis ta machine.

On définit des règles par **port** et/ou par **source IP** :

- `ALLOW 22` (autoriser SSH)
- `ALLOW 443/tcp` (autoriser HTTPS)
- `ALLOW from 192.168.1.0/24` (autoriser un réseau interne)

**État de connexion** : un pare-feu **à état** (stateful) reconnaît les réponses d'une connexion sortante autorisée et les laisse revenir, sans tout revérifier.

### Port ouvert vs filtré
- **Ouvert** : un service écoute et accepte.
- **Filtré** : le port est fermé côté pare-feu (il ne répond pas, « stealth »).

> 🔵 **À mentionner, définir, ne pas creuser — nftables** : successeur moderne d'iptables sur Linux. UFW (que tu vas configurer) est une interface plus simple par-dessus iptables/nftables.

Le **WAF** (Web Application Firewall) est un pare-feu **applicatif** qui inspecte le **contenu** des requêtes HTTP/SQL (ex. bloquer une injection), au niveau L7 — à distinguer du pare-feu réseau (L3/L4).

### 2.3 Le « quand »

| Moment | Action |
|--------|--------|
| Première mise en production d'un serveur cloud | Activer UFW, ouvrir seulement 22/80/443 |
| Après installation d'un service | Vérifier qu'il n'ouvre pas un port qu'on ne veut pas (`ss`) |
| Réseau interne / VLAN | penser aux groupes de sécurité cloud (Bloc 6, 8) |

---

## 📖 Mini-glossaire (à consulter avant les exemples)

> Définitions d'une ligne pour ne jamais être perdu(e).

- **Paquet** : petit bloc de données qui circule sur le réseau (jugé par le pare-feu).
- **ACL** (Access Control List, liste de contrôle d'accès) : la « liste du portier » indiquant qui/quelsports sont autorisés.
- **L3/L4/L7** : repères de couches réseau. L3 = adresses IP, L4 = transport (TCP/UDP), L7 = application (HTTP). « Pare-feu L3/L4 » filtre par IP/port ; « pare-feu L7 » analyse le contenu applicatif.
- **Inbound/ingress vs outbound/egress** : trafic **entrant** (vers ta machine) vs **sortant** (depuis ta machine).
- **Stateful** : un pare-feu « à état » qui mémorise les connexions en cours pour laisser revenir leurs réponses.
- **Port filtré** : port fermé par le pare-feu (ne répond pas). **Port ouvert** : un service écoute et accepte.
- **Stealth / furtif** : comportement où le pare-feu ne répond même pas (invisible, au lieu de dire « refusé »).
- **UFW** (Uncomplicated Firewall) : outil simple pour configurer le pare-feu sous Ubuntu.
- **iptables / nftables** : moteurs de pare-feu « bruts » de Linux, sur lesquels UFW s'appuie en coulisses.
- **WAF** (Web Application Firewall) : pare-feu **applicatif** qui inspecte le contenu des requêtes (ex. bloquer une injection SQL).
- **VM** (machine virtuelle) : un ordinateur simulé dans un fichier (ex. VirtualBox/VMware) — idéale pour tes tests.
- **Gateway** : la passerelle/routeur de sortie (déjà vu Leçon 2).
- **ICMP** : protocole de diagnostic léger (utilisé par `ping`).
- **Security group / NACL** : règles de filtrage réseau configurées dans le cloud (AWS…) — détaillées au Bloc 6/8.
- **Bastion / VPN** : machines ou tunnels d'accès restreint réservés à l'administration.

---

### 🧪 À faire maintenant (5 min) — inspecter le pare-feu sur une machine de test

> ⚠️ **Machine de test uniquement** (VM/WSL) — jamais une machine de production. Objectif : voir ce qu'écoute ta machine et l'état de ton pare-feu.

```bash
ss -tulpn              # quels ports écoute ta machine ?
sudo ufw status verbose  # l'état du pare-feu (activez ? règles ?)
```

**Ce que tu dois observer / écrire dans ta tête** :
- `ss -tulpn` liste les services en écoute et leur port (ex. `:22` = SSH, `:5432` = PostgreSQL, `:80` = HTTP).
- Si `ufw status` dit `inactive`, ton pare-feu **n'est pas activé** : tout le monde peut tenter d'accéder à tes ports ouverts.

**Mini-défi pratique (test uniquement)** :
```bash
sudo ufw default deny incoming   # refuser l'entrant par défaut
sudo ufw allow 22/tcp            # autoriser SSH AVANT d'activer (sinon bloqué !)
sudo ufw allow 80/tcp
sudo ufw enable
sudo ufw status numbered         # tes règles numérotées
```
Re-vérifie : `ss -tulpn` montre toujours le port en écoute, mais le pare-feu décide qui peut **l'atteindre**. C'est la différence « port ouvert » vs « port autorisé ».

---

## 📖 Vocabulaire / Abréviations

| Terme | Définition (une ligne) |
|---|---|
| **Pare-feu (firewall)** | le gardien qui décide quels flux réseau sont autorisés ou bloqués |
| **UFW** | Uncomplicated Firewall : l'interface simple pour gérer le pare-feu Linux |
| **Défaut-deny** | tout refuser par défaut, n'ouvrir que le nécessaire (moindre exposition) |
| **Règle** | une ligne du pare-feu : « autoriser/refuser X sur le port Y » |
| **Port ouvert / filtré** | accessible par tous / silencieux (paquets ignorés) |
| **Security Group** | l'équivalent du pare-feu dans le cloud (Bloc 6) |
| **L3 / L4 / L7** | filtrage par adresse / par port+protocole / par contenu applicatif |
| **Bastion** | petite machine publique servant de porte d'entrée sécurisée |

---

## 3. Exemples concrets

### 3.1 Activer UFW et régler le défaut deny

```bash
sudo ufw default deny incoming   # par défaut, on refuse l'entrant
sudo ufw default allow outgoing  # mais on laisse sortir
sudo ufw allow 22/tcp             # SSH avant tout (sinon on se coupe l'accès !)
sudo ufw allow 80/tcp             # HTTP
sudo ufw allow 443/tcp            # HTTPS
sudo ufw enable                   # ACTIF
sudo ufw status numbered          # liste des règles
```

> ⚠️ **Ordre critique** : **autoriser 22/tcp AVANT** `ufw enable`, sinon tu te coupes l'accès SSH au serveur distant.

### 3.2 Autoriser/bloquer par IP

```bash
sudo ufw allow from 192.168.1.0/24    # réseau interne autorisé
sudo ufw deny 5432/tcp                # bloquer PostgreSQL entrant
sudo ufw allow from 212.38.91.10 to any port 22   # SSH seulement depuis une IP d'admin
```

### 3.3 Vérifier

```bash
sudo ufw status verbose
ss -tulpn    # même info côté écoute réelle
```
---

## 4. Bonnes pratiques modernes (2025-2026)

- **Défaut-deny à l'entrée** : n'accepter que le nécessaire (22/80/443), refuser le reste.
- **Ne jamais exposer de base de données** (5432) en public : la garder derrière le pare-feu, accessible seulement du réseau interne.
- **Autoriser par IP pour l'admin** : SSH limité à une IP de bastion/VPN plutôt qu'à tout l'Internet.
- **Désactiver les ports inutiles** et vérifier régulièrement `ufw status` + `ss -tulpn`.
- En cloud : utiliser les **security groups** et NACL plutôt qu'UFW seul (Bloc 6/8).
- **Ne pas toucher iptables à la main** sauf si on sait ce qu'on fait ; UFW, c'est bien pour 95 % des serveurs simples.

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi c'est dangereux/inefficace | ✅ Version correcte |
|----------------|-------------------------------------|---------------------|
| `ufw enable` avant d'ouvrir 22 | Tu te coupes l'accès SSH au serveur distant. | Ouvrir `22/tcp` avant d'activer, et garder une session ouverte. |
| Ouvrir 0.0.0.0 (toutes interfaces) sans besoin | Surface d'attaque maximale. | N'exposer que les ports/IP nécessaires. |
| Oublier le state : bloquer les réponses sortantes | Les retours des connexions autorisées passent mal → app « cassée ». | Laisser `allow outgoing` ; contrôler le trafic entrant. |
| Confondre « port ouvert » et « pare-feu autorisé » | Un service écoute mais le pare-feu le bloque (ou l'inverse) | Tester les deux : `ss -tulpn` + `ufw status`. |
| Bloquer la gateway/ICMP par erreur | À diagnostiquer de l'extérieur, ça casse tout. | Laisser `ping` et laisser 22 toujours ouvert. |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`**, la correction dans **`03-correction.md`**.

**Énoncé court** : en machine de test (VM/Ubuntu test), installe ufw, règle le défaut deny à l'entrée, autorise 22, 80, 443, bloque 5432, puis vérifie l'état avec `ufw status`. Teste avec `nc` depuis un autre terminal que 443 est accessible et 5432 bloqué.

---

## 7. Correction détaillée de l'exercice

> La correction complète pas-à-pas est dans **`03-correction.md`**. Le raisonnement : autoriser SSH d'abord, régler le défaut, activer, puis tester les deux sens (port autorisé vs port bloqué) pour confirmer le pare-feu.

---

## 8. Checklist de validation

- [ ] J'explique le rôle d'un pare-feu et la différence réseau/applicatif.
- [ ] Je configure UFW : défaut deny incoming, `allow`/`deny` par port et par IP.
- [ ] J'active et j'inspecte avec `ufw status`, `ufw status numbered`.
- [ ] Je verbalise la règle d'ordre (SSH avant enable) et le risque de se couper l'accès.
- [ ] Je vérifie les ports réels avec `ss -tulpn`.
- [ ] Je connais le principe de moindre exposition et le lien avec les security groups cloud.

---

🧭 **Pont vers la suite** — Le pare-feu contrôle *qui* passe, mais sur Internet, pour garantir *confidentialité, intégrité et identité*, il faut **chiffrer** : c'est le domaine du **TLS/HTTPS**, sujet de la Leçon 4.

---

*Prochaine étape :* Leçon 4 — **TLS, HTTPS et certificats** dans `04-TLS-HTTPS-et-certificats`.