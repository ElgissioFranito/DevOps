# Introduction au Bloc 3 — Scripting et programmation

> **À lire en premier**, avant la Leçon 1. Ce fichier te dit :
> - de quoi parle ce bloc et **pourquoi il est indispensable en DevOps**,
> - ce qu'il te faut **préparer** avant de commencer (matériel, Python…),
> - les **7 leçons** du bloc et le **fil rouge** qui les relie,
> - le vocabulaire des **outils et concepts** que tu vas croiser.

---

## 🎯 De quoi parle ce bloc ?

Dans le **Bloc 2**, tu as appris à administrer Linux à la main (commandes, fichiers, services). Mais en DevOps, taper les mêmes commandes encore et encore est **perdre du temps et multiplier les erreurs**. Ce bloc te donne le **super-pouvoir de l'automatisation** : transformer tes commandes manuelles en **scripts réutilisables**, d'abord en **Bash**, puis en **Python** (le langage DevOps moderne), en passant par les **formats de données** (YAML/JSON) qui configurent tout l'écosystème.

> 💡 **C'est un bloc très « concret »** : tu vas écrire du vrai code, le rendre exécutable, le tester. Prépare ton terminal.

---

## ✅ Prérequis et préparation

- **Du Bloc 2 (Linux)** : savoir naviguer, lire des fichiers, lancer des commandes (`cd`, `chmod`, `nano`…). C'est acquis si tu as suivi le bloc 2.
- **Python** : installe-le si absent (`python3 --version`). Sous Ubuntu : `sudo apt install python3 python3-venv python3-pip`.
- **Un éditeur** : `nano` (suffit) ou vs code ; et **`shellcheck`** pour les scripts Bash (`sudo apt install shellcheck`).
- Aucune installation de dépendance pour les leçons Bash ; pour Python, on apprendra à créer un `venv`.

> ⚠️ **Conseil** : tu peux **lire toutes les leçons** d'abord pour avoir la vision, mais les scripts **ne prennent leur sens qu'en les exécutant**. Teste chaque exemple au fil de l'eau.

---

## 🗺️ Les 7 leçons du bloc (et le fil rouge)

Le fil rouge : *« créer un outil de diagnostic système qui vérifie un serveur, analyse des logs et produit un rapport »* — c'est le critère « bloc acquis » de la roadmap.

| # | Leçon | Compétence |
|---|-------|-----------|
| 1 | Bases du scripting Bash | Premiers scripts, variables, args, exit codes |
| 2 | Structures de contrôle & fonctions Bash | `if`, `for`, `while`, `case`, fonctions |
| 3 | Scripts Bash fiables | `set -euo pipefail`, `trap`, validation, `shellcheck` |
| 4 | Formats de données YAML & JSON | Écrire, lire, `jq`/`yq` |
| 5 | Introduction à Python pour DevOps | `venv`, variables, listes/dicts, fichiers |
| 6 | Automatisation avec Python | Fichiers, JSON, **appels HTTP/API**, erreurs |
| 7 | **Projet récapitulatif** : diagnostic & rapport | Assembler Bash+Python+YAML dans un vrai outil |

Chaque dossier contient 4 fichiers : `01-lecon.md`, `02-exercice.md`, `03-correction.md`, `04-commandes-references.md`.

---

## 🧠 Vocabulaire & outils que tu vas croiser

| Terme | C'est quoi ? (1 phrase) | Tu l'apprendras vraiment |
|-------|--------------------------|--------------------------|
| **Script** | Un fichier texte de commandes exécutables qui automatise une tâche | Partout dans le bloc |
| **Shebang** | La 1re ligne `#!/usr/bin/env bash` qui désigne l'interpréteur | Leçon 1 |
| **Exit code** | Le nombre retourné par un script (0 = succès, non-0 = erreur) | Leçon 1 |
| **`set -euo pipefail`** | La « ceinture de sécurité » qui fait échouer tôt et proprement un script | Leçon 3 |
| **venv / pip** | Environnement Python isolé et gestionnaire de paquets | Leçon 5 |
| **jq / yq** | Outils de manipulation JSON / YAML en ligne de commande | Leçon 4 |
| **shellcheck** | Linter Bash : détecte les bugs avant exécution | Leçon 3 |
| **YAML / JSON** | Formats de configuration / échange de données | Leçon 4 |
| **`requests` (HTTP)** | Faire des appels réseau/API en Python | Leçon 6 |
| **`subprocess`** | Lancer des commandes système depuis Python | Leçons 6-7 |

> ✅ **Comprendre la logique**, la *transposition* de ta logique dev (Java/JS) vers ces mondes compte plus que la mémorisation des commandes.

---

## 🧭 Où mène ce bloc ?

Après la Leçon 7, tu sauras **automatiser un vrai diagnostic Linux** (Bash + Python) et formatter ses données (JSON/YAML). C'est la base :
- pour **Git** (bloc 4) : versionner ces scripts comme du code ;
- puis **GitOps / CI** (blocs 11, 13) : tes scripts deviendront les actions des pipelines ;
- et pour **Docker** (bloc 9) : tu sauras *écrire* les scripts qu'on embarque dans les conteneurs.

---

*Démarre maintenant avec la **Leçon 1** (les bases du scripting Bash) dans `01-Scripts-Bash-bases/`.*