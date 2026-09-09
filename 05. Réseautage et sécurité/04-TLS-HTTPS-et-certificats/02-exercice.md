# Exercice — Leçon 4 : TLS, HTTPS et certificats

> **Bloc 5 · Leçon 4** — Exercice à faire en autonomie, tout local (pas besoin de machine distante).

---

## Contexte

Tu vois souvent l'avertissement « votre connexion n'est pas privée » sur un site. But de l'exercice : **démystifier les certificats** en générant, inspectant et testant toi-même, pour savoir reagir à une erreur TLS.

---

## Énoncé

> 📌 **Options utilisées** : `-out` = fichier de sortie ; `-key` = la clé utilisée ; `-new -x509` = créer un nouveau certificat ; `-days 365` = validité 1 an ; `-subj` = les infos du sujet ; `-text -noout` = afficher le détail sans réécrire le fichier ; `s_client -connect` = test client TLS ; `-brief` = sortie courte ; `2>/dev/null` = masquer les messages d'erreur.

### Étape 1 — Générer un certificat auto-signé
```bash
openssl genrsa -out monkey.pem 2048
openssl req -new -x509 -key monkey.pem -out moncert.pem -days 365 -subj "/CN=demo-local"
```

### Étape 2 — Inspecter le certificat
```bash
openssl x509 -in moncert.pem -text -noout
```
Relève : le **subject** (CN), la **période de validité** (notBefore / notAfter), et l'**algorithme** de signature.

### Étape 3 — Tester une connexion TLS publique
```bash
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -subject
openssl s_client -connect example.com:443 -brief
```
Note le sujet du certificat réel reçu.

### Étape 4 — Réflexion (dans `notes-exercice-04.md`)
1. Pourquoi les navigateurs n'aiment-ils pas un certificat auto-signé (en gros) ?
2. Quelle est la différence entre « certificat expiré » et « hostname mismatch » ?

---

## Livrable
`notes-exercice-04.md` avec : sorties, subject/dates du cert local, sujet du cert public, et les 2 réponses.

Correction détaillée dans **`03-correction.md`**.