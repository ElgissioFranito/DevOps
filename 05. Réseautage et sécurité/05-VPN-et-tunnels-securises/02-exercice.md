# Exercice — Leçon 5 : VPN et tunnels sécurisés

> **Bloc 5 · Leçon 5** — Exercice à réaliser sur un **VPS de test** (à partir de ~5 €/mois, ou essai gratuit) **ou** avec **deux VMs locales** (VirtualBox : une VM « serveur », une VM « client », en réseau « accès par pont »). **Jamais** sur une machine de production.
> **Durée estimée : 60-90 min.**
> 🧭 **Articulation des fichiers** : `01-lecon.md` a expliqué le pourquoi et le comment ; ici tu montes un **vrai tunnel de A à Z** ; la correction pas-à-pas est dans `03-correction.md` ; l'aide-mémoire des commandes est dans `04-commandes-references.md`.

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
    - Un besoin ponctuel (lire une base pendant 5 min) : VPN ou tunnel SSH ?
    - Un pare-feu d'entreprise ne connaît qu'OpenVPN : que montes-tu ?

---

## ✅ Critères de réussite

- [ ] `sudo wg show` affiche un **handshake** et du trafic des deux côtés.
- [ ] `ping 10.66.66.1` répond depuis le client.
- [ ] `sudo ufw status verbose` n'expose que `51820/udp` (SSH accepté uniquement via `wg0`, sinon rien).
- [ ] `curl https://ifconfig.me` prouve que tu maîtrises split vs full tunnel.
- [ ] Aucune clé dans un dépôt Git, permissions 600 sur les `.key`.

