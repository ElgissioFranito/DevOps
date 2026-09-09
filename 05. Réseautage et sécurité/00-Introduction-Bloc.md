# Introduction au Bloc 5 — Réseautage et sécurité

> **À lire en premier**, avant la Leçon 1. Ce fichier te dit :
> - de quoi parle ce bloc et **pourquoi il est central en DevOps**,
> - ce qu'il te faut **préparer** avant de commencer,
> - les **7 leçons** du bloc et le **fil rouge** qui les relie,
> - le vocabulaire que tu vas croiser, et ce que tu sauras faire à la fin.

---

## 🎯 De quoi parle ce bloc ?

Dans les blocs précédents, ton code est versionné (Git) et tu sais l'automatiser (Bash/Python). Mais un DevOps passe sa vie à se poser deux questions : **« comment les machines communiquent ? »** et **« pourquoi une application est-elle accessible ou non, et en sécurité ? »**.

Ce bloc répond à ces questions :
- **Réseau** : les protocoles, l'adressage IP, le diagnostic, le pare-feu.
- **Exposition** : TLS/HTTPS, reverse proxy et load balancing.
- **Accès & code** : contrôle d'accès, secrets, et intégration de la sécurité dès le départ (DevSecOps).

Objectif de la roadmap : *« un problème réseau ne te paraît plus magique »* et *« tu peux expliquer où circule une requête HTTP et à quel niveau elle peut être bloquée »*. C'est un **socle bloquant** : le cloud (Bloc 6), Docker (Bloc 9) et Kubernetes (Bloc 10) supposent que ces notions sont acquises.

> 💡 **Bloc très concret** : chaque leçon se pratique en terminal. On n'a besoin que d'une machine de test (VM/WSL) et d'accès Internet pour les exemples publics.

---

## ✅ Prérequis et préparation

- **Les Blocs 01-04** : le parcours « du Git à la production » (Bloc 01) et les blocs Linux/scripting. Le schéma `DNS → IP → port → service` y a été évoqué : on l'approfondit ici.
- **Une machine de test** : VM (ex. VirtualBox) ou WSL ou VPS de test — pour les leçons pare-feu et reverse proxy. **Jamais** une machine de production.
- **Outils à installer en cours de route** (chaque leçon le précise) : `curl`, `dig`/`dnsutils`, `openssl`, `ufw`, `nginx`. Sur Ubuntu/WSL, la plupart sont déjà là ou s'installent via `apt`.
- **Un éditeur** : `nano` (suffit) ou VS Code.

---

## 🗺️ Les 7 leçons du bloc (et le fil rouge)

Le **fil rouge** : *« comprendre pourquoi une application (frontend Angular + backend Spring Boot + base PostgreSQL) est accessible ou pas — et comment la sécuriser »* (le schéma de la roadmap section 5).

| # | Leçon | Compétence |
|---|-------|------------|
| 1 | Protocoles de communication | TCP, UDP, HTTP, HTTPS, DNS, SSH ; ports ; outillage `curl`/`nc`/`dig` |
| 2 | Adressage IP et diagnostic | IP, CIDR, subnet, gateway, privée/publique ; `ping`/`traceroute`/`ip` |
| 3 | Pare-feu et contrôle des flux | UFW, open/filtré, défaut-deny, moindre exposition, L3/L4/L7 |
| 4 | TLS, HTTPS et certificats | Clés, certificats, CA ; `openssl` ; erreurs TLS |
| 5 | Reverse Proxy et Load Balancing | Nginx, upstream, health check, failover, L4/L7 |
| 6 | Contrôle d'accès et secrets | Auth vs autorisation, RBAC/ABAC, `.env`, `.gitignore`, coffres |
| 7 | DevSecOps et Shift-Left | SAST/DAST, dependency/container/secret scan, CVE, pipeline |

Chaque dossier contient 4 fichiers : `01-lecon.md`, `02-exercice.md`, `03-correction.md`, `04-commandes-references.md`.

> 🔁 **Comment s'articulent les fichiers** : lis d'abord `01-lecon.md` (la théorie), puis fais `02-exercice.md` en autonomie, et compare avec `03-correction.md`. La `04-commandes-references.md` est l'aide-mémoire à garder à côté.

---

## 🧠 Vocabulaire que tu vas croiser

| Terme | C'est quoi ? (1 phrase) | Tu l'apprendras |
|-------|--------------------------|-----------------|
| **Protocole** | Un ensemble de règles pour que les machines se comprennent | Leçon 1 |
| **IP / port** | Adresse d'une machine / guichet d'un service | Leçons 1-2 |
| **DNS** | L'annuaire qui transforme un nom en IP | Leçon 1 |
| **CIDR** | Notation d'une plage d'adresses (`/24`) | Leçon 2 |
| **Pare-feu (firewall)** | Le gardien qui filtre les flux | Leçon 3 |
| **TLS / HTTPS** | Le chiffrement qui protège les échanges | Leçon 4 |
| **Certificat / CA** | « Carte d'identité » d'un serveur, signée par une autorité | Leçon 4 |
| **Reverse proxy** | Le point d'entrée unique devant les applications | Leçon 5 |
| **Load balancer** | Le répartiteur de charge entre serveurs | Leçon 5 |
| **RBAC / ABAC** | Modèles de contrôle d'accès par rôles / attributs | Leçon 6 |
| **Secret** | Mot de passe/clé/token à ne jamais committer | Leçon 6 |
| **DevSecOps / Shift-left** | Intégrer la sécurité tôt dans le cycle | Leçon 7 |
| **SAST / DAST / CVE** | Familles de scanners / identifiant de faille | Leçon 7 |

---

## ✅ Bloc acquis si

Tu peux, **de mémoire** :

- expliquer la **chaîne** `DNS → IP → route → firewall → port → service → application` et identifier où une requête peut être bloquée ;
- diagnostiquer avec `ping`, `traceroute`, `curl`, `nc`, `dig`, `ip` ;
- configurer un pare-feu simple (UFW) en respectant le moindre exposition ;
- expliquer et manipuler un certificat TLS avec `openssl` ;
- expliquer pourquoi on place un reverse proxy/load balancer et en écrire une config Nginx simple ;
- appliquer le moindre privilège et ne jamais committer de secret ;
- expliquer où intégrer la sécurité dans le cycle (SAST, dépendances, container) et lire une CVE.

Si tu coches tout, la suite logique de la roadmap t'attend : **Bloc 6 — Cloud Providers**, où ce réseau/sécurité s'applique à grande échelle (et où tes `security groups`, tes certificats et tes load balancers prennent vie).

---

*Démarre maintenant avec la **Leçon 1** (les protocoles de communication) dans `01-Protocoles-communication/`.*