# Référence rapide — Leçon 5 : Bases de données managées (RDS)

> Bloc 6 · Leçon 5 — Aide-mémoire.

## Concepts
- **RDS** (Relational Database Service) : bases relationnelles **managées** d'AWS.
- **Managé** = le cloud gère install, mises à jour, backups, disques, haute dispo.
- **IaaS vs PaaS** : RDS est plutôt un service **managé** (proche PaaS) : tu fournis le moteur + la config, AWS le reste.
- **Moteur** : PostgreSQL, MySQL, MariaDB…
- **Endpoint** : adresse de connexion (host) ; **port** : 5432 (PostgreSQL) / 3306 (MySQL).
- **Réplica** : copie en lecture pour répartir / secourir (failover).
- **Point-in-time recovery** : restauration à un instant précis.

## Règle d'or
La base **ne doit jamais être accessible depuis Internet** :
```
Application (privé) → RDS PostgreSQL (privé)
```
- Subnet **privé**.
- Security group restreint à l'application (port 5432 uniquement).
- Credentials dans un **coffre de secrets** (Jamais dans Git).

## Connexion
```bash
psql -h MON_ENDPOINT.rds.amazonaws.com -p 5432 -U admin -d postgres
```
En local pour s'entraîner :
```bash
sudo apt install -y postgresql
sudo systemctl start postgresql
ss -tulpn | grep 5432                       # port ouvert ?
sudo -u postgres psql -h localhost -p 5432  # connexion locale (psql)
```

## Commandes AWS (après clés — Leçon 6)
```bash
aws rds create-db-instance --db-instance-identifier ma-base --db-instance-class db.t3.micro --engine postgres --allocated-storage 20 --master-username admin --master-user-password "MotDePasseTemporaire!"
```

## Bonnes pratiques
- Backups automatiques activés + **test de restauration**.
- Réplica pour lecture/failover (approfondi Bloc 7).
- Commencer petit (`db.t3.micro`), observer, dimensionner (FinOps Leçon 8).
- On différencie : credentials de la **base** vs **clés AWS** (IAM, Leçon 6).