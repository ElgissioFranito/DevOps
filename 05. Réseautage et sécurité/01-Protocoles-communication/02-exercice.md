# Exercice — Leçon 1 : Protocoles de communication

> **Bloc 5 · Leçon 1** — Exercice à faire en autonomie.
> **Objectif** : te mettre dans la peau d'un DevOps qui vérifie que « le service répond » avant de chercher dans les logs. Tu vas manipuler `curl`, `nc`, `dig` et `ss` sur ta propre machine (aucun accès au réseau d'entreprise requis — utilise des services publics Neutra).

---

## Contexte

Tu as reçu un ticket : *« Je n'arrive pas à joindre notre API depuis mon poste. »* Avant de paniquer, un DevOps méthodique commence par vérifier la **chaîne** : DNS → IP → port → service. Tu vas reproduire cette chaîne de tests avec des API publiques inoffensives.

---

## Énoncé

> 📌 **Options utilisées (à retenir)** : `+short` = n'afficher que l'IP, sans détail ; `nc -z` = tester sans envoyer de données (`-v` = afficher le résultat) ; `curl -I` = ne récupérer que les **en-têtes** ; `ss -tulpn` / `netstat -tulpn` = lister les ports **en écoute** et le programme associé.

### Étape 1 — Vérifier la couche DNS
- Résous `api.github.com` en IP avec `dig +short api.github.com`.
- Résous-le de façon plus verbeuse avec `nslookup api.github.com`.
- Note l'IP obtenue dans ton fichier `notes-exercice-01.md`.

### Étape 2 — Vérifier la couche transport (port)
- Vérifie que le port `443` est ouvert avec `nc -zv api.github.com 443`.
- Teste le port `80` (HTTP) : `nc -zv api.github.com 80`.

### Étape 3 — Vérifier la couche application (HTTP/HTTPS)
- Récupère les headers : `curl -I https://api.github.com/zen`.
- Identifie la méthode, le code de statut (ex. `200`), et le header `content-type`.

### Étape 4 — Observer tes propres connexions
- Liste les ports en écoute sur ta machine : `ss -tulpn` (ou `netstat -tulpn`).
- Note dans le fichier : quelles technologies (ports) sont couramment en écoute chez toi (ex. 5432 si PostgreSQL, 22 si SSH).

### Étape 5 — Réflexion (questionnaire)
Dans `notes-exercice-01.md`, réponds en 2-3 phrases à :
1. Quelle est la différence entre l'étape 2 (port) et l'étape 3 (application) ?
2. Si `nc -zv api.github.com 80` échoue mais que `curl https://api.github.com` fonctionne, qu'est-ce que ça t'apprend ?

---

## Livrable

Un fichier `notes-exercice-01.md` avec :
- les sorties des commandes,
- les réponses aux 2 questions de l'étape 5.

La correction structurée est dans **`03-correction.md`** : compare avant de continuer.