# Exercice — Leçon 3 : Pare-feu et contrôle des flux

> **Bloc 5 · Leçon 3** — Exercice à réaliser sur une **machine de test** (VM ou WSL ou VPS de test) — **jamais** sur une machine sensible.

---

## Contexte

Tu viens d'installer une mini-application web (ex. un service sur le port 8080) sur ton serveur de test. Tu dois sécuriser ce serveur : n'exposer que ce qu'il faut, laisser l'application joignable, et bloquer le reste.

---

## Énoncé

> 📌 **Options utilisées** : `sudo` = exécuter en administrateur ; `ufw status numbered` = lister les règles avec un numéro ; `ss -tulpn` = lister les ports en écoute ; `nc -zv` = tester la connexion à un port (voir Leçon 1).

1. **Installe et prépare UFW** (si pas déjà présent) :
   - `sudo apt install ufw`
   - Règle le défaut `deny incoming` et `allow outgoing`.
2. **Ouvre uniquement** :
   - SSH sur `22/tcp`,
   - HTTP sur `80/tcp`,
   - ton application sur `8080/tcp` (pour l'exercice, en adéquation avec ta config),
   - `443/tcp` (HTTPS) même si tu n'as pas encore de certificat.
3. **Bloque** la base PostgreSQL `5432/tcp` (elle doit rester accessible seulement interne).
4. **Active** le pare-feu : `sudo ufw enable`.
5. **Vérifie** :
   - `sudo ufw status numbered` : toutes tes règles doivent apparaître dans le bon ordre.
   - `ss -tulpn` : liste les ports réels.
6. **Teste** depuis un autre terminal (ou une autre machine du même réseau) :
   - `nc -zv <ip> 8080` doit être **accessible**.
   - `nc -zv <ip> 5432` doit être **bloqué/filtré**.
7. **Réflexion** : écris dans `notes-exercice-03.md` la réponse à :
   - Pourquoi devoir autoriser `22` **avant** d'activer le pare-feu ?
   - Quelle est la réflexion qui guide ton choix de ports autorisés (principe) ?

---

## Livrable
`notes-exercice-03.md` avec les sorties de `ufw status`, `ss -tulpn`, et les réponses au questionnement.

Comparer à `03-correction.md`.**