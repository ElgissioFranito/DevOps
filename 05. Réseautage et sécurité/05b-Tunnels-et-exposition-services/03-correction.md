# Correction — Leçon 5b : tunnels ponctuels et exposition de services

> **Bloc 5 · Leçon 5b** — Correction pas à pas.
> 🧭 **Articulation** : compare ta réalisation à chaque étape ; les questions ❓ sont répondues à la fin de chaque bloc, le mini-quiz à la section 2.

---

## 1. Correction pas à pas

### Étape 1 — Tunnel local `-L`

```bash
ssh -L 7999:localhost:5999 toto@TON-SERVEUR
# -L (local) : le port 7999 de TON poste devient une fenêtre vers le port 5999 VU DEPUIS le serveur
# premier localhost = « depuis le serveur » ; 5999 = le service privé ; 7999 = le port chez toi (libre, au choix)
```

❓ **Réponses** : `7999` = le port ouvert **sur ton poste** ; le premier `localhost` = l'adresse vue **depuis le serveur** (le serveur lui-même) ; `5999` = le port **réel** du service sur le serveur.

```bash
curl -I http://localhost:7999
# -I → n'affiche que les en-têtes ; réponse « HTTP/1.1 200 OK » = le service privé est joignable
# sans que le port 5999 soit exposé au public : TOUT passe dans le tunnel SSH chiffré
```

Après Ctrl+C, le `curl` **échoue** (« connection refused ») : le tunnel n'existait **que pendant la session SSH**. C'est la définition d'un tunnel **ponctuel** — et la preuve qu'aucun port n'est resté ouvert.

### Étape 2 — Tunnel reverse `-R`

```bash
ssh -R 8080:localhost:3000 toto@TON-SERVEUR
# -R (remote) : le port 8080 DU SERVEUR pointe vers le 3000 de TON poste
# le sens est inversé : tu PUSSES ton service local vers la machine joignable
```

```bash
curl -I http://localhost:8080   # sur le serveur
# réussit : ton app locale est devenue « visible » depuis le serveur
```

❓ **Réponse (le « pourquoi NAT »)** : le tunnel `-R` est **initié par ton poste**, dans le **sens sortant** — exactement comme quand tu surfs. Or le NAT **autorise toujours le sortant** (il note la connexion dans sa table pour router les réponses, zoom Leçon 5a §2.2) et **bloque l'entrant spontané**. Ton poste « téléphone » au serveur, et les données de ton app **remontent ce fil** : l'extérieur n'a jamais eu besoin de « sonner » chez toi.

### Étape 3 — Tunnel dynamique `-D`

```bash
ssh -D 1080 toto@TON-SERVEUR
# -D : proxy SOCKS5 local sur 1080 ; le navigateur y envoie ses requêtes, elles ressortent depuis le serveur
```

❓ **Réponse** : le résultat observé (l'IP de sortie = celle du serveur) ressemble au **full tunnel** de la Leçon 5a (§2.5) — mais le périmètre est différent : **un seul navigateur**, une seule session, rien d'autre sur ta machine n'est affecté, et rien à installer côté serveur. Pour **toute la machine**, de façon persistante → le VPN (Leçon 5a).

### Étape 4 — Cloudflare Tunnel

```bash
./cloudflared tunnel --url http://localhost:3000
# tunnel SORTANT : cloudflared se connecte de l'intérieur vers Cloudflare (comme une navigation web)
# Cloudflare renvoie les visiteurs de l'URL publique DANS ce tunnel, jusqu'à ton localhost:3000
```

❓ **Réponse (question piège)** : **zéro port ouvert**. Tu n'as rien configuré dans la box — le tunnel est **sortant**, il traverse le NAT sans redirection. C'est toute la valeur de l'approche « tunnel initié de l'intérieur ».

Après Ctrl+C, l'URL ne répond plus : le tunnel vivait **aussi longtemps que le processus**. Réflexe acquis : ce qui s'expose se referme — et vérifie avec `ss -tlnp` sur le serveur qu'aucun port `-R` ne traîne.

---

## 2. Réponses au mini-quiz

- **Q13 (les sens)** : `-L` = je vais **chercher** un port distant (il arrive chez moi) ; `-R` = je **pousse** un port local (il apparaît sur le serveur) ; `-D` = je fournis un **relais générique** (SOCKS) que des applications utilisent à la demande.
- **Q14 (NAT)** : ta box maintient une **table de traduction** remplie **par tes connexions sortantes** (zoom Leçon 5a §2.2) — un paquet entrant spontané n'y correspond à rien, il est jeté ; d'où l'asymétrie « je peux appeler, on ne peut pas m'appeler ». Les tunnels de cette leçon **s'initient tous de l'intérieur** : ils empruntent le sens toujours autorisé, donc ils traversent le NAT sans aucune configuration.
- **Q15 (production)** : **non**. Une URL `trycloudflare.com` est aléatoire, non garantie (pas de SLA — engagement de disponibilité), publique à quiconque a le lien. En production : `cloudflared` **en service systemd**, lié à **ton domaine**, avec une **authentification** devant (Cloudflare Access ou celle de l'app) — et l'autorisation de l'entreprise si le service est interne.
- **Q16 (le bon outil)** : (a) `ssh -L` — ponctuel, zéro installation ; (b) **VPN** (Leçon 5a) — persistant, réseau entier ; (c) `ssh -R` ou Cloudflare Tunnel — exposer une app locale rapidement.

---

## 3. Checklist de validation

- [ ] Je crée un tunnel local `-L` et je rejoins un service privé sans l'exposer.
- [ ] Je crée un tunnel reverse `-R` et j'explique pourquoi il traverse le NAT (initié de l'intérieur).
- [ ] Je configure un proxy SOCKS `-D` et je sais en quoi il diffère d'un full tunnel VPN.
- [ ] J'expose un service local en HTTPS avec `cloudflared` **sans ouvrir de port**.
- [ ] Je sais choisir l'outil selon le besoin (ponctuel → tunnel ; réseau entier → VPN ; production → reverse proxy, Leçon 6).
- [ ] Je ferme toujours mes tunnels et je le vérifie (`ss -tlnp`).

## 🧠 Conseils pour la suite

- La **Leçon 6 (reverse proxy)** reprend le fil : quand tu exposes **sérieusement** plusieurs services, tu remplaces « un tunnel par service » par **un point d'entrée unique** — et tu peux placer ce reverse proxy **derrière** le VPN (5a) ou un Cloudflare Tunnel.
- En entreprise, ces outils servent surtout en **dépannage ponctuel** ; l'accès régulier passe par le **VPN** (Leçon 5a) — c'est exactement le tableau d'aiguillage §2.4.
- Attention **sécurité** : un tunnel reverse mal surveillé est un trou dans le pare-feu (Leçon 3) que toi-même as creusé. Réflexe : tunnel ouvert = session visible dans ton terminal ; tunnel fini = Ctrl+C.
