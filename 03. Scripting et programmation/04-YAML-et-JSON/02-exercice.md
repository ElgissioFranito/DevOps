# Exercice pratique — Formats de données YAML & JSON

> **Bloc 3 · Leçon 4** — Exercice à faire en autonomie.
> Contexte : tu prépares la configuration d'une micro-application (un service web) qu'on déploiera plus tard avec Docker. On te demande d'écrire la **même** configuration en **JSON** puis en **YAML**, de la convertir, et de repérer un bug d'indentation.

---

## 🎯 Objectif de l'exercice

Manipuler sans erreur les deux formats que tu utiliseras partout en DevOps : écrire une config **API** en JSON, la transcrire en YAML, convertir avec `jq`/`yq`, et diagnostiquer deux fichiers volontairement cassés.

---

## 📋 Étape 1 — Écrire une config en JSON

Crée `~/mon-config/config.json` qui décrit ceci :

- une application nommée `api` ;
- `replicas` = 3 ;
- un champ `ports` qui est une **liste** `[3000, 3001]` ;
- un champ `environment` qui est un objet `{ "ENV": "production", "DEBUG": false }`.

Valide sa syntaxe : `jq . config.json` (doit afficher un JSON indenté sans erreur).

---

## 📋 Étape 2 — Transcrire en YAML

Crée `~/mon-config/config.yaml` qui décrit **exactement la même structure**, en YAML cette fois.

Rappel des règles :
- indentation de **2 espaces**, jamais de tabulation ;
- les listes sont introduites par `-` ;
- les objets imbriqués se décalent d'un niveau ;
- tu peux ajouter un commentaire `# configuration de l'api`.

Valide avec `yq eval . config.yaml` (si `yq` absent, utilise un simple éditeur et vérifie visuellement l'indentation).

---

## 📋 Étape 3 — Conversion

1. Convertir le YAML en JSON : `yq -o=json . config.yaml > c2.json` (ou via un convertisseur en ligne).
2. Comparer : `cat c2.json` doit ressembler à `config.json`.
3. Convertir le JSON en YAML : `yq -P . config.json > c2.yaml`.

---

## 📋 Étape 4 — Diagnostics (cherche les bugs)

Pour chacun des 2 fichiers ci-dessous, identifie **précisément** l'erreur et donne la version corrigée.

**Fichier A (JSON)** :

```json
{
  "name": "api",
  "replicas": 3,
  "ports": [3000, 3001],
}
```

**Fichier B (YAML)** :

```yaml
name: api
ports:
  - 3000
   - 3001
```

---

## 📋 Étape 5 — Auto-vérification

1. Pourquoi une **tabulation**, interdite en YAML, peut casser le fichier alors qu'elle « semble » juste visuellement ?
2. Quelle est la différence entre une **liste** et un **objet** dans les deux formats ?
3. Quand préfères-tu YAML à JSON, et l'inverse ?
4. Que signifie **sérialisation** ?

---

## 🏁 Rendu attendu

`config.json` + `config.yaml` valides, la conversion JSON↔YAML effectuée, et tes 2 diagnostics + 4 réponses écrites.

> Compare ensuite avec `03-correction.md`.
