# Exercice — Leçon 5a : VPN (accès privé avec WireGuard)

> **Bloc 5 · Leçon 5a** — Exercice à réaliser sur un **VPS de test** (à partir de ~5 €/mois, ou essai gratuit) **ou** avec **deux VMs locales** (VirtualBox : une VM « serveur », une VM « client », en réseau « accès par pont »). **Jamais** sur une machine de production.
> **Durée estimée : 60-90 min.**
> 🧭 **Articulation des fichiers** : `01-lecon.md` a expliqué le pourquoi et le comment ; ici tu montes un **vrai tunnel VPN de A à Z** ; la correction pas-à-pas est dans `03-correction.md` ; l'aide-mémoire des commandes est dans `04-commandes-references.md`. Les **tunnels SSH et Cloudflare Tunnel** (besoins ponctuels) ont leur propre exercice dans la **Leçon 5b**.

---

## Prérequis

- Un serveur Linux (Ubuntu 22.04/24.04) que tu appelleras **serveur**, joignable par SSH depuis ton poste.
- Ton poste (Ubuntu/WSL) = **client**.
- Variante 100 % gratuite : deux VMs VirtualBox en réseau « accès par pont » (chaque VM a une IP du même réseau local ; la VM « serveur » joue le rôle du VPS).

---

## Étape 1 — Préparer les clés sur le serveur (10 min)

1. Installe WireGuard et le générateur de QR code :
   ```bash
   sudo apt update && sudo apt install -y wireguard wireguard-tools qrencode
   ```
2. Génère la **paire de clés du serveur** (une commande, avec `wg genkey` et `wg pubkey`).
3. Génère la **paire de clés du client** (`client1`), avec permissions `600`.
4. ❓ **Question** : pourquoi une clé **par appareil** plutôt qu'une clé partagée ?

## Étape 2 — Configurer le tunnel (15 min)

5. Écris `/etc/wireguard/wg0.conf` **côté serveur** : réseau `10.66.66.0/24`, serveur `10.66.66.1`, client `10.66.66.2`, port `51820`.
6. Écris `/etc/wireguard/wg0.conf` **côté client** : `Endpoint` = IP publique du serveur, `AllowedIPs` en **split tunnel**, `PersistentKeepalive = 25`.
7. Configure le **pare-feu** du serveur : ouvre **exactement un port**. ❓ Lequel ? Quel protocole ? (rappel Leçon 3)

## Étape 3 — Monter et vérifier (15 min)

8. Démarre le tunnel des deux côtés (`wg-quick up wg0`), puis rends-le **persistant** côté serveur (au reboot).
9. Vérifie : `sudo wg show` sur le serveur **et** le client, puis `ping 10.66.66.1` depuis le client. Que observes-tu (handshake, trafic) ?
## Étape 4 — Prouver le périmètre (10 min)

10. Depuis le client, teste ta sortie Internet :
    ```bash
    curl https://ifconfig.me
    # -s n'est pas nécessaire ici ; la commande affiche l'IP publique avec laquelle Internet te voit
    ```
    En split tunnel, l'IP affichée doit être **ta vraie IP**. Passe temporairement en full tunnel (`AllowedIPs = 0.0.0.0/0`), refais le test : l'IP devient **celle du serveur**. Note la différence, puis **remets le split tunnel**.
11. ❓ Pendant le full tunnel, ta navigation web est aussi transportée dans le tunnel. Quel avantage sur un Wi-Fi public ? Quel inconvénient au quotidien ?

## Étape 5 — Bonus (optionnel)

12. Génère le QR code pour ton téléphone (section 3.6 de la leçon) et connecte-le. `sudo wg show` doit alors afficher **deux peers**.
13. Mini-quiz (réponds de mémoire) :
    - Quel port/protocole WireGuard utilise-t-il par défaut ?
    - Quel fichier ne se partage **jamais** ?
    - Que signifie `AllowedIPs = 10.66.66.2/32` côté serveur, vs `AllowedIPs = 10.66.66.0/24` côté client ?
    - Un besoin ponctuel (lire une base pendant 5 min) : VPN ou tunnel SSH ? *(indice : réponse en Leçon 5b — tu peux déjà tenter ta chance)*
    - Un pare-feu d'entreprise ne connaît qu'OpenVPN : que montes-tu ?
    - Tailscale ou NetBird remplacent-ils WireGuard ? Que t'apportent-ils **par-dessus** ?
    - ❓ *Question NAT* : pourquoi ton serveur VPN **hébergé chez toi** a besoin d'une règle de **port forwarding** dans la box, alors qu'un VPS (cloud) n'en a pas besoin ?
14. **Optionnel (10 min)** : installe Tailscale (gratuit) sur ton PC et ton téléphone (`curl -fsSL https://tailscale.com/install.sh | sh` côté Linux, ou l'app mobile), connecte-toi avec le même compte, puis `tailscale ping <ip-du-téléphone>`. Observe : aucune config de clés à écrire — le serveur de coordination a tout fait. Relis l'encart mesh de la leçon et dis ce qu'il t'a épargné par rapport à l'étape 1.

## Étape 6 — Bonus « site-à-site » (optionnel, ~20 min)

> Rappel Leçon 2 (adresse IP, masque/CIDR) : un sous-réseau, ex. `192.168.10.0/24`, est un groupe d'adresses qui se voient sans routeur. Ici on va faire en sorte que **deux sous-réseaux** se voient à travers un tunnel.
> Pour ce bonus, utilise **trois VMs** (ou simule mentalement en étudiant la section 3.8 de la leçon) : une VM **passerelle A** (deux cartes réseau : une vers ton LAN, une vers Internet), une VM **passerelle B**, et une VM **simple client** derrière la passerelle A **sans** WireGuard installé.

15. Active l'**IP forwarding** sur les deux passerelles (technique de la section 3.7 de la leçon) — sinon elles ne retransmettront pas les paquets.
16. Génère une paire de clés **par passerelle** et écris deux configs miroir (modèle en section 3.8) : réseau de tunnel `10.77.77.0/24`, passerelle A `10.77.77.1`, passerelle B `10.77.77.2`, réseau A `192.168.10.0/24`, réseau B `192.168.20.0/24`. ❓ Quelles `AllowedIPs` écris-tu de chaque côté, et en quoi diffèrent-elles de celles de l'accès distant (étape 2) ?
17. Monte le tunnel (`wg-quick up wg0` des deux côtés), puis depuis la **VM client simple**, fais `ping 192.168.20.1` (une IP de la passerelle B dans son réseau local). ❓ Pourquoi ce ping fonctionne-t-il alors que la VM n'a jamais entendu parler de WireGuard ?

---

## ✅ Critères de réussite

- [ ] `sudo wg show` affiche un **handshake** et du trafic des deux côtés.
- [ ] `ping 10.66.66.1` répond depuis le client.
- [ ] `sudo ufw status verbose` n'expose que `51820/udp` (SSH accepté uniquement via `wg0`, sinon rien).
- [ ] `curl https://ifconfig.me` prouve que tu maîtrises split vs full tunnel.
- [ ] Aucune clé dans un dépôt Git, permissions 600 sur les `.key`.
- [ ] *(Bonus)* Je sais expliquer comment un site-à-site diffère d'un accès distant (`AllowedIPs` = sous-réseaux, IP forwarding, config par passerelle).

