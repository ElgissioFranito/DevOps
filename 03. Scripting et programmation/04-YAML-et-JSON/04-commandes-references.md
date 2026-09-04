# Aide-mémoire — Formats de données YAML & JSON

> **Bloc 3 · Leçon 4** — Fiche de référence pour écrire/valider/convertir du YAML et du JSON.

## 📌 JSON — règles

- Objets : `{ }`, listes : `[ ]`
- Clés et chaînes en **doubles guillemets** `"name": "api"`
- Nombres / booléens **non** cités : `3`, `false`
- **Pas de virgule** après le dernier élément

```json
{
  "name": "api",
  "ports": [80, 443]
}
```

## 📌 YAML — règles

- Indentation **espaces** (2), jamais de tabulation
- Objet : `cle: valeur` ; liste : `- element`
- Commentaires avec `#`

```yaml
name: api
ports:
  - 80
  - 443
environment:
  ENV: production
```

## 📌 Types YAML

| Valeur | Type |
|--------|------|
| `nom: api` | chaîne |
| `replicas: 3` | entier |
| `ratio: 0.85` | décimal |
| `actif: true` | booléen (`true`/`false`) |
| `ports:` + `- 80` | liste |

## 📌 `jq` (JSON)

```bash
jq . fichier.json                     # pretty-print
jq '.replicas' fichier.json           # un champ
jq '.ports[0]' fichier.json           # 1er element d'une liste
jq '.environment.ENV' fichier.json    # champ imbriqué
jq -c . fichier.json                  # minifier
```

## 📌 `yq` (YAML)

```bash
yq eval . fichier.yaml                # valider / formater
yq -o=json . fichier.yaml             # YAML → JSON
yq -P . fichier.json                  # JSON → YAML
yq '.services | keys' fichier.yaml    # selection
```

## 📌 Conversion / validation rapide

```bash
# valider JSON
jq . fichier.json > /dev/null && echo OK

# valider YAML
yq eval . fichier.yaml > /dev/null && echo OK
```

## 📌 Pièges fréquents

- tabulation en YAML → interdit
- tirets `-` mal alignés
- virgule finale JSON
- simples quotes en JSON
- `yes`/`no` au lieu de `true`/`false`
