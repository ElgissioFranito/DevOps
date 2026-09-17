# Leçon 5b — Tunnels ponctuels et exposition de services (SSH & Cloudflare Tunnel)

> **Bloc 5 · Réseautage et sécurité** — Leçon 5b (seconde moitié de la Leçon 5)
> 🧭 **Pont depuis la Leçon 5a (VPN)** : tu sais maintenant relier **durablement** une machine ou un réseau entier en privé, avec WireGuard. Mais tous les besoins ne durent pas des semaines : lire une base **5 minutes**, montrer une app de dev **ce soir**, ou exposer un service **sans ouvrir de port** dans la box. Pour ces besoins **ponctuels**, il existe plus léger qu'un VPN : les **tunnels** — et c'est exactement le sujet de cette leçon.
> 👉 **Fil rouge de la leçon** : choisir le **bon outil pour le bon besoin** grâce au zoom NAT (rappel ci-dessous) et au tableau d'aiguillage final.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Distinguer** VPN (réseau entier, persistant — Leçon 5a) et **tunnel ponctuel** (un port, une session), et choisir l'un ou l'autre selon le besoin.
2. **Créer** les trois tunnels SSH : local (`-L`), **reverse** (`-R`) et dynamique (`-D`) — et savoir **quel sens** chacun fait circuler.
3. **Expliquer** le **reverse SSH tunnel** et pourquoi il contourne le **NAT** (rappel : ta machine derrière une box est injoignable de l'extérieur).
4. **Exposer** un service interne sur Internet avec **Cloudflare Tunnel** ou **ngrok**, **sans IP publique ni port ouvert**.
5. **Éviter** les pièges de sécurité de ces outils (tunnel laissé ouvert, service privé exposé par erreur).

---

## 2. Explication simple

### 2.1 Le « pourquoi » : un VPN, c'est souvent trop

La Leçon 5a t'a appris à monter un **VPN** : un tunnel **persistant** qui fait de ta machine une **vraie habitante** du réseau distant. C'est parfait pour un usage **régulier** (tous les jours, plusieurs services). Mais imagine : tu veux lire **une seule donnée** dans une base, **une seule fois**. Monter un VPN pour ça (générer des clés, écrire deux configs, ouvrir un port au pare-feu, démarrer un service), c'est comme **louer un couloir privé permanent** entre deux immeubles pour **une seule visite**.

> 💡 **Analogie** : le VPN, c'est le **couloir fermé permanent** entre deux immeubles (Leçon 5a). Le tunnel ponctuel, c'est **emprunter le service de messagerie** : tu envoies une enveloppe précise à une adresse précise, une fois, et c'est fini. Moins de mise en place — mais un seul « colis » à la fois.

### 2.2 Le « comment » : SSH, le tunnel déjà installé

Tu connais déjà **SSH** (Bloc 2) : une connexion chiffrée vers un serveur. Surprise : SSH sait aussi faire **circuler un port à travers** cette connexion. On écrit `ssh -X portA:destination:portB` et le port A devient une **fenêtre** qui donne sur le port B, à travers le tunnel chiffré SSH.

La règle d'or : la machine **qui lance la commande SSH** doit pouvoir joindre la **destination** (le vrai service). Tout le reste, SSH le gère.

### 2.3 Zoom NAT : pourquoi certains tunnels sont « inversés »

Rappel du zoom §2.2 de la Leçon 5a : ton PC à la maison a une **IP privée** (`192.168.x.x`) et **partage** l'IP publique de la box ; de l'extérieur, **personne ne peut « sonner » chez toi** spontanément — sauf règle de **port forwarding** (redirection de port) configurée dans la box.

Conséquence pratique, lisible en une phrase : **ta machine peut toujours « appeler » l'extérieur (connexion sortante), mais l'extérieur ne peut pas l'appeler**. C'est exactement ce que la table de NAT oublie après un moment d'inactivité — et pourquoi WireGuard a besoin de `PersistentKeepalive`.

Les tunnels de cette leçon exploitent cette asymétrie : ils **s'initient tous depuis l'intérieur**. Le plus spectaculaire est le **reverse SSH tunnel** : ta machine **pousse** un de ses ports vers un serveur joignable — et c'est le serveur qui devient la « vitrine » du service qui tourne chez toi.

### 2.4 Le « quand » : quel outil pour quel besoin ?

| Besoin | Outil | Pourquoi |
|---|---|---|
| Lire une base privée **5 minutes** | `ssh -L` (tunnel local) | zéro installation, zéro config serveur |
| Montrer **une app locale** à un collègue (PC derrière le NAT) | `ssh -R` (reverse) ou Cloudflare Tunnel/ngrok | la machine initie la connexion → traverse le NAT |
| Exposer un service **sans ouvrir de port**, en HTTPS | **Cloudflare Tunnel** (`cloudflared`) ou **ngrok** | tunnel sortant + TLS inclus |
| Accéder à **plusieurs services / un réseau entier**, tous les jours | **VPN** (Leçon 5a) | persistant, machine = habitante du réseau |
| Exposer **sérieusement** (production, domaine à soi, plusieurs services) | **reverse proxy** (Leçon 6) | point d'entrée unique, load balancing |

> 🧭 **Lien avec la suite** : ces outils sont **complémentaires**, pas concurrents. La Leçon 6 (reverse proxy) placera volontiers son point d'entrée unique **derrière** un VPN (Leçon 5a) ou un Cloudflare Tunnel.

## 📖 Vocabulaire / Abréviations

| Terme | Définition (une ligne) |
|---|---|
| **Tunnel** | canal chiffré qui transporte le trafic d'un port (ou d'un réseau) à travers une connexion existante |
| **Tunnel local** (`-L`) | un port **de ma machine** devient une fenêtre vers un port vu **depuis le serveur** |
| **Reverse SSH tunnel** (`-R`) | on **pousse** un port de sa machine (derrière NAT) vers le serveur, qui en devient la vitrine |
| **Tunnel dynamique** (`-D`) | proxy **SOCKS** local : le navigateur fait passer tout son trafic par le serveur |
| **SOCKS** | protocole générique de « relais de ports » : l'app configure un seul point, il route tout |
| **Tunnel sortant** (outbound) | connexion initiée **de l'intérieur** vers l'extérieur — traverse le NAT sans port forwarding |
| **NAT** | rappel Leçon 5a (zoom §2.2) : la box traduit « plusieurs IP privées ↔ une IP publique » ; l'extérieur ne peut pas initier de connexion vers l'intérieur |
| **Cloudflare Tunnel** / **`cloudflared`** | service qui expose un service interne sur Internet via un tunnel sortant, sans IP publique ni port ouvert |
| **ngrok** | équivalent générique de Cloudflare Tunnel, très utilisé en développement |
| **WAF** (Web Application Firewall) | pare-feu applicatif : filtre les requêtes HTTP malveillantes avant qu'elles n'atteignent le service |
| **Domaine** | le nom lisible d'un site (`mon-app.exemple.com`), résolu en IP par le DNS (Leçon 9) |

---

## 3. Exemples concrets

> 🧭 **Transition** : la théorie est posée (pourquoi §2.1, comment §2.2, le rôle du NAT §2.3). Passons aux commandes — toutes **copiables-collables** et sans installation (SSH est déjà partout).

### 3.1 Tunnel local (`-L`) : joindre une base privée

```bash
ssh -L 5433:localhost:5432 toto@mon-serveur
# -L (local) : le port 5433 de MA machine pointe vers le port 5432 VU DEPUIS le serveur
# localhost:5432 = « depuis le serveur, localhost est lui-même » → sa base PostgreSQL (5432)
# pendant que la session est ouverte, je me connecte avec : psql -h localhost -p 5433
```

Le schéma mental : `mon PC:5433 ──tunnel SSH──> serveur ──> serveur:5432`. Si la base tourne sur une **autre machine** que le serveur SSH, remplace le premier `localhost` : `ssh -L 5433:10.0.0.5:5432 toto@mon-serveur`.

### 3.2 Tunnel reverse (`-R`) : montrer une app locale

```bash
ssh -R 8080:localhost:3000 toto@mon-serveur
# -R (remote) : le port 8080 DU SERVEUR pointe vers le 3000 de MA machine
# ton collègue ouvre http://IP-publique-du-serveur:8080 → il voit ton app locale
# ⚠️ par défaut SSH n'écoute que sur localhost du serveur (sûr) ;
# ouvrir au public nécessite GatewayPorts=yes côté serveur — évite, préfère 3.4
```

C'est le **reverse SSH tunnel** : le sens est inversé par rapport à `-L` — tu **pousses** ton service local vers la machine joignable. C'est la solution quand ton PC est **derrière le NAT** (zoom §2.3).


### 3.3 Tunnel dynamique (`-D`) : un proxy SOCKS

```bash
ssh -D 1080 toto@mon-serveur
# -D (dynamic) : proxy SOCKS local sur le port 1080
# configure le navigateur → « serveur proxy SOCKS5 : localhost:1080 »
# tout le trafic du navigateur ressort DEPUIS le serveur (chiffré jusqu'à lui)
```

En pratique : c'est la version « un navigateur, une session » du full tunnel (Leçon 5a, §2.5) — sans VPN à monter.

### 3.4 Cloudflare Tunnel : exposer un service sans ouvrir de port

Le principe : un petit programme, **`cloudflared`**, tourne sur ta machine et ouvre un **tunnel sortant** (il se connecte de l'intérieur vers Cloudflare — le NAT ne gêne pas, c'est TA machine qui initie). Cloudflare fournit alors un domaine public HTTPS qui « tombe » dans ce tunnel.

```bash
# test sans compte ni installation complexe (mode « quick tunnel ») :
# 1) lance n'importe quel petit serveur web local :
python3 -m http.server 3000
# -m http.server → module Python qui sert le dossier courant sur le port 3000

# 2) dans un autre terminal, récupère cloudflared puis ouvre le tunnel :
curl -fsSL https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64 -o cloudflared
# -f → échoue si l'URL renvoie une erreur ; -s → silencieux ; -S → montre les erreurs ; -L → suit les redirections
chmod +x cloudflared
# +x → rend le fichier exécutable
./cloudflared tunnel --url http://localhost:3000
# ouvre un tunnel SORTANT vers Cloudflare et affiche une URL https://<hasard>.trycloudflare.com
# ouvre-la depuis un navigateur — même depuis ton téléphone en 4G : ton service local est joignable,
# SANS IP publique, SANS port forwarding, SANS port ouvert
```

**En usage sérieux** (un service à garder) : on installe `cloudflared` **en service** (`systemctl`, rappel Bloc 2) sur la machine, on le connecte à **son propre domaine** dans le tableau de bord Cloudflare, et on obtient `mon-app.mon-domaine.fr` en HTTPS permanent — avec en bonus le **WAF** (pare-feu applicatif) et le cache de Cloudflare devant le service.

### 3.5 ngrok : l'alternative générique

```bash
ngrok http 3000
# crée un tunnel sortant vers ngrok (il faut un compte gratuit) et affiche une URL publique HTTPS
```

Même mécanique que Cloudflare Tunnel ; pratique pour le partage rapide en dev.

> 📌 **Résumé mental** : montrer un service **ponctuellement** → `ssh -R` ; exposer un service web **sans ouvrir de port** → Cloudflare Tunnel / ngrok ; **plusieurs services, production** → reverse proxy (Leçon 6) ; **relier des réseaux** → VPN (Leçon 5a).


---

## 4. Bonnes pratiques modernes (2025-2026)

- **SSH d'abord** : pour un besoin ponctuel, `-L` suffit et ne nécessite **aucune installation** — réflexe par défaut.
- **Tunnel éphémère = vérifier qu'il est bien fermé** : Ctrl+C coupe la session SSH → le port exposé se ferme. Ne laisse pas un `-R` tourner sans y penser.
- **Cloudflare Tunnel en service** pour du durable : `sudo cloudflared service install`, jamais un `quick tunnel` de production (URL aléatoire, non garantie).
- **Méfiance avec les URL temporaires partagées** : quiconque a le lien peut accéder au service — ajoute une authentification devant (Cloudflare Access, auth de l'app).
- **Ne contourne pas les règles de l'entreprise** : exposer un service interne via ngrok/Cloudflare sans autorisation est une faille de sécurité (et une faute) — en entreprise, passe par le VPN (Leçon 5a).
- **Le NAT n'est pas un ennemi** : l'asymétrie « je peux appeler, on ne peut pas m'appeler » est une **protection**. Ces outils la contournent proprement, sans ouvrir de trou.

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi c'est dangereux/inefficace | ✅ Version correcte |
|---|---|---|
| Un `ssh -R` laissé tourner en arrière-plan | le service local reste exposé sur le serveur sans qu'on y pense | Ctrl+C quand c'est fini ; vérifier avec `ss -tlnp` sur le serveur |
| `GatewayPorts=yes` « pour tester » et jamais retiré | n'importe qui sur Internet atteint ton service local | garder l'écoute sur `localhost` du serveur + accès via tunnel/VPN |
| URL `trycloudflare.com` ou ngrok en production | URL aléatoire, pas de SLA (garantie de service), souvent publique | `cloudflared` en service + domaine à soi + authentification devant |
| Monter un VPN complet pour une lecture de 5 min | heures de setup pour un besoin d'une session | `ssh -L` en une ligne |
| Exposer sa base (5432) au public « plus simple » que le tunnel | attaques constantes sur les ports de base (Leçon 3) | `ssh -L` ou VPN ; la base reste fermée au public |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`**, la correction dans **`03-correction.md`**, l'aide-mémoire dans **`04-commandes-references.md`**.

**Énoncé court** : sur le serveur de la Leçon 5a (ou n'importe quel serveur où tu as un compte), rejoins une base privée avec `ssh -L`, montre une app locale avec `ssh -R`, navigue via `ssh -D`, puis expose un service local en HTTPS avec `cloudflared` — et réponds au mini-quiz (NAT inclus).

## 7. Correction détaillée de l'exercice

> La correction complète pas-à-pas est dans **`03-correction.md`**. Le raisonnement : chaque tunnel **s'initie depuis l'intérieur** (donc traverse le NAT), chaque tunnel a un **sens** (`-L` : je vais chercher ; `-R` : je pousse), et tout ce qui s'expose se **referme** après usage.

## 8. Checklist de validation

- [ ] Je distingue VPN (réseau, persistant — 5a) et tunnel ponctuel (un port, une session — 5b).
- [ ] Je crée un tunnel local `-L` et je rejoins un service privé sans l'exposer.
- [ ] Je crée un tunnel reverse `-R` et j'explique pourquoi il traverse le NAT.
- [ ] Je crée un proxy SOCKS `-D` et je le configure dans un navigateur.
- [ ] J'expose un service local en HTTPS avec `cloudflared` (ou ngrok) **sans ouvrir de port**.
- [ ] Je sais quel outil choisir selon le besoin (tableau §2.4) et je ferme toujours mes tunnels.

---

🧭 **Pont vers la suite (Leçon 6)** — Tu sais maintenant **atteindre** et **exposer** des services ponctuellement. Mais quand **plusieurs services** cohabitent en production (front **Angular**, backend **Spring Boot**), on ne veut pas créer un tunnel par service : on place **un seul point d'entrée** devant — le **reverse proxy** — capable aussi de **répartir la charge** entre serveurs. C'est la Leçon 6.

---

*Prochaine étape :* Leçon 6 — **Reverse Proxy et Load Balancing** dans `06-Reverse-Proxy-et-Load-Balancing/`.

