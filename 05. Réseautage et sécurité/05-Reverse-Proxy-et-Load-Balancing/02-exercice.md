# Exercice — Leçon 5 : Reverse Proxy et Load Balancing

> **Bloc 5 · Leçon 5** — Exercice à réaliser sur une **machine de test** (VM/WSL/VPS de test) — jamais en production.

---

## Contexte

Tu déploies la stack de la roadmap : un **frontend** (fichiers statiques) et un **backend** (service sur un port), avec Nginx comme **unique point d'entrée**. Tu vas construire, vérifier puis répartir la charge, étape par étape.

---

## Énoncé

> 📌 **Rappel des besoins** : `sudo` = exécuter en administrateur (sous Ubuntu, apt installe des paquets). `curl -s` = silencieux (sans barre de progression). `-I` = en-têtes seulement.

### Étape 1 — Installer Nginx
```bash
sudo apt update
sudo apt install nginx -y
```

### Étape 2 — Servir un frontend statique
- Crée un fichier `/tmp/mon-site/index.html` contenant `<h1>Bienvenue</h1>`.
- Configure Nginx pour le servir sur le port 80 (`root /tmp/mon-site;`).
- Recharge : `sudo nginx -t` puis `sudo systemctl reload nginx`.
- Vérifie : `curl -I http://localhost/` doit répondre `200`.

### Étape 3 — Ajouter le reverse proxy `/api`
- Lance un backend local de test : `python3 -m http.server 8080 --directory /tmp/mon-site` (dans un terminal).
- Ajoute à la config Nginx : `location /api/ { proxy_pass http://localhost:8080; }`.
- Recharge et teste : `curl http://localhost/api/` doit renvoyer le contenu du backend.

### Étape 4 — Load balancing avec `upstream`
- Lance un 2e backend sur le port `8081` (même commande avec `8081`).
- Ajoute un bloc `upstream backend { server localhost:8080; server localhost:8081; }`.
- Fais pointer `location /api/` vers `http://backend`.
- Répète `curl -s http://localhost/api/` plusieurs fois et observe la réponse.

### Étape 5 — Réflexion (`notes-exercice-05.md`)
- Pourquoi le proxy est-il le seul accès exposé ? (rappelle le principe vu en Leçon 3)
- Que se passe-t-il si tu coupes un des deux backends ? Teste-le et note.

---

## Livrable

`notes-exercice-05.md` avec : la config utilisée, les sorties `curl`, et les réponses.
La correction détaillée est dans **`03-correction.md`**.