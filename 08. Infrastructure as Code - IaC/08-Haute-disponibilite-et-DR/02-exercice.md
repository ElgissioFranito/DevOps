# Exercice — Leçon 8 : concevoir pour survivre

> **Bloc 8 · Leçon 8** — Exercice de **conception** (plus de code). Tu produis un **document d'architecture** : schéma, choix de scalabilité, analyse de pannes, RTO/RPO. Le livrable est un fichier `notes-exercice-08.md` — la correction en donne une version complète.

---

## Contexte

Ton application (la bibliothèque du fil rouge — frontend, backend Spring Boot, PostgreSQL) devient populaire : **10 000 utilisateurs actifs**, contrainte forte : *« le service doit reprendre en moins de 30 minutes après un sinistre, et perdre au maximum 15 minutes de données »*. Ton architecture actuelle (Leçon 5) : **une** EC2, **une** base RDS. Cet exercice fait de toi l'architecte.

---

## Énoncé

### Étape 1 — Le schéma d'architecture cible

Dessine (en ASCII dans tes notes, ou sur papier — photo à joindre) l'architecture cible avec **exactement ces éléments** :

```
Utilisateurs
     │
[ ?? ]           ← quel composant repartit la charge et masque les pannes ?
     │
[ ?? ]  [ ?? ]   ← combien de machines applicatives ?
     │
[ ?? ]           ← quel réglage protège la base d'une panne de zone ? (Bloc 7, Leçon 6)
     │
[ ?? ]           ← quel mécanisme garantit le RPO ? (Bloc 7, Leçon 4)
```

Pour chaque `??`, donne : le composant, son nom Terraform/chez AWS quand tu le connais (ex. `aws_lb` pour le load balancer — tu l'indiqueras d'après tes acquis des blocs 6-7, c'est une lecture de ta propre Leçon 5), et **une phrase** sur son rôle.

### Étape 2 — Choix de scalabilité, justifiés

Pour chaque composant, réponds **verticale ou horizontale** (ou « les deux, dans cet ordre ») + une justification d'une phrase :

1. Les machines applicatives (elles reçoivent les requêtes web).
2. La base de données PostgreSQL (une écriture cohérente).
3. Le stockage des documents (S3).

### Étape 3 — L'analyse de pannes (le cœur de l'exercice)

Pour chaque incident, écris : **ce qui se passe immédiatement** / **ce que l'utilisateur voit** / **l'action qui restaure le service** / **la durée estimée (RTO approx.)**.

1. *Server 1 (l'EC2 applicative) tombe.* — avec ton architecture d'étape 1, la réponse doit être « l'utilisateur ne voit rien » : explique pourquoi (health check + bascule).
2. *Un serveur est saturé (10 000 → 50 000 utilisateurs).*
3. *La base de données est détruite.* — la question exacte de la roadmap ! Reponse attendue : qui protège ? (réplication / snapshot / restore), et quelle perte de données max (RPO) ?
4. *Toute la région cloud est indisponible pendant 2 heures.* — que prévoit le plan DR ? (à ce stade : le plan écrit + les backups HORS de la région + la reconstruction IaC. Sois honnête sur ce qui n'est pas automatique.)

### Étape 4 — RTO/RPO cibles et plan de sauvegarde

1. Ton exigence : RTO 30 min, RPO 15 min. Écris la **stratégie de sauvegarde** (fréquence des sauvegardes, où elles vivent — région ?) qui **garantit** le RPO 15 min. Justifie avec le calcul vu en leçon (§ 2.4).
2. Écris le **plan DR en 6 étapes numérotées** (du constat du sinistre au service rendu), en indiquant à chaque étape : à la main, ou via IaC (quelles commandes) ?
3. Le test : comment **prouvera** tu une fois par trimestre que ce plan marche ? (rappel du Bloc 7, Leçon 8 : le test trimestriel de restauration.)

### Étape 5 — Le lien avec ton code (bonus réel)

Reprends le code de la Leçon 5 et réponds (pas besoin d'exécuter — c'est de la lecture de conception) :

1. Quelle ressource existe déjà pour le backup de la base ? (`skip_final_snapshot = true` — que se passe-t-il avec ce réglage ? en quoi le réglage production serait-il différent ?)
2. Si tu devais passer à **2 EC2**, que changerais-tu dans `main.tf` (quelle instruction de répétition — nommée dans la leçon § 3.4), et quels composants deviendraient **obligatoires** (`aws_lb`, health checks) ?
3. Pourquoi la Leçon 7 (un rôle Ansible configurant N cibles) est-elle la moitié de la solution ?

---

## Livrable

`notes-exercice-08.md` : le schéma complété, les 3 choix de scalabilité, les 4 analyses de pannes, le plan de sauvegarde + les 6 étapes DR + le test trimestriel, et les 3 réponses du lien avec le code.

Correction détaillée (architecture complète, en une version de référence) dans **`03-correction.md`**.