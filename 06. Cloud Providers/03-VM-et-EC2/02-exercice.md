# Exercice — Leçon 3 : Machines virtuelles (EC2)

> **Bloc 6 · Leçon 3** — Exercice en autonomie. **Aucune instance payante n'est lancée** : on travaille la décision et la simulation (script local) — les vraies commandes viendront avec le compte configuré (Leçon 6).

---

## Contexte

Ton entreprise veut héberger le **backend Spring Boot** (50 requêtes/min en moyenne). Avant de cliquer « créer », il faut **décider** et **documenter** : c'est ce qu'on fait ici.

---

## Énoncé

### Étape 1 — Rédiger la « fiche de décision » (dans `notes-exercice-03.md`)

Complète ce tableau avec tes choix + 1 phrase d'explication chacun :

| Question | Ton choix | Pourquoi (1 phrase) |
|----------|-----------|---------------------|
| Quoi (service AWS) | EC2 | ? |
| AMI (système) | ? | ? |
| Type d'instance | ? (petit/moyen ?) | ? |
| Subnet (public ou privé) | ? | ? |
| Security group (ports) | ? | ? |
| Clé SSH (nom) | ? | ? |
| Snapshot (fréquence) | ? | ? |

### Étape 2 — Créer et lancer le script `simuler-instance.sh`

Reprends l'exemple 3.2 de la leçon (l'affichage `🧠 Simulation…`), puis exécute-le :

```bash
nano simuler-instance.sh   # crée le fichier
bash simuler-instance.sh   # l'exécute (avec Bash — Bloc 3)
```

Colle la sortie dans `notes-exercice-03.md`.

### Étape 3 — Réflexion (dans `notes-exercice-03.md`)

1. Que coûte une instance **running** vs **stopped** vs **terminated** (en 2 lignes) ?
2. Si la demande passe de 50 à 50 000 requêtes/min, que proposes-tu ? (Pense dimensionnement et autoscaling.)

---

## Livrable

`notes-exercice-03.md` (fiche remplie + sortie du script + 2 réponses) + `simuler-instance.sh`.

Correction détaillée dans **`03-correction.md`**.