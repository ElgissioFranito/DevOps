# Fiche de référence 02 — Debian 13 + Tailscale derrière un pare-feu Fortinet

> 🧭 **Pont depuis les leçons et la fiche 01** — Cette fiche raconte un **incident réel** de la vie d'un apprenti : installer un serveur de staging (Debian 13 dans Proxmox, voir fiche 01) et le joindre via **Tailscale** (un VPN mesh, Bloc 5 Leçon 5a)… quand le pare-feu d'entreprise **bloque Tailscale lui-même**. Elle réutilise : `systemctl`/`journalctl` (Bloc 2, Leçon 4), `curl`/HTTPS (Bloc 5, Leçons 1 et 4), pare-feu (Bloc 5, Leçon 3), certificats (Bloc 5, Leçon 4).
> **À quoi sert cette fiche ?** Une référence : « un outil HTTPS ne s'installe pas / ne s'authentifie pas derrière le pare-feu de l'entreprise → commencer ici ».

---

## 1. 📖 Vocabulaire / Abréviations

| Terme | Définition (1 ligne) |
|---|---|
| **Debian** | distribution Linux « mère » d'Ubuntu, très stable, standard des serveurs |
| **Version stable / Testing / Unstable** | maturité d'une version : testée / en cours de test / expérimentale |
| **Point release** (ex. 13.7) | mise à jour corrective d'une version (13 → 13.1 → … → 13.7), sans changement majeur |
| **ISO** | image disque d'installation gravable/montable |
| **netinst** | ISO d'installation **minimale** (« network install ») : télécharge le reste depuis Internet |
| **FortiGate / Fortinet** | marque d'équipement de sécurité d'entreprise (pare-feu avancé) |
| **SSL Inspection** (inspection TLS) | le pare-feu **déchiffre et réchiffre** le HTTPS pour l'inspecter, en présentant son propre certificat |
| **DPI** | Deep Packet Inspection : analyser le **contenu** des paquets, pas seulement leur adresse (rappel Bloc 5, Leçon 3 : niveau 7) |
| **Application Control** | fonction du pare-feu qui **reconnaît l'application** utilisée (par sa signature) et la bloque si catégorisée interdite |
| **CA** | Certificate Authority (autorité de certification) : organisme de confiance qui signe les certificats (Bloc 5, Leçon 4) |
| **`x509`** | le format standard des certificats (Bloc 5, Leçon 4) |
| **Trust store** | le « carnet d'adresses des CA de confiance » d'un système (Bloc 5, Leçon 4) |
| **`CA:FALSE`** | propriété d'un certificat qui **interdit** de l'utiliser comme autorité de signature → ne peut pas être ajouté comme CA |
| **Tailscale** | VPN moderne (basé sur WireGuard) créant un réseau privé entre tes machines (« tailnet ») |
| **tailnet** | le réseau privé virtuel de tes machines Tailscale (adresses `100.x.x.x`) |
| **Control plane** | le serveur central de coordination de Tailscale (`controlplane.tailscale.com`) |
| **auth-key** | clé d'authentification permettant d'enregistrer une machine **sans navigateur** (idéal serveur) |
| **tag** (Tailscale) | étiquette (ex. `tag:staging`) attribuée à une machine pour les règles d'accès |
| **ACL** | Access Control List : règles disant qui peut joindre quoi (rappel Bloc 5, Leçon 7) |

## 2. Étape 1 — Préparer le serveur (Debian 13 « Trixie »)

### 2.1 Quelle version choisir ?

- **Stable** = version longuement testée, avec correctifs de sécurité continus → le choix pour un serveur de production ou de **staging** (environnement de pré-production, Bloc 1, Leçon 3). Actuellement : **Debian 13 « Trixie »** (point release 13.7).
- **Testing / Unstable** = logiciels plus récents mais qui peuvent casser → jamais pour un serveur.

### 2.2 Quelle image ? La `netinst`

```text
debian-13.7.0-amd64-netinst.iso
```

**netinst** = image **minimale** (~750 Mo) : elle installe juste le noyau de base, puis **télécharge depuis Internet** uniquement ce que tu choisis. Parfait pour un serveur : système propre, contrôlé, sans environnement graphique inutile (moins de ressources, moins de **surface d'attaque** — rappel Bloc 5, Leçon 3).

### 2.3 Les choix d'installation et pourquoi

| Choix | Pourquoi |
|---|---|
| Pas d'environnement graphique | Un serveur s'administre en SSH (Bloc 2, Leçon 5) ; l'interface graphique coûte des ressources et agrandit la surface d'attaque |
| Cocher « SSH server » + « standard system utilities » | Le minimum pour se connecter et administrer |
| LVM (optionnel) | Permet de **redimensionner les partitions plus tard** (encart LVM du Bloc 2, Leçon 4) |

Première action après installation — **toujours** :
```bash
apt update && apt full-upgrade -y
# update : rafraîchit la liste des paquets ; full-upgrade -y : applique les mises à jour ( Bloc 2, Leçon 6)
```

---

## 3. Étape 2 — L'incident : « `tailscale up` ne marche pas »

### 3.1 L'objectif

Installer Tailscale sur le serveur de staging pour le joindre de n'importe où **sans ouvrir de ports sur Internet** (exactement le cas 8 de la fiche 01 : jamais le port exposé en direct).

### 3.2 Le symptôme

```bash
sudo tailscale up            # ou : sudo tailscale up --authkey=...
# → la commande reste bloquée, puis : « context canceled »
```

### 3.3 Le diagnostic (l'ordre qui a marché — les outils du Bloc 2/5)

```bash
sudo systemctl status tailscaled
# → « active (running) » mais « Needs login » : le SERVICE tourne (Bloc 2, Leçon 4),
#   mais il n'arrive pas à s'AUTHENTIFIER auprès du control plane
sudo journalctl -u tailscaled -f
# → les logs montrent les messages « Fortinet » et « certificate signed by unknown authority »
#   (les logs = meilleure source d'information — réflexe du Bloc 2, Leçon 4)
curl -v https://controlplane.tailscale.com
# -v : mode verbeux → montre si le serveur est joint, quel CERTIFICAT est présenté,
#   s'il est accepté… et ici, la page HTML de BLOCAGE du FortiGate
```

### 3.4 Les deux obstacles découverts (l'explication sans étonnement)

**Obstacle n°1 — L'inspection SSL (SSL Inspection / DPI).**
Le FortiGate **intercepte** toutes les connexions HTTPS : au lieu de laisser passer le vrai certificat de `controlplane.tailscale.com`, il présente **le sien**. Résultat :

```text
x509: certificate signed by unknown authority
```

**Analogie** : quelqu'un remplace le badge officiel à l'entrée du bâtiment par son propre badge. Ton système refuse de faire confiance — c'est *le comportement correct* de la vérification de certificats (Bloc 5, Leçon 4).

**Obstacle n°2 — L'Application Control (le vrai bloqueur).**
Même en traitant le certificat, on a reçu une page HTML :

```text
Application Blocked
Application : Tailscale
Category    : Proxy
```

Le FortiGate **reconnaît l'application** (par sa signature de trafic — c'est le niveau 7, comme un WAF du Bloc 5, Leçon 3) et la bloque volontairement car Tailscale est classé « Proxy ». **Cela ne se contourne pas côté client** : c'est une décision de l'administrateur réseau.

### 3.5 Pourquoi la piste « ajouter le certificat Fortinet au trust store » a échoué

```bash
echo | openssl s_client -connect controlplane.tailscale.com:443 \
  -servername controlplane.tailscale.com -showcerts 2>/dev/null > full-chain.pem
# s_client : examine le certificat TLS présenté (Bloc 5, Leçon 4) ; -showcerts : la chaîne complète
```

Découverte : le FortiGate n'envoyait qu'**un seul certificat** (pas une chaîne complète), avec **`CA:FALSE`** → il est interdit d'agir comme autorité de certification. L'installer dans `/usr/local/share/ca-certificates/` ne peut donc **pas** faire confiance à ce qu'il signe ensuite. *Diagnostic propre, conclusion propre : ce n'était pas la bonne piste.*

## 4. Bilan des tentatives (et pourquoi chacune a échoué)

| Tentative | Résultat | Pourquoi |
|---|---|---|
| `tailscale up` simple | ❌ Échec | Bloqué par le FortiGate (les 2 obstacles) |
| Ajouter le certificat Fortinet au trust store | ⚠️ Partiel | Le certificat n'était pas un CA (`CA:FALSE`) |
| `--force-reauth` | ❌ Échec | Le problème est réseau, pas d'authentification |
| `--auth-key` | ❌ Échec derrière FortiGate | Le control plane reste injoignable quoi qu'on fasse |
| Passer par un autre VPN (temporairement) | ✅ A permis d'avancer | On a contourné le FortiGate le temps de l'installation |

> 💡 **La leçon à retenir** : quand un pare-feu cumule **SSL Inspection + Application Control**, régler uniquement le certificat ne suffit pas. Les contournements côté client sont limités : **la solution propre est une exception côté pare-feu** (demander à l'admin réseau d'autoriser Tailscale).

## 5. La bonne façon de lancer `tailscale up`

> ⚠️ **Piège connu** : Tailscale **mémorise** les options déjà passées. Si tu changes un flag sans le redéclarer, il te demande les autres — d'où le `--reset`.

```bash
sudo tailscale up --reset \
  --auth-key=tskey-auth-XXXXXXXX \
  --accept-routes \
  --advertise-tags=tag:staging \
  --ssh
```

| Option | À quoi ça sert |
|---|---|
| `--reset` | Oublie les anciens réglages mémorisés |
| `--auth-key=...` | Authentifie la machine **sans navigateur** — indispensable sur un serveur |
| `--accept-routes` | Accepte les routes annoncées par les autres machines du tailnet |
| `--advertise-tags=tag:staging` | Étiquette la machine (utile pour les ACL : « autoriser tag:staging vers X ») |
| `--ssh` | Active Tailscale SSH (connexion SSH **via le tunnel**, Bloc 5, Leçons 5a-5b) |

Vérification : `tailscale status` → la machine doit apparaître avec son IP `100.x.x.x`.

## 6. Quand ce savoir resservira-t-il ?

| Situation | Ce qui te servira |
|---|---|
| Entreprise avec Fortinet / Palo Alto / Cisco | Suspecter d'abord **SSL Inspection + Application Control** |
| Serveur derrière un proxy d'entreprise | Même symptôme de certificat inconnu (`x509…`) |
| Machine sans accès Internet direct | S'authentifier d'abord via un autre chemin (VPN, 4G), puis revenir proprement |
| N'importe quel outil qui parle HTTPS ne s'installe pas | `curl -v` + `openssl s_client` = les deux premières investigations |
| Gestion d'un parc Tailscale | Toujours **auth-key + tags**, jamais un login navigateur sur un serveur |

## 7. Ce qu'il faut retenir (version courte)

1. **Debian stable + netinst** = le réflexe serveur (propre, léger, sécurisé).
2. Derrière un FortiGate, il y a souvent **deux** obstacles : l'inspection SSL (certificat) **puis** le blocage applicatif — régler le premier ne suffit pas face au second.
3. Le trio de diagnostic : `systemctl status` → `journalctl -u … -f` → `curl -v`. Les **logs racontent toujours** ce que le symptôme ne dit pas.
4. Sur un serveur : **auth-key + `--reset` + tags** = la façon propre de connecter Tailscale.
5. La solution durable face à un blocage d'entreprise = **exception côté pare-feu**, pas de contournement bricolé.

## 8. Prochaines actions (sur site)

1. Se connecter en SSH à la machine Debian (`ssh user@<ip>`, Bloc 2, Leçon 5).
2. Lancer la commande complète de la section 5 (avec `--reset`).
3. Vérifier avec `tailscale status`.
4. Si ça bloque encore : **demander l'exception Tailscale** à l'admin du FortiGate (c'est la voie normale et professionnelle — rien de honteux : les pare-feu font leur travail).

---

🧭 **Fiche liée** : `01-Proxmox-et-diagnostic.md` — le serveur hôte, ses bridges et sa méthode de diagnostic réseau.

*Fiche 02 — basée sur une session réelle de diagnostic (septembre 2026).*


