# Exercice — Leçon 6 : Contrôle d'accès et secrets

> **Bloc 5 · Leçon 6** — Exercice à faire en autonomie, tout local (pas de machine distante nécessaire).

---

## Contexte

Un collègue débutant a commité son `.env` (avec de faux mots de passe) par mégarde et te demande de l'aider à faire mieux. Tu vas refaire le geste propre : secrets hors Git, `.gitignore`, et réflexion sur les modèles d'accès.

---

## Énoncé

> 📌 **Rappel** : `git status` = voir les fichiers suivis/modifiés (Bloc 4) ; `set -a / source` = charger un fichier `.env` dans le shell (voir cours) ; `--allow-empty` inutile ici.

### Étape 1 — Projet avec de faux secrets
- Crée un dossier `projet-demo`.
- Crées-y un fichier `.env` avec :
  ```
  DB_PASSWORD=FAUX-motdepasse
  API_KEY=FAUX-cle
  ```

### Étape 2 — Se protéger avant tout commit
- Crée un `.gitignore` avec : `.env`, `*.key`, `*.pem`.
- `git init`, puis `git add .` et vérifie avec `git status` que `.env` n'apparaît **pas**.
- Commit seulement si le `.env` est ignoré.

### Étape 3 — Charger le secret dans un script
```bash
set -a && source .env && set +a
echo "La base est $DB_HOST"   # attends une variable du .env
```
> ❗ N'**affiche jamais** `DB_PASSWORD` en vrai — c'est un secret. Ici, un faux, pour comprendre.

### Étape 4 — Quiz (réponds dans `notes-exercice-06.md`)
1. Quelle est la différence entre **authentification** et **autorisation** ? Donne un exemple de chacun.
2. Dans quel cas préférer **RBAC** ? dans quel cas **ABAC** ?
3. Pourquoi ne jamais mettre un secret dans l'historique Git, même si on le retire après ?
4. Cite **2** façons propres de fournir un secret en production.

---

## Livrable

`notes-exercice-06.md` avec : captures `git status`, sortie du script, réponses au quiz.
La correction détaillée dans **`03-correction.md`**.