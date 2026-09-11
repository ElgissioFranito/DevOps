# Leçon 4 — TLS, HTTPS et certificats

> **Bloc 5 · Réseautage et sécurité** — Leçon 4 sur 8
> 🧭 **Pont depuis les Leçons 1-3** : tu sais faire circuler les données (protocoles), adresser et filtrer (pare-feu). Dernière brique pour que « app accessible » rime avec **sécurisé** : le **chiffrement TLS/HTTPS** et les **certificats**, que tu manipuleras concrètement avec **`openssl`**. On ne « subit » plus un certificat : on l'inspecte, le génère, le comprend.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Expliquer** pourquoi HTTP doit devenir HTTPS et ce qu'apporte TLS (confidentialité, authentification, intégrité).
2. **Définir** une clé privée, un certificat, une autorité de certification (CA), et la notion de certificat auto-signé vs signé.
3. **Manipuler** les outils `openssl` de base : générer une clé, un certificat auto-signé, inspecter un certificat, tester une connexion TLS.
4. **Diagnostiquer** une erreur TLS courante (« certificat expiré », « hostname mismatch », « self-signed »).
5. **Comprendre** la chaîne de confiance : le rôle des certificats publics via Let's Encrypt / une CA (survol).

---

## 2. Explication simple

### 2.1 Le « pourquoi » : pourquoi chiffrer ?

Sur HTTP en clair, tout ce qui passe (login, token, données) peut être **lu** par quiconque est entre toi et le serveur (un réseau public, un routeur compromis). Chiffrer (HTTPS) répond à trois besoins :

1. **Confidentialité** : personne ne peut lire le trafic.
2. **Authentification** : le serveur prouve qu'il est bien qui il prétend être.
3. **Intégrité** : les données n'ont pas été modifiées en route.

> 💡 **Analogie** : envoyer en HTTP, c'est une **carte postale** lisible par tous. HTTPS, c'est une **lettre fermée**, vérifiable de la bonne personne, et s'assurer que le paquet n'a pas été ouvert en route.

### 2.2 Le « comment » : clé privée, certificat, CA

Pour chiffrer de façon **asymétrique**, chaque serveur possède une **clé privée** (secrète) et un **certificat** qui contient sa clé **publique** + ses infos (domaine, dates, émetteur).

- **Clé privée** : reste secrète sur le serveur (`server.key`).
- **Certificat** (`server.crt` / `.pem`) : livré au visiteur pour lui permettre de vérifier.

Le certificat est **signé** par une **autorité de certification (CA)** de confiance (Let's Encrypt, Sectigo, DigiCert…). Ton navigateur a la liste des CA de confiance embarquée ; si le certificat est signé par une de ces CA pour ce domaine, il est accepté.

> 🔒 **Certificat auto-signé** : généré par toi-même, pas par une CA publique. Utile en test/entreprise interne, mais les navigateurs affichent un avertissement (« non fiable ») car aucune CA reconnue ne l'a signé.

### 2.3 Le « quand » et les erreurs TLS fréquentes

| Signe | Cause probable | Réaction |
|-------|----------------|----------|
| « votre connexion n'est pas privée » / « certificat expiré » | Certificat échu (sa date de validité est dépassée) | Inspecter les dates avec `openssl x509`. |
| « certificat auto-signé » | Cert sans CA reconnue | En interne, utiliser tout de même un vrai certificat (et idéalement une CA) ; en prod, une CA publique. |
| « hostname mismatch » | Certificat ne correspond pas au domaine | Générer un cert pour le bon domaine/SAN |

---

## 📖 Mini-glossaire (à consulter avant les exemples)

> Définitions d'une ligne pour ne jamais être perdu(e).

- **Chiffrement** : transformer des données en code illisible sans la bonne clé.
- **TLS** (Transport Layer Security) : le protocole moderne qui chiffre les échanges (HTTPS = HTTP + TLS). Anciennement appelé **SSL**.
- **Confidentialité / Authentification / Intégrité** : ne pas être lisible / prouver son identité / ne pas être modifié en route.
- **Chiffrement asymétrique** : système à **deux clés** liées (une privée, une publique) : on chiffre avec l'une, on déchiffre avec l'autre.
- **Clé privée** (`.key`) : le secret qui reste **sur le serveur**, jamais révélé ni committé.
- **Certificat** (`.crt`/`.pem`) : « carte d'identité numérique » du serveur, contenant sa clé publique + ses infos, signée par une CA.
- **CA** (Certificate Authority, autorité de certification) : organisme de confiance qui signe les certificats publics (Let's Encrypt, DigiCert…).
- **Auto-signé** : certificat que tu génères toi-même ; aucune CA ne l'a signé → le navigateur se méfie (test/interne seulement).
- **RSA / ed25519** : algorithmes de chiffrement pour générer des clés (RSA 2048, ed25519 — moderne et rapide).
- **openssl** : outil en ligne de commande de référence pour les clés ET certificats.
- **x509** : le standard/format de certificat le plus courant.
- **s_client** : sous-commande d'openssl pour **tester une connexion TLS** côté client.
- **CN / subject / notBefore / notAfter** : champs d'un certificat : le nom (CN), le sujet, la date de début (notBefore) et de fin (notAfter) de validité.
- **SAN** (Subject Alternative Name) : la liste des domaines couverts par un certificat.
- **Hostname mismatch** : le domaine demandé ne correspond pas à ceux du certificat → refus.
- **Let's Encrypt / certbot** : service gratuit qui délivre et **renouvelle automatiquement** des certificats publics valables 90 jours.

---

### 🧪 À faire maintenant (5 min) — générer et inspecter un vrai certificat

> Objectif : ne plus jamais « subir » un certificat. Tout se fait localement, sans réseau.

```bash
# 1) génère une clé privée + un certificat auto-signé (pour 1 an)
openssl genrsa -out ma-cle.pem 2048
openssl req -new -x509 -key ma-cle.pem -out mon-cert.pem -days 365 -subj "/CN=test-local"

# 2) inspecte-le : sujet, dates de validité, algorithme
openssl x509 -in mon-cert.pem -text -noout | head -30
```

**Ce que tu dois observer / écrire dans ta tête** :
- Le champ `Subject: CN = test-local` → à qui appartient le cert.
- `Not Before` / `Not After` → la **période de validité** (essentielle : c'est là qu'un « certificat expiré » se voit).
- `Signature Algorithm` → l'algorithme de signature.

**Mini-observation** : relance la même commande sur un site réel pour voir un certificat **signé par une CA** (reconnait : `issuer: ... Let's Encrypt ...`) :
```bash
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -issuer -subject
```

---

## 3. Exemples concrets (openssl)

### 3.1 Inspecter un certificat existant

```bash
openssl x509 -in moncerts.pem -text -noout | head -40
```

### 3.2 Générer un certificat auto-signé (test)

```bash
# 1. clé privée RSA 2048
openssl genrsa -out key.pem 2048
# 2. certificat auto-signé (avec infos)
openssl req -new -x509 -key key.pem -out cert.pem -days 365 -subj "/CN=mon-site.local"
# 3. inspecter
openssl x509 -in cert.pem -text -noout
```

### 3.3 Tester une connexion TLS depuis le client

```bash
openssl s_client -connect example.com:443 -showcerts
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -subject
```
---

## 4. Bonnes pratiques modernes (2025-2026)

- **HTTPS partout, même en interne** dès qu'on approche la production.
- **Certificats gratuits et automatiques avec Let's Encrypt** (via `certbot` ou les solutions de ton hébergeur) : renouvellement automatique, validité 90 jours.
- **Ne jamais ouvrir une clé privée** : fichiers `.key` en `600`, jamais dans Git, jamais dans un log.
- **Vérifier les dates et le domaine** avec `openssl` avant de redémarrer un service.
- **TLS 1.2/1.3** par défaut ; désactiver TLS 1.0/1.1 (vulnérables) et SSL.
- **Utiliser ed25519/RSA 2048+** ; éviter les clés faibles (512/1024 bits, dépréciées).

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi | ✅ Version correcte |
|----------------|----------|---------------------|
| Certificat auto-signé en production | Navigateurs refusent / alertent l'utilisateur. | Une CA publique (Let's Encrypt). |
| Ne pas renouveler un certificat | « Certificat expiré » = interruption de service. | Renouvellement automatique (certbot). |
| Mettre la clé privée dans Git | Attaque par vol de clé = chiffrement inutile. | `chmod 600`, jamais versionné. |
| Utiliser un certificat pour le mauvais domaine | mismatch → la connexion échoue. | Générer avec le bon domaine (SAN). |
| Désactiver l'HTTPS « pour tester » en prod | Expose en clair. | Tester en préprod, garder HTTPS en prod. |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`**, la correction dans **`03-correction.md`**.

**Énoncé court** : génère une clé RSA 2048 avec `openssl genrsa`, crée un certificat auto-signé `/CN=demo-local`, inspecte-le (`subject`, dates, algorithmes), teste `openssl s_client -connect <site>:443`, et note comment réagir à un « certificat expiré » vs « auto-signé ».

---

## 7. Correction détaillée de l'exercice

> La correction complète pas-à-pas est dans **`03-correction.md`**. On observe que : générer est trivial avec `openssl`, inspecter donne la cause d'une erreur, tester depuis le client confirme la chaîne. Les certificats publics (Let's Encrypt) t'apportent la confiance du navigateur.

---

## 8. Checklist de validation

- [ ] J'explique pourquoi HTTPS et ce qu'apporte TLS.
- [ ] Je génère une clé et un certificat auto-signé avec `openssl`.
- [ ] J'inspecte un certificat (dates, sujet, émetteur) avec `openssl x509`.
- [ ] Je teste une connexion TLS avec `openssl s_client`.
- [ ] Je distingue auto-signé vs signé par une CA publique, et certificat expiré / mismatch.
- [ ] Je connais le rôle de Let's Encrypt/certbot (renouvellement automatique).

---

🧭 **Pont vers la suite** — Le trafic web est maintenant **chiffré** (HTTPS). Mais chiffrer une session web, ce n'est pas encore **relier deux réseaux en privé** : pour administrer une machine distante, joindre un service interne ou te protéger sur un Wi-Fi public, il faut un **tunnel VPN**. C'est la Leçon 5 (WireGuard & OpenVPN). Ensuite seulement, avec plusieurs services (front Angular, backend Spring Boot), on placera un **reverse proxy / load balancer** comme point d'entrée unique — c'est la Leçon 6.

---

*Prochaine étape :* Leçon 5 — **VPN et tunnels sécurisés (WireGuard & OpenVPN)** dans `05-VPN-et-tunnels-securises`.