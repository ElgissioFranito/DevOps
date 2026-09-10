# Correction — Leçon 5 : Bases de données managées (RDS)

> **Bloc 6 · Leçon 5** — Correction pas à pas.

---

## Étape 1 — Connexion locale

```bash
sudo apt update && sudo apt install -y postgresql   # installe PostgreSQL
sudo systemctl start postgresql                     # démarre le service
ss -tulpn | grep 5432   # affiche une ligne avec "5432" si PostgreSQL écoute
sudo -u postgres psql -h localhost -p 5432          # se connecte (utilisateur postgres)
```
**Explication** : `systemctl start` lance le service (notion de service du Bloc 2) ; `ss -tulpn | grep 5432` vérifie que le **port 5432** est ouvert (le guichet de PostgreSQL) ; `psql -h localhost -p 5432` se connecte avec `psql` (le client) sur la machine même (`localhost`) au port 5432. Dans `psql`, `\l` liste les bases, `\q` quitte. Les notions **host/port/utilisateur** sont exactement celles utilisées avec RDS.

## Étape 2 — Ce que « managé » change

**Le cloud fait (avec RDS)** :
1. L'installation et les mises à jour du moteur (PostgreSQL) ;
2. Les sauvegardes automatiques (+ point-in-time recovery) ;
3. Le remplacement de disque / la haute disponibilité (réplica, failover).

**Reste à toi** :
1. La **modélisation** (tables, relations, index) et les migrations SQL ;
2. Les **requêtes** et la performance applicative (où les index manquent ?).

Autrement dit : le cloud gère le **conteneur**, tu gères le **contenu**.

## Étape 3 — Schéma + sécurité

```
Internet
   ↓
Load Balancer (public)
   ↓
Application Spring Boot (privé) → SEULE elle parle à la base
   ↓
RDS PostgreSQL (privé, jamais public)
```

3 règles de sécurité :
1. **Subnet privé** — aucun accès Internet direct à la base ;
2. **Security group restreint** à l'application (port 5432 uniquement vers l'IP de l'app) ;
3. **Credentials dans un coffre de secrets** (Secret Manager / Vault) et **jamais** dans Git, ni en clair dans le code.

---

## Checklist de validation (leçon 5)

- [ ] J'explique une base « managée » et ce que le cloud gère.
- [ ] Je définis RDS, moteur, instance, endpoint, port, backup, réplica, failover.
- [ ] Je sais pourquoi une base ne doit jamais être publique.
- [ ] Je connecte une base avec `psql` (local, puis RDS plus tard).
- [ ] Je pratique backups + point-in-time recovery (au moins un test).
- [ ] Je place les identifiants dans un coffre, jamais dans Git.

---

## 🧠 Conseils pour la suite

- **Règle d'or** : base = jamais publique ; les données sont le trésor.
- Un backup qu'on **n'a jamais testé** n'existe pas : teste la restauration.
- La Leçon 6 (IAM) va te donner les **clés d'accès** pour enfin piloter AWS (et créer tes propres bases/buckets) ; les credentials de la **base** et les **clés AWS** sont deux choses différentes — à ne pas mélanger.