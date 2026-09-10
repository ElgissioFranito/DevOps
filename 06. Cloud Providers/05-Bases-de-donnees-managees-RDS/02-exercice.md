# Exercice — Leçon 5 : Bases de données managées (RDS)

> **Bloc 6 · Leçon 5** — Exercice en autonomie. Entraînement **en local** (PostgreSQL) + conception sur papier : **aucune RDS payante n'est créée**.

---

## Contexte

Ton application Spring Boot a besoin d'une **base PostgreSQL** pour ses données. Tu dois comprendre ce que change le « managé » et savoir **où** la placer dans le réseau (réflexe Leçon 2).

---

## Énoncé

### Étape 1 — Pratiquer la connexion (en local)

Installe et démarre PostgreSQL (ou réutilise celui du Bloc 2) :

```bash
sudo apt update && sudo apt install -y postgresql   # installe (si pas déjà fait)
sudo systemctl start postgresql                     # démarre le service
ss -tulpn | grep 5432                               # voit-on le port 5432 ? (guichet PostgreSQL)
sudo -u postgres psql -h localhost -p 5432          # se connecte en local
# dans psql : \l liste les bases ; \q quitte
```

Note dans `notes-exercice-05.md` : le résultat de `ss -tulpn | grep 5432` et ce que tu as vu avec `\l` (lister).

### Étape 2 — Ce que « managé » change (dans `notes-exercice-05.md`)

Liste **3 tâches** que le cloud fait à ta place avec RDS (par rapport à une base auto-hébergée), et **2 tâches** qui restent **à toi** (ce que le cloud ne fait pas).

### Étape 3 — Schéma d'architecture (dans `notes-exercice-05.md`)

Dessine le schéma (ASCII) : `Internet → Load Balancer → Application → RDS PostgreSQL`, et ajoute dessous **3 règles de sécurité** pour la base (pense subnet privé, security group, credentials/coffre).

---

## Livrable

`notes-exercice-05.md` (sortie de connexion + 3+2 tâches + schéma + 3 règles).

Correction détaillée dans **`03-correction.md`**.