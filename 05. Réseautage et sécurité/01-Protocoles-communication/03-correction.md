# Correction — Leçon 1 : Protocoles de communication

> **Bloc 5 · Leçon 1** — Correction de l'exercice, pas à pas.

---

## Étape 1 — DNS

```bash
dig +short api.github.com
# (ex.) 140.82.121.6
nslookup api.github.com
# Server: ...
# Address: 140.82.121.6
```

**Explication** : `dig` résout le nom de domaine en IP. C'est la **première** brique : si ici on n'obtient aucune réponse, le problème est DNS, pas le serveur applicatif.

---

## Étape 2 — Transport (port)

```bash
nc -zv api.github.com 443
# Connection to api.github.com (140.82.121.6) port 443 [tcp/https] succeeded!
nc -zv api.github.com 80
# Connection refused ou timeout selon les règles côté serveur
```

**Explication** : `nc -z` teste silencieusement si le port accepte une connexion TCP. Le succès du 443 confirme que l'IP et le port sont joignables depuis ton poste → la couche réseau/transport fonctionne.

---

## Étape 3 — Application (HTTP/HTTPS)

```bash
curl -I https://api.github.com/zen
# HTTP/2 200
# content-type: application/json
# ...
```

**Explication** : `-I` = requête `HEAD` (en-têtes sans le corps). Le `200` dit que **l'application répond**. On distingue bien ici les niveaux : étape 2 = la porte est ouverte, étape 3 = quelqu'un de l'autre côté répond « bonjour ».

---

## Étape 4 — Tes connexions locales

```bash
ss -tulpn
# Active Internet connections (only servers)
# tcp  LISTEN  0  511  0.0.0.0:22 ... sshd
# tcp  LISTEN  0  ...  127.0.0.1:5432 ... postgres
```

**Explication** : fiables, tu vois quel service écoute sur quel port et sur quelle interface (`0.0.0.0` = toutes les interfaces, `127.0.0.1` = local uniquement). S'il n'y a pas de service → le port reste guidé.

---

## Étape 5 — Questionnaire

**1. Différence entre l'étape 2 (port) et l'étape 3 (application) ?**
> Le port ouvert = un « guichet » existe (étape 2, niveau transport TCP). Mais le guichet peut exister sans que personne (application) réponde correctement derrière. L'étape 3 (HTTP) vérifie que le logiciel applicatif répond effectivement par un code `200`.

**2. `nc ... 80` échoue mais `curl https://api.github.com` fonctionne ?**
> Ça nous apprend que le port expose uniquement le service HTTPS (443/80 des fois refusé par le serveur) — le trafic est redirigé ou le port 80 est bloqué/filtré côté serveur. Le service se trouve bien sur 443. Le « port 80 » échoue souvent car GitHub ne sert pas en clair : il redirige vers HTTPS (port 443 restant).

---

## Checklist de validation (leçon 1)

- [ ] J'utilise `dig` / `nslookup` pour résoudre un nom en IP.
- [ ] Je teste un port avec `nc -zv` et je sais lire la réponse.
- [ ] Je vérifie le code HTTP avec `curl -I` / `-i`.
- [ ] Je liste les ports en écoute avec `ss -tulpn`.
- [ ] Je distingue les niveaux DNS / port / application dans un diagnostic.

---

## 🧠 Conseils pour la suite

- **Mémorise la cascade** : DNS → IP → route → firewall → port → service → application. Tu t'en serviras à chaque incident.
- Ne travaille jamais un protocole sensible via `telnet` en clair : remplace par `nc` / `openssl s_client`.
- **Garde ce fichier** comme référence : les commandes reviendront (Leçon 2 diagnostic, Leçon 4 TLS).