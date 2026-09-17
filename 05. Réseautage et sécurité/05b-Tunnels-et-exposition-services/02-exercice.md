# Exercice — Leçon 5b : tunnels ponctuels et exposition de services

> **Bloc 5 · Leçon 5b** — Exercice **léger** : tout se fait avec SSH (déjà installé) et un service web local. Serveur : celui de la Leçon 5a (VPS de test ou VM VirtualBox). **Jamais** sur une machine de production.
> **Durée estimée : 45-60 min.**
> 🧭 **Articulation des fichiers** : `01-lecon.md` a expliqué les trois tunnels SSH et les tunnels sortants ; ici tu les **exécutes un par un** ; la correction est dans `03-correction.md` ; l'aide-mémoire dans `04-commandes-references.md`.

---

## Prérequis

- Un serveur Linux joignable par SSH (celui de la Leçon 5a fait parfaitement l'affaire).
- Sur ce serveur, installe un service « cible » (une petite base PostgreSQL du Bloc 2, ou à défaut un simple serveur web) :
  ```bash
  python3 -m http.server 5999 &
  # & → lance la commande en arrière-plan (le prompt revient)
  # ce serveur web joue le rôle de « service privé » sur le port 5999
  ```

---

## Étape 1 — Tunnel local `-L` : rejoindre un service privé (10 min)

1. Depuis ton poste, ouvre le tunnel : `ssh -L 7999:localhost:5999 toto@TON-SERVEUR`. ❓ Chaque élément est quoi : `7999`, le premier `localhost`, `5999` ?
2. **Dans un second terminal** (le tunnel occupe le premier), teste : `curl -I http://localhost:7999`.
   - `-I` → ne demande que les **en-têtes** (headers) de la réponse, pas le contenu.
3. Coupe le tunnel (Ctrl+C dans le terminal SSH) et refais le `curl` : ❓ que se passe-t-il, et pourquoi ?

## Étape 2 — Tunnel reverse `-R` : montrer une app locale (10 min)

4. Sur ton **poste**, lance une app web locale :
   ```bash
   python3 -m http.server 3000
   # sert le dossier courant sur le port 3000 (rôle : « ton app de dev »)
   ```
5. Ouvre le tunnel inverse : `ssh -R 8080:localhost:3000 toto@TON-SERVEUR`.
6. Sur le **serveur** (autre session SSH), vérifie : `curl -I http://localhost:8080`. ❓ Pourquoi ce `curl` réussit-il alors que ton poste est derrière une box/NAT et injoignable de l'extérieur ? *(rappel : zoom §2.3 de la leçon)*

## Étape 3 — Tunnel dynamique `-D` : proxy SOCKS (10 min)

7. Ouvre `ssh -D 1080 toto@TON-SERVEUR`.
8. Configure ton navigateur : proxy **SOCKS5** sur `localhost:1080` (Firefox : Paramètres → Paramètres réseau).
9. Visite `https://ifconfig.me` : l'IP affichée est celle du **serveur**. Compare avec un onglet sans proxy. ❓ Quel est le rapport avec le **full tunnel** de la Leçon 5a (§2.5) ?

## Étape 4 — Cloudflare Tunnel : HTTPS public sans ouvrir de port (10 min)

10. Sur ton poste : `python3 -m http.server 3000` (déjà lancé à l'étape 2), puis :
    ```bash
    ./cloudflared tunnel --url http://localhost:3000
    # binaire téléchargé comme en section 3.4 de la leçon ; ouvre un tunnel SORTANT
    ```
11. Ouvre l'URL `https://<hasard>.trycloudflare.com` affichée, **depuis ton téléphone en 4G** si possible. ❓ Quel port as-tu ouvert dans ta box pour que ça marche ? *(question piège !)*
12. Coupe le tunnel (Ctrl+C) et recharge l'URL : elle ne répond plus. Prends le réflexe : **ce qui s'expose se referme**.

## Étape 5 — Mini-quiz (de mémoire)

13. `-L`, `-R`, `-D` : quel sens fait circuler chaque tunnel, en une phrase chacun ?
14. ❓ *Question NAT* : ton PC derrière une box peut toujours **initier** des connexions vers Internet, mais Internet ne peut pas l'**initier**. Explique pourquoi en 2 phrases (zoom §2.3), et pourquoi les tunnels de cette leçon ne souffrent pas de cette limite.
15. ❓ Tu veux exposer en production un service interne de l'entreprise : `trycloudflare.com` convient-il ? Pourquoi ?
16. ❓ Repère le bon outil : (a) lire une base 5 min ; (b) accéder tous les jours au réseau du bureau ; (c) montrer ce soir une app de dev à un ami.

---

## ✅ Critères de réussite

- [ ] `curl -I http://localhost:7999` répond **tant que** le tunnel `-L` est ouvert, et échoue après.
- [ ] `curl -I http://localhost:8080` **sur le serveur** montre ton app locale (tunnel `-R`).
- [ ] Avec le proxy SOCKS, `https://ifconfig.me` affiche l'IP du serveur.
- [ ] L'URL `trycloudflare.com` a fonctionné **sans aucun port ouvert** dans ta box.
- [ ] Aucun tunnel laissé ouvert après l'exercice (`ss -tlnp` propre sur le serveur).
