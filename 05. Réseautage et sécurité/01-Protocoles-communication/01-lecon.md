# Leçon 1 — Protocoles de communication (TCP, UDP, HTTP, HTTPS, DNS, SSH)

> **Bloc 5 · Réseautage et sécurité** — Leçon 1 sur 8
> 🧭 **Bienvenue dans le bloc 5 !** (Si ce n'est pas déjà fait, lis d'abord `00-Introduction-Bloc.md`.) Tu sais déjà créer des commits (Bloc 4) et automatiser sur Linux (Blocs 2-3). Dans ce bloc, on répond à la question que tout DevOps se pose chaque jour : **comment deux machines « se parlent » ?** et **pourquoi une application est-elle accessible ou non ?** Cette leçon pose les fondations : les **protocoles**.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Expliquer** simplement ce qu'est un protocole réseau et à quoi sert le modèle client-serveur.
2. **Distinguer** TCP (fiable, ordonné) et UDP (rapide, pas de garantie), et savoir quand utiliser l'un ou l'autre.
3. **Comparer** HTTP et HTTPS, et connaître la différence HTTP/1.1 vs HTTP/2.
4. **Expliquer** le rôle du DNS (transformer un nom en adresse IP) et celui de SSH (connexion chiffrée à distance).
5. **Tester** concrètement avec `curl` (HTTP/HTTPS), `dig`/`nslookup` (DNS), `nc`/`telnet` (port) et comprendre le rôle des **ports** (22, 80, 443, 5432…).

---

## 2. Explication simple

### 2.1 Qu'est-ce qu'un protocole ? (le « pourquoi »)

Un **protocole réseau**, c'est un **ensemble de règles** que deux machines acceptent de suivre pour se comprendre. Comme deux personnes qui conviennent d'une langue commune pour discuter : sans accord, pas de dialogue.

> 💡 **Analogie** : Internet, un grand réseau de coursiers. Un **protocole**, c'est la façon dont les coursiers s'accordent pour emballer, adresser et livrer les colis. TCP et UDP sont deux « méthodes de livraison » ; DNS est l'**annuaire** ; HTTP est « le contenu » du colis.

Chaque transfert suit le modèle **client-serveur** :

```
[ Client ]  ── demande ──>  [ Serveur ]
   ^                            │
   └──────── réponse ──────────┘
```

- Le **client** (ton navigateur, `curl`, une app Angular) **demande**.
- Le **serveur** (Nginx, Spring Boot, une API) **répond**.

### 2.2 Le « comment » : couches, ports et protocoles de transport

On n'embrasse pas le réseau d'un bloc. On le découpe en **couches**. Pour le DevOps, retiens les deux qui te servent tous les jours :

| Couche | Rôle | Exemples |
|--------|------|----------|
| **Transport** | Comment livrer le paquet de machine à machine (fiable ou non) | TCP, UDP |
| **Application** | Comment deux logiciels se parlent (le format des messages) | HTTP, HTTPS, DNS, SSH |

Chaque service écoute sur un **port** (un « numéro de guichet »). L'adresse IP donne la machine ; le port donne le guichet de la machine.

| Port | Service habituel |
|------|------------------|
| 22 | SSH |
| 80 | HTTP |
| 443 | HTTPS |
| 5432 | PostgreSQL |
| 27017 | MongoDB |

### 2.3 TCP vs UDP (le « pourquoi / quand »)

**TCP** (Transmission Control Protocol) = livraison **fiable et ordonnée**. Il établit une connexion (« poignée de main »), renumérote les paquets, et renvoie ce qui est perdu.

> 💡 Comparaison : TCP c'est une **lettre recommandée avec accusé de réception** — si le destinataire ou la page arrive, tout arrive, dans l'ordre. Parfait pour les fichiers, les pages web, les API.

**UDP** (User Datagram Protocol) = livraison **rapide sans garantie**. Ce n'est pas un service postal mais un **lancer de balles** : on envoie, on n'attend pas, on ne renvoie pas ce qui est perdu.

> 💡 Comparaison : UDP c'est une **discussion en direct** (appel vidéo, voix, jeu en ligne) : si un morceau se perd, inutile de le renvoyer, on continue.

### 2.4 HTTP / HTTPS (le contexte des applications web)

**HTTP** = le protocole de la plupart des applications web. Une requête porte :
- une **méthode** : `GET` (lire), `POST` (créer), `PUT`, `DELETE`…
- un **URL/chemin** : `GET /api/users`
- des **headers**, un corps éventuel (ex. des données JSON).

**HTTPS** = HTTP **chiffré** via une couche **TLS**. Il apporte **confidentialité** (personne n'espionne), **authentification** (le serveur prouve son identité par un **certificat**) et **intégrité** (rien n'a été falsifié). On y reviendra en détail à la Leçon 4.

> ⚠️ En production, **HTTP seul est inacceptable** dès qu'il y a login, mot de passe, ou clé API : il faut HTTPS.

### 2.5 DNS et SSH (deux « protocoles-services »)

- **DNS** (Domain Name System) : transforme un nom lisible `api.example.com` en adresse IP `192.168.x.x`, comme un annuaire téléphonique. Avant toute connexion, DNS résout le nom.
- **SSH** : protocole **chiffré et authentifié** pour administrer un serveur à distance (déjà vu au Bloc 2). Port 22.

> 🔵 **À mentionner, définir, ne pas creuser — SMTP** : protocole d'envoi de courriels. On le croise pour configurer des notifications/alertes (ex. un pipeline CI/CD qui envoie un e-mail en cas d'échec).

### Le « quand » :

| Situation | Protocole / Outil |
|-----------|-------------------|
| Charger une page web ou appeler une API | HTTP / HTTPS |
| Voir si un service répond sur un port | `nc`, `telnet`, `curl` |
| Résoudre un nom en IP | DNS (`dig`, `nslookup`) |
| Administrer un serveur à distance | SSH |
| Voix/jeu (tolérants aux pertes) | UDP |

---

## 📖 Mini-glossaire (à consulter avant les exemples)

> Définitions d'une ligne pour ne jamais être perdu(e). Un terme surligné dans le reste de la leçon est expliqué ici.

- **Paquet** : petit bloc de données découpé et envoyé sur le réseau (le « contenu » quand il circule).
- **Adresse IP** : numéro unique qui identifie une machine sur un réseau (détaillé à la Leçon 2).
- **Port** : numéro de « guichet » d'un service sur une machine (22=SSH, 80=HTTP, 443=HTTPS…).
- **Header (en-tête)** : les métadonnées d'une requête/réponse HTTP (méthode, type de contenu, date…), distinctes du corps du message.
- **API** : interface qui permet à un programme de parler à un autre (ex. `api.github.com`). « Appeler une API » = envoyer une requête à ce service.
- **JSON** : format texte pour structurer des données, ex. `{"nom":"devops"}` (vu au Bloc 3).
- **localhost** : un nom spécial qui désigne **ta propre machine** (équivaut à l'IP `127.0.0.1`).
- **GET / POST** : deux méthodes HTTP. `GET` = « récupérer », `POST` = « envoyer/déposer » quelque chose.
- **Code `404`** : réponse HTTP signifiant « ressource introuvable » (page/url pas trouvée).
- **Timeout** : délai maximal d'attente dépassé sans réponse → la connexion est probablement bloquée.
- **Latence** : temps de réponse d'un serveur (plus il est bas, mieux c'est).
- **Mode verbose** : mode « bavard » (`-v`), affiche davantage de détails.
- **Nginx / HAProxy / Spring Boot / Angular** : serveurs web ou frameworks utilisés en exemple dans ce bloc (détaillés à la Leçon 5 et dans les blocs suivants).

---

### 🧪 À faire maintenant (5 min) — ta première série de commandes réseau

> Objectif : voir une requête HTTP « en vrai » avant toute théorie. Copie-colle ces commandes dans ton terminal :

```bash
curl -I https://example.com      # affiche les en-têtes (méthode + code HTTP)
# HTTP/2 200
# content-type: text/html; charset=UTF-8   <-- "content-type" dit le format

dig +short example.com           # l'IP derrière le nom example.com
# 93.184.215.14

nc -zv example.com 443           # le port 443 (HTTPS) est-il joignable ?
# Connection ... port 443 ... succeeded!
```

**Ce que tu dois observer / écrire dans ta tête** :
- `curl -I` → le `200` = l'app répond ; `content-type` = le format.
- `dig` → le **nom de domaine est converti en IP** (c'est le DNS).
- `nc -zv` → **un port accepte une connexion TCP** (c'est le réseau/transport).

Ces trois commandes sont le réflexe n°1 de tout DevOps. Elles te resserviront dans chaque leçon du bloc.

---

## 📖 Vocabulaire / Abréviations

| Terme | Définition (une ligne) |
|---|---|
| **Protocole** | ensemble de règles pour que deux machines se comprennent |
| **TCP** | transport fiable (connexion, accusés de réception) — web, SSH |
| **UDP** | transport rapide sans garantie — DNS, VPN (Leçon 5) |
| **Port** | numéro (1-65535) désignant le « guichet » d'un service sur une machine |
| **HTTP / HTTPS** | protocole du web / sa version chiffrée (Leçon 4) |
| **DNS** | l'annuaire : transforme un nom (`exemple.com`) en adresse IP |
| **SSH** | connexion à distance sécurisée (Bloc 2, Leçon 5) |
| **Paquet (packet)** | petit bloc de données qui circule sur le réseau |

---

## 3. Exemples concrets

### 3.1 HTTP avec `curl`

```bash
# Méthode GET : récupérer la réponse d'une API publique
curl -i https://api.github.com/zen
#   -i = affiche les headers de la réponse

# POST : envoyer des données (JSON)
curl -X POST https://httpbin.org/post \
     -H "Content-Type: application/json" \
     -d '{"name":"devops","role":"learner"}'
```
### 3.2 Vérifier qu'un port est ouvert (TCP)

```bash
# Ouvrir une connexion TCP sur un port et voir si ça répond
nc -zv example.com 443        # verbose "success"
telnet example.com 443         # si connecte = port ouvert (sinon connexion refusée)
```

### 3.3 Résoudre un nom avec DNS

```bash
dig +short example.com      # -> l'IP
nslookup example.com        # -> plus verbeux
```

### 3.4 Voir tes propres connexions

```bash
ss -tulpn   # liste des ports/listen/ports TCP (t) et UDP (u)
curl -I https://httpbin.org   # affiche les headers de la réponse web
```

---

## 4. Bonnes pratiques modernes (2025-2026)

- **HTTPS partout** : plus de 90 % du web passe par HTTPS ; en DevOps, on ne déploie pas d'HTTP nu en clair pour du login.
- **HTTP/2** par défaut : plus rapide que HTTP/1.1 sur les serveurs modernes ; les fichiers de conf Nginx/HAProxy activent souvent HTTP/2 pour les clients terminaux.
- **N'exposer que le minimum** (moindre surface) : une base de données (5432) ne doit **jamais** être ouverte en public sur 80/443.
- **Ne jamais conserver une adresse IP en dur** dans un fichier de configuration : passer par DNS + port.
- **Tester avant de déployer** : `nc -zv` et `curl` sont les réflexes de premier niveau avant de regarder les logs applicatifs.

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi c'est dangereux/inefficace | ✅ Version correcte |
|----------------|-------------------------------------|---------------------|
| `curl http://...` pour transmettre un login | Les données passent **en clair**, espionnables. | Toujours `https://` pour toute donnée sensible. |
| Croire qu'HTTP fonctionne sans connaître ports/chemin | On voit « connexion refusée » ou « 404 » sans comprendre pourquoi. | Vérifier en cascade : DNS → IP → route → firewall → port → service. |
| Utiliser TCP pour un streaming vidéo temps réel | Latence + surcoût inutilisables. | UDP pour la vidéo/voix (tolérant aux pertes). |
| Tester avec `telnet` une connexion sensible | `telnet` envoie en clair, pas chiffré. | Utiliser `nc` ou `openssl s_client`, sinon `ssh`. |
| Mettre des IP en dur dans un fichier | Réglage cassé au moindre changement d'IP. | Passer par le nom DNS + le port. |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`**, la correction commentée dans **`03-correction.md`**. Lis bien cette leçon avant de t'y mettre.

**Énoncé court** : comme un DevOps qui diagnostique, interroge une API publique avec `curl -I`, vérifie que le port 443 de ton site favori est ouvert avec `nc -zv`, résous un nom de domaine en IP avec `dig`/`nslookup`, et identifie les ports en écoute sur ta machine avec `ss -tulpn`. Note chaque observation dans `notes-exercice-01.md`.

---

## 7. Correction détaillée de l'exercice

> La correction complète pas-à-pas est dans **`03-correction.md`**. Essentiel du raisonnement :
> - on **choisit le bon outil pour le bon niveau** : `curl` (niveau application), `nc` (niveau transport/port), `dig` (niveau du nom → IP) ;
> - on **hiérarchise** les tests : DNS → IP → port → service, comme dans la vraie vie ;
> - on **déduit** d'un retour (refus, timeout, 404) quelle brique est en cause.

---

## 8. Checklist de validation

- [ ] Je peux expliquer ce qu'est un protocole et la différence client/serveur.
- [ ] Je distingue TCP et UDP et je sais dans quel cas utiliser l'un ou l'autre.
- [ ] Je compare HTTP vs HTTPS et je cite les méthodes `GET`/`POST`.
- [ ] Je comprends le rôle du DNS et de SSH.
- [ ] Je sais utiliser `curl`, `nc`, `dig`/`nslookup` et `ss` pour tester un service.
- [ ] Je connais les ports standards (22, 80, 443, 5432).

---

🧭 **Pont vers la suite** — Tu sais de quoi parlent les machines. Mais pour diagnostiquer un problème réel (« je n'arrive pas à joindre le serveur »), le protocole seul ne suffit pas : il faut **trouver l'adresse d'une machine et tracer le chemin**. C'est l'objet de la Leçon 2 : l'**adressage IP et le diagnostic réseau**.

---

*Prochaine étape :* Leçon 2 — **Adressage IP et diagnostic réseau** dans `02-Adressage-IP-et-diagnostic/`.