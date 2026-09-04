# Leçon 4 — Formats de données YAML & JSON

> **Bloc 3 · Scripting et programmation** — Leçon 4 sur 7
> JSON et YAML sont **les deux formats que tu croiseras absolument partout** en DevOps : manifests Kubernetes, pipelines GitHub Actions/GitLab CI, `docker-compose.yml`, réponses d'API REST, configurations Ansible/Terraform. Tu connais déjà JSON en tant que dev (Spring/NestJS). Cette leçon consolide JSON et te fait maîtriser **YAML**, son « cousin » plus lisible mais plus traître.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Écrire et lire** du JSON et du YAML sans erreur de syntaxe (`{ }`, `[ ]`, indentation).
2. **Convertir mentalement** de l'un à l'autre (même structure, syntaxe différente).
3. **Identifier à l'œil** une erreur d'indentation ou de syntaxe YAML (le piège n°1).
4. **Manipuler** des fichiers YAML/JSON en ligne de commande avec `jq` (JSON) et `yq` (YAML).
5. **Expliquer** la notion de **sérialisation** et savoir où chaque format est utilisé en DevOps.

---

## 2. Explication simple

### 2.1 Le « pourquoi » : deux formats qui organisent de la donnée en texte

En DevOps, on configure **énormément de choses par de la donnée** : « 3 replicas », « port 8080 », « mot de passe de la base »… Cette donnée doit être stockable dans un fichier texte, lisible par un humain **et** par une machine. C'est le rôle des formats de **sérialisation** : transformer une structure (objet, liste) en texte.

```
Concepts
  (objet, liste, valeur)
        ↓ sérialisation
   texte (JSON ou YAML)
        ↓ dé-sérialisation
   Concepts à nouveau utilisables
```

L'analogie : JSON et YAML sont deux **langues** pour écrire la même phrase. JSON est un peu « robotique » (beaucoup d'accolades, strict) ; YAML est plus naturel (moins de symboles) mais exige une indentation **exacte**.

### 2.2 Le « comment » : la même chose en deux langues

Même structure en JSON :

```json
{
  "name": "api",
  "replicas": 3,
  "ports": [80, 443]
}
```

La **même** structure en YAML (attention à l'indentation, jamais de tabulation) :

```yaml
name: api
replicas: 3
ports:
  - 80
  - 443
```

| Point | JSON | YAML |
|-------|------|------|
| Structure | `{}` objets, `[]` listes | Indentation + `-` listes, `clé: valeur` |
| Commentaires | ❌ aucun | ✅ avec `#` |
| Strict / validable | ✅ très strict | ⚠️ sensible à l'indentation |
| Lisible pour un humain | OK | 🏆 meilleur |
| Utilisé où en DevOps | réponses API, config apps | Kubernetes, docker-compose, pipelines CI/CD |

### 2.3 Le « quand » : quel format choisir ?

- **YAML** quand on écrit de la **configuration** pour une personne qui la lit (docker-compose, manifests K8s, pipelines de CI). Lisible, commentable.
- **JSON** quand des **programmes** échangent de la donnée (réponses d'API REST, sorties d'outils). Plus sûr à parser, sans ambiguïté d'indentation.
---

## 3. Exemples concrets

### 3.1 Écrire du YAML proprement (exemple docker-compose)

```yaml
version: "3.8"
services:
  api:
    build: ./api
    ports:
      - "3000:3000"
    depends_on:
      - db
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: secret
volumes:
  db-data:
```

Points à retenir :
- **2 espaces** d'indentation (standard), jamais de tabulation.
- Une liste est introduite par `-` alignée sous la clé.
- Un **dictionnaire** imbriqué se décale d'un niveau.

### 3.2 Types dans YAML

```yaml
nom: api                # chaîne
replicas: 3             # nombre entier
ratio: 0.85             # nombre décimal
actif: true             # booléen (true/false, yes/no)
ports:                  # liste
  - 80
  - 443
config: {log: true}     # objet en une ligne (flow style)
```

### 3.3 Convertir JSON ↔ YAML avec `jq` et `yq`

```bash
# jq : est déjà installé ou via apt
jq . mon-fichier.json                      # formate (pretty-print)
jq '.services | keys' docker-compose.json  # sélection
jq '.replicas' api.json                    # champ précis

# yq : souvent à installer (pip ou binaire)
yq eval . config.yaml                      # formatte le YAML
yq -o=json '.' config.yaml > config.json   # YAML → JSON
yq -P '.' config.json > config.yaml        # JSON → YAML
```

> ℹ️ `jq` est quasi-universellement présent sur les serveurs : `sudo apt install jq`. `yq` est moins souvent préinstallé ; il existe aussi `python3 -c` + module PyYAML.

### 3.4 Vérifier la validité sans outil (réflexe d'œil)

Pour un YAML, cherche systématiquement :
1. La **tabulation** : elle est interdite → erreur.
2. L'**alignement** des `-` d'une liste (tous au même niveau).
3. Les clés d'un même dictionnaire **au même niveau d'indentation**.

```yaml
# ❌ faux :
ports:
   - 80
  - 443      # le tiret n'est pas aligné avec le précédent

# ✅ bon :
ports:
  - 80
  - 443
```

> **🛡️ DevSecOps — jamais de secrets en dur dans JSON/YAML** : que ce soit dans un `docker-compose.yml`, un manifest ou un `.conf`, on **ne met jamais** de mot de passe/clé/TOKEN en clair dans le fichier ni dans Git. On utilise des **variables d'environnement / références** (`${VAR}`, `env:`, `--from-env-file`) alimentées par un **coffre à secrets** (Vault, secret manager, secrets CI). Un fichier YAML avec un token exposé est une **faille courante** (fichier commité par erreur). Cette habitude devra t'être *naturelle* avant les blocs Docker/CI/CD.

```json
{
  "name": "api",        // ✅ guillemets doubles obligatoires
  "replicas": 3,        // ✅ virgule entre champs
  "ports": [80, 443]    // {} PAS de virgule finale après le dernier
}
```
---

## 4. Bonnes pratiques modernes (2025-2026)

- **Jamais de tabulation en YAML** : les tabulations rendent un fichier invalide ou ambigu. Configure ton éditeur pour transformer `Tab` en espaces (2 par défaut).
- **`jq` est le réflexe JSON** : formater (`jq .`), extraire un champ (`.replicas`), agréger (`|`) font partie du quotidien DevOps.
- **`yq` pour lire/écrire du YAML dans des scripts** : au lieu de parser du YAML à la main (fragile), on utilise `yq` — idem en CI/CD (Github Actions, GitLab CI).
- **Toujours valider avant de consommer** : en CI/CD, un manifest invalide fait planter le pipeline. On l'appelle tôt : `yq eval . app.yaml > /dev/null` ou `kubectl apply --dry-run=client -f file.yaml`.
- **Séparer la donnée de la structure** : rester minimaliste, éviter les valeurs répétées, utiliser des variables d'environnement quand l'outil le permet (éviter de hard-coder).
- **JSON en « pretty » pour la lecture humaine** : une réponse d'API compacte est illisible ; `jq .` l'aère. À l'inverse, on garde un JSON minifié pour le stockage (`jq -c`).
- **Les booléens YAML** : préférer `true`/`false` (au lieu de `yes`/`no`) pour éviter les ambiguïtés inter-outils.

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi c'est inefficace / casse | ✅ Version correcte |
|----------------|--------------------------------------|---------------------|
| Indentation YAML avec des **tabs** | Tabulation non autorisée → parseur en erreur silencieuse. | Espaces uniquement (2 ou 4, homogène). |
| Liste `-` non alignés dans un YAML | L'indentation d'une liste rompt la structure → erreur. | Aligner tous les `-` au même niveau. |
| **Virgule** finale dans un JSON | JSON strict n'accepte pas de virgule de fin → parse erreur. | Pas de virgule après le dernier élément. |
| Guillemets **simples** en JSON | JSON exige des doubles guillemets (`"`) pour les clés et valeurs. | Guillemets doubles `"name": "api"`. |
| Confondre **`yes`/`no`** avec booléen | Selon le parseur, `yes`/`no` peuvent être des chaînes ou booléens → comportement différent. | `true` / `false` explicites. |
| Parser du YAML à la main dans un script | Fragile : une espace de plus et tout change. | `yq` (ou un parseur dédié). |
| Croire que le JSON est lisible sans formatage | Une réponse API minifiée est illisible pour déboguer. | `jq .` pour le pretty-print. |

> Note : dans la ligne du bug d'indentation la plus courante, c'est **un espace de décalage** qui casse tout. Quand tu soupçonnes un problème YAML : **ré-indente de zéro** ou passe par `yq`.

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`**, la correction commentée dans **`03-correction.md`**. Lis bien cette leçon avant de t'y mettre.

**Énoncé court** : à partir d'un exemple de `docker-compose.yml` (services, environnements, volumes — comme dans la leçon) et d'une réponse d'API JSON, écris la **version YAML** d'un manifest minimal (indentation 2 espaces), convertis-la **mentalement** en JSON, puis valide : lis un champ avec `jq`, transforme un fichier YAML avec `yq`, et repère à l'œil une mauvaise indentation / tabulation.

---

## 7. Correction détaillée de l'exercice

> La correction complète pas-à-pas est dans **`03-correction.md`**. Essentiel du raisonnement :
- **2 espaces**, jamais de tabulation ; une **liste** introduite par `-` alignée, un **objet** imbriqué décalé d'un niveau ;
- la **même structure** JSON ↔ YAML (mêmes clés/valeurs, syntaxe différente), on la convertit à la main ;
- **`jq`** (JSON) : `.champ`, `jq -c`, `jq .` ; **`yq`** (YAML) : `yq eval . fichier.yml` pour lire ;
- **valider avant** (`yq eval . > /dev/null` / `kubectl apply --dry-run`) pour attraper les erreurs tôt ;
- repérer **tabulation** et **alignement** des `-` = le piège n°1 YAML.

---

## 8. Checklist de validation

- [ ] Je sais écrire **JSON** correct (doubles guillemets, virgules entre éléments, pas de virgule finale).
- [ ] Je sais écrire **YAML** correct (indentation 2 espaces, listes `-`, objets imbriqués, `#` commentaires).
- [ ] Je peux **convertir mentalement** une structure JSON ↔ YAML.
- [ ] J'utilise **`jq`** pour formater/extraire du JSON, et **`yq`** pour manipuler du YAML.
- [ ] Je sais repérer à l'œil une **tabulation** ou un **mauvais alignement** YAML.
- [ ] Je connais les cas où on choisit YAML (config) vs JSON (échange programme).

---

🧭 **Pont vers la suite** — Tu sais maintenant écrire et lire de la **donnée** (YAML/JSON) en scripts. Mais quand la logique devient complexe — fichiers, API, erreurs — **Bash atteint ses limites**. La suite du bloc bascule donc vers le **langage du DevOps moderne** : **Python**. La **Leçon 5** te montre sa syntaxe et la mise en place d'un environnement propre (`venv`).

---

*Prochaine étape :* Leçon 5 — **Introduction à Python pour DevOps** dans `05-Python-pour-DevOps-bases/`.