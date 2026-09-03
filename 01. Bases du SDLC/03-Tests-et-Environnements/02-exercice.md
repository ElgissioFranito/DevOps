# Exercice — Tests et environnements

> **Bloc 1 · Leçon 3** — Exercice pratique, à réaliser **en autonomie**, après avoir lu `01-leçon.md`.

---

## 🎯 Objectif de l'exercice

Montrer que tu sais **raisonner sur les tests, les environnements et le build** de ta fonctionnalité, sans avoir besoin d'exécuter de code. Tu vas appliquer la théorie à ta fonctionnalité du panier.

---

## 📋 Énoncé

> Utilise la **même fonctionnalité** que les Leçons 1 et 2 (ex. : le panier d'achat).

### Étape 1 — Tes tests (3 + 1 + 1)

Rédige tes **5 tests** suivants, chacun en **une phrase** décrivant ce qu'on vérifie :

1. **3 tests unitaires** (ex. : le calcul du total, l'ajout d'un produit, la suppression).
2. **1 test d'intégration** (ex. : code du panier + base de données).
3. **1 test E2E** (le parcours utilisateur complet).

Pour chaque test, précise aussi le **résultat attendu** (ex. : « le total vaut 10 € »).

### Étape 2 — Associe les tests aux environnements

Pour chaque niveau (unitaire, intégration, E2E), indique **où** il se déroule naturellement (PC du dev / dev / staging) et **pourquoi** (rapide ? demande l'app complète ?).

### Étape 3 — Le build de ta fonctionnalité

Décris :

- quel **code source** on part (ex. : projet Java Spring, projet Node/NestJS),
- la **commande** de build (`mvn clean package` ou `npm run build`),
- **quel artifact** est produit (`.jar` ? `dist/` ? quelque chose d'autre).

### Étape 4 — Pourquoi staging ?

Explique **en 2-3 phrases** pourquoi on déploie sur staging avant la production, en t'appuyant sur :
- ce qu'on risque si on saute staging,
- le principe de « la plus proche possible de la production ».

### Étape 5 — Auto-vérification

Avant d'ouvrir `03-correction.md` :

1. Mon test unitaire est-il **isolé** (il ne dépend pas de la base ou du réseau) ?
2. Mon test E2E décrit-il un **parcours complet utilisateur**, pas juste une fonction ?
3. Ai-je bien distingué **build** (action) et **artifact** (résultat) dans l'étape 3 ?

---

## 📦 Livrable attendu

Un document contenant les 5 tests (étape 1), la table dev/staging/prod (étape 2), la description de build (étape 3), l'explication staging (étape 4), et tes réponses (étape 5).

> ⏱️ **Temps estimé** : 20 à 30 minutes.

---

*Ensuite, compare avec `03-correction.md`.*