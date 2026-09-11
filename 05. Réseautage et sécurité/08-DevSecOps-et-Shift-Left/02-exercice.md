# Exercice — Leçon 8 : DevSecOps et Shift-Left

> **Bloc 5 · Leçon 8** — Exercice à faire en autonomie. Deux scénarios au choix selon ton setup (Node ou dépôt Git seul).

---

## Contexte

Tu déploies bientôt ton premier pipeline (Bloc 11). Avant, tu t'entraînes à **lire un rapport de vulnérabilité** et à **classer les familles de scans** — exactement ce qu'un DevOps fait pour décider si on déploie ou pas.

---

## Énoncé

> 📌 **Rappel** : `npm` = gestionnaire de paquets Node (Bloc 3) ; `-level`/`--json` = options de sortie ; un **CVE** est un identifiant de faille (Leçon 7).

### Étape 1 — (Option A) Lire un rapport de dépendances
Si tu as Node :
```bash
mkdir sast-demo && cd sast-demo && npm init -y
npm install lodash@4.17.20   # une très vieille version (délibérément vulnérable)
npm audit --json             # liste les vulnérabilités avec sévérité et CVE
```
Ouvre le rapport : repère une **CVE**, sa **sévérité**, et la **version à jour** recommandée.

> 💡 **Pourquoi cette vieille version ?** Pour te montrer un vrai rapport de faille, en toute sécurité, sur une machine de test.

### Étape 2 — (Option B) Chercher des secrets dans un dépôt
```bash
git clone https://github.com/example/demo-repo.git   # ou ton propre dépôt de test
# avec gitleaks (outil) ou un simple grep :
grep -rn "password\|API_KEY\|secret" --exclude-dir=.git .
```
Note les fichiers où des mots-clés suspects apparaissent. (En vrai, on utilise un outil dédié, mais le principe est là.)

### Étape 3 — Classer les scans (réponds dans `notes-exercice-07.md`)
Pour chaque cas, dis quelle **famille de scan** est adaptée :
1. « Vérifier que mon code Java n'a pas d'injection SQL » → ?
2. « Savoir si la librairie `log4j` que j'utilise a une faille » → ?
3. « Vérifier une image Docker avant de la déployer » → ?
4. « Tester mon application lancée face à des attaques » → ?
5. « M'assurer qu'aucune clé API n'est dans Git » → ?

### Étape 4 — Réflexion
- Pourquoi vaut-il mieux détecter ces failles **avant** la production (shift-left) ?

---

## Livrable

`notes-exercice-07.md` avec : sorties du scan, réponses, réflexion.
Correction dans **`03-correction.md`**.