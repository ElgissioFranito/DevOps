# Correction détaillée — Formats de données YAML & JSON

> **Bloc 3 · Leçon 4** — Correction pas-à-pas de `02-exercice.md`. Suis chaque étape et compare à ton travail.

---

## ✅ Étape 1 — La config en JSON

```bash
mkdir -p ~/mon-config
cd ~/mon-config
nano config.json
```

```json
{
  "name": "api",
  "replicas": 3,
  "ports": [3000, 3001],
  "environment": {
    "ENV": "production",
    "DEBUG": false
  }
}
```

Validation :

```bash
jq . config.json
# → affiche le JSON indenté, aucune erreur
```

**Explications** :
- Les **clés** et les **chaînes** sont en **doubles guillemets** (`"`) — obligatoire en JSON.
- Les **nombres** (`3`, `3000`) et **booléens** (`false`) ne sont **pas** entre guillemets (sinon ils seraient des chaînes).
- Un objet = `{ }`, une liste = `[ ]`. Aucune **virgule** après le dernier élément d'une liste/objet.

---

## ✅ Étape 2 — La même config en YAML

```yaml
# configuration de l'api
name: api
replicas: 3
ports:
  - 3000
  - 3001
environment:
  ENV: production
  DEBUG: false
```

Validation :

```bash
yq eval . config.yaml
# → affiche le YAML, sans erreur
```

**Explications** :
- Une **liste** se note par `- ` alignés sous la clé (`ports:` puis deux tirets au même niveau).
- Un **objet** imbriqué se décale de 2 espaces (`environment:` puis `  ENV:` / `  DEBUG:` au même niveau).
- Les clés sont des **chaînes implicites** (pas besoin de guillemets en YAML).
- Le commentaire `#` est propre à YAML (JSON n'en a pas).

---

## ✅ Étape 3 — Conversion

```bash
yq -o=json . config.yaml > c2.json
cat c2.json

# la sortie doit être équivalente à config.json
yq -P . config.json > c2.yaml
cat c2.yaml
```

**Explication** : `-o=json` produit du JSON, `-P` (ou `--prettyPrint`) produit du YAML. Le **contenu logique** est identique ; seule la **syntaxe** change. C'est toute la notion de **sérialisation** : la donnée reste la même, la représentation change.

---

## ✅ Étape 4 — Diagnostics

**Fichier A (JSON)** : la **virgule finale** après `[3000, 3001],` est interdite en JSON.

Correction :

```json
{
  "name": "api",
  "replicas": 3,
  "ports": [3000, 3001]
}
```

**Fichier B (YAML)** : le second tiret ` - 3001` est **mal aligné** (3 espaces au lieu de 2). Une liste doit avoir des tirets au **même niveau d'indentation**.

Correction :

```yaml
name: api
ports:
  - 3000
  - 3001
```

---

## ✅ Étape 5 — Réponses de l'auto-vérification

1. **Tabulation vs espaces** : YAML définit la hiérarchie **par l'indentation**. Une tabulation a une largeur visuellement différente d'un espace et les parseurs YAML la rejettent (ou l'interprètent différemment d'un éditeur à l'autre). Un fichier « visuellement juste » avec tab peut donc être **invalide**. Règle : toujours des **espaces**.
2. **Liste vs objet** : une **liste** est une suite ordonnée de valeurs (JSON `[a, b]`, YAML `- a / - b`) ; un **objet/dictionnaire** est un ensemble clé→valeur non ordonné (JSON `{k:v}`, YAML `k: v` indentation-sensible).
3. **YAML vs JSON** : YAML pour la **configuration lisible par des humains** (docker-compose, K8s, pipelines) car plus concis et commentable ; JSON pour les **échanges entre programmes** (réponses API, sorties d'outils) car plus strict et facile à parser/valider.
4. **Sérialisation** : transformer une structure de données (objet, dictionnaire) en **texte** (JSON/YAML) pour la stocker ou l'envoyer sur le réseau ; la **désérialisation** fait l'inverse.

---

## 📝 Checklist de validation (récapitulatif + conseils)

- [ ] J'ai écrit un JSON **valide** (doubles guillemets, pas de virgule finale) et `jq .` le valide.
- [ ] J'ai écrit le YAML équivalent (indentation 2 espaces, listes `-`, objets imbriqués) et `yq` le valide.
- [ ] J'ai converti JSON ↔ YAML avec `yq` et compris que la donnée est identique.
- [ ] J'ai repéré une **virgule finale JSON** et un **tiret mal aligné YAML**.
- [ ] Je sais quand choisir YAML (config) vs JSON (échange programme).
- [ ] Je peux définir **sérialisation** avec mes mots.

### 💡 Conseils pour la suite

- **Re-teste-toi** dans 2-3 jours en refaisant uniquement la partie YAML (sans la correction).
- **Réflexe pro** : face à un pipeline/manifest invalide, soupçonne d'abord l'**indentation** (YAML) ou une **virgule/simple quote** (JSON).
- La **Leçon 5** démarre **Python** — le langage que tu découvriras ensuite. Les notions de fichiers JSON/YAML y seront indispensables (`json`, `yaml` sont des modules Python).

---

*Prochaine étape :* Leçon 5 — **Introduction à Python pour DevOps** → dossier `05-Python-pour-DevOps-bases/`.
