# Exercice — Du Git à la production

> **Bloc 1 · Leçon 4** — Exercice de synthèse, à réaliser **en autonomie**, après avoir lu `01-leçon.md` (et idéalement les 3 leçons précédentes).

---

## 🎯 Objectif de l'exercice

Prouver que tu sais **expliquer le parcours complet** de ton application : de ton ordinateur (Git) jusqu'au serveur de production. C'est le **critère final du bloc**.

---

## 📋 Énoncé

> Utilise la **même fonctionnalité** que les Leçons 1, 2 et 3 (ex. : le panier d'achat).

### Étape 1 — Redessine le schéma complet

Reproduis le schéma mental, à la main ou en texte :

```text
GIT → BUILD → TESTS → ARTIFACT → DEPLOY(STAGING) → DEPLOY(PROD) → MONITOR
```

Ajoute, là où c'est pertinent, les liens vers les notions vues en L1/L2/L3 (backlog, environnements, monitoring de boucle).

### Étape 2 — Explique chaque étape (1-2 phrases)

Pour **chacune** de ces étapes, écris une ou deux phrases : *ce qu'on fait* + *avec quoi* (commandes/outils) :

1. Git
2. Build
3. Tests
4. Artifact
5. Déploiement en staging
6. Déploiement en production
7. Monitoring

### Étape 3 — Le point clé du « même artifact »

Explique **en 2-4 phrases** pourquoi on ne **re-construit** **pas** le code en production, mais on déploie **l'exact artifact** validé en staging. (Tu peux t'appuyer sur l'analogie du plat livré.)

### Étape 4 — Le discours final (le plus important)

À **voix haute**, sans notes, réponds à la question finale du bloc :

> « Comment ton code passe de ton ordinateur jusqu'au serveur de production ? »

Objectif : **5 à 8 phrases fluides**, dans l'ordre, sans hésiter. Entraîne-toi jusqu'à y arriver sans bafouiller.

### Étape 5 — Auto-vérification

Avant d'ouvrir `03-correction.md`, vérifie :

1. Ai-je bien mentionné **toutes** les étapes dans le bon ordre ?
2. Ai-je expliqué **quoi** et **avec quoi** (pas juste le nom d'une étape) ?
3. Ai-je bien justifié le **même artifact** (pas de recompilation en prod) ?

---

## 📦 Livrable attendu

- Le schéma (étape 1),
- l'explication par étape (étape 2),
- le paragraphe « même artifact » (étape 3),
- un enregistrement mental (voix) de ton discours final (étape 4),
- tes réponses d'auto-vérification (étape 5).

> ⏱️ **Temps estimé** : 25 à 35 minutes (+ entraînement oral).

---

*Ensuite, compare avec `03-correction.md`.*