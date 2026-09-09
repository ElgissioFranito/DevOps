# Référence rapide — Leçon 4 : TLS / openssl

> Bloc 5 · Leçon 4 — Aide-mémoire.

## Concepts
- **HTTPS** = HTTP + TLS.
- **Clé privée** (`.key`) : secrète, reste sur le serveur.
- **Certificat** (`.crt`/`.pem`) : clé publique + infos, signé par une CA.
- **CA** : autorité de certification de confiance (Let's Encrypt, DigiCert…).
- **Auto-signé** : cert généré soi-même, dans navigateur alerte.

## Commandes openssl

| Besoin | Commande |
|--------|----------|
| Générer une clé privée RSA 2048 | `openssl genrsa -out key.pem 2048` |
| Certificat auto-signé | `openssl req -new -x509 -key key.pem -out cert.pem -days 365 -subj "/CN=..."` |
| Inspecter un certificat | `openssl x509 -in cert.pem -text -noout` |
| Tester une connexion TLS | `openssl s_client -connect host:443` |
| Afficher le sujet d'un cert distant | `echo \| openssl s_client -connect host:443 2>/dev/null \| openssl x509 -noout -subject` |

## Erreurs fréquentes
- **Expiré** → regarder `notBefore/notAfter`.
- **Mismatch** → le CN/SAN ne correspond pas au domaine.
- **Auto-signé** → pas de CA publique (test/interne seulement).

## Bonnes pratiques
- Let's Encrypt / certbot (renouvellement auto).
- Désactiver TLS 1.0/1.1, favoriser TLS 1.2/1.3.
- Clé privée `chmod 600`, jamais dans Git.