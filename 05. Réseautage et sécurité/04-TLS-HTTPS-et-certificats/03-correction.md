# Correction — Leçon 4 : TLS, HTTPS et certificats

> **Bloc 5 · Leçon 4** — Correction pas à pas.

---

## Étape 1 — Génération

```bash
openssl genrsa -out monkey.pem 2048        # clé privée RSA 2048
openssl req -new -x509 -key monkey.pem -out moncert.pem -days 365 -subj "/CN=demo-local"
```

**Explication** : `genrsa` produit la clé privée ; `req -new -x509` crée directement un certificat auto-signé (self-signed) pour 1 an. `/CN=demo-local` = le « nom » du sujet. Sans CA publique, c'est un auto-signé.

## Étape 2 — Inspection

```bash
openssl x509 -in moncert.pem -text -noout
# Subject: CN = demo-local
# ... Not Before / Not After ...
# Signature Algorithm: ...
```

**Explication** : le `subject`, la **période de validité**, et l'algorithme sont la première chose à regarder pour comprendre une erreur de certificat. Un `notAfter` dépassé = certificat expiré.

## Étape 3 — Test d'une connexion TLS publique

```bash
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -subject
openssl s_client -connect example.com:443 -brief
```

**Explication** : `openssl s_client` ouvre une connexion TLS côté client et affiche le certificat envoyé par le serveur. Ce cert est signé par une CA publique → les navigateurs le font confiance.

## Étape 4 — Réflexion

**1. Pourquoi les navigateurs n'aiment-ils pas l'auto-signé ?**
> Aucune **autorité de certification (CA)** reconnue n'a signé ce certificat : le navigateur ne peut pas vérifier que le serveur est bien « quelqu'un de sûr ». Il l'accepte seulement en l'ajoutant explicitement aux exceptions.

**2. Certificat expiré vs hostname mismatch ?**
> - **Expiré** : la date de validité (`notAfter`) est dépassée — le certificat n'est plus valide.
> - **mismatch** : le **domaine** dans le certificat ne correspond pas à celui du site (ex. cert pour `x.com`, accès via `y.com`).

---

## Checklist de validation (leçon 4)

- [ ] J'explique HTTPS/TLS et ses 3 garanties.
- [ ] Je génère une clé + un certificat auto-signé avec `openssl`.
- [ ] J'inspecte dates, sujet, émetteur d'un certificat.
- [ ] Je teste une connexion TLS avec `openssl s_client`.
- [ ] Je distingue auto-signé / CA publique / expiré / mismatch.

---

## 🧠 Conseils pour la suite

- En prod, utilise **Let's Encrypt / certbot** (renouvellement auto, 90 jours).
- La clé privée **ne se commit jamais** (Bloc 6 reviendra sur les secrets).
- En Leçon 6, tu verras où placer ce certificat (chiffrement terminé au niveau du reverse proxy) ; en Leçon 5, tu verras le VPN, qui chiffre lui aussi — mais tout le trafic, pas seulement le web.