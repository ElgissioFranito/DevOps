# Correction — Leçon 7 : Contrôle d'accès et secrets

> **Bloc 5 · Leçon 7** — Correction pas à pas.

---

## Étape 1 — Projet avec de faux secrets

```bash
mkdir projet-demo && cd projet-demo
cat > .env <<'EOF'
DB_HOST=localhost
DB_PORT=5432
DB_PASSWORD=FAUX-motdepasse
API_KEY=FAUX-cle
EOF
```

## Étape 2 — `.gitignore`

```bash
cat > .gitignore <<'EOF'
.env
*.key
*.pem
EOF
git init
git add .
git status        # .env ne doit PAS apparaître (ignoré)
```

**Explication** : grâce au `.gitignore`, Git **s'arrête avant** de committer le `.env`. C'est la protection — on la met AVANT, pas après.

Vérification plus explicite :
```bash
git check-ignore -v .env   # affiche la règle qui l'ignore (le .gitignore)
```

## Étape 3 — Charger le secret

```bash
set -a && source .env && set +a
echo "La base est $DB_HOST"   # -> localhost
```
> `set -a` exporte automatiquement chaque ligne lue par `source` ; `set +a` arrête ça. On **n'affiche pas** le mot de passe en pratique.

## Étape 4 — Quiz

**1. Authentification vs autorisation ?**
> **Auth** = vérifier QUI tu es (saisir ton login + mot de passe). **Autorisation** = vérifier CE QUE tu peux faire (ton rôle). Ex. : tu es identifié (auth), mais sans le rôle `admin`, tu ne peux pas supprimer (autorisation).

**2. RBAC ou ABAC ?**
> **RBAC** si des rôles fixes suffisent (majorité des cas : devs, admins, lecteurs). **ABAC** s'il faut des règles fines conditionnelles (ex. « accès RH seulement 8h-18h, ressource interne »).

**3. Pourquoi ne jamais mettre un secret dans Git ?**
> Une fois dans l'historique, le secret y reste **à jamais** (chaque commit ancien le contient), même si on le retire ensuite. Il peut être récupéré par n'importe qui ayant accès au dépôt (public ou fuite).

**4. Deux façons propres ?**
> Variables d'environnement ou un **coffre** (Vault, AWS Secrets Manager, Kubernetes Secrets). Dans le pipeline, les **secrets CI/CD** de l'outil (Bloc 11).

---

## Checklist de validation (leçon 6)

- [ ] Je distingue authentification / autorisation.
- [ ] J'explique RBAC vs ABAC et je choisis selon le besoin.
- [ ] J'applique le moindre privilège.
- [ ] Mon `.env` est protégé par `.gitignore` et absent de Git.
- [ ] Je sais charger un `.env` sans l'afficher.
- [ ] Je cite plusieurs solutions propres de secrets.

---

## 🧠 Conseils pour la suite

- **Si tu as déjà commité un secret** : considère-le comme compromis → **rotation** (changer la clé), pas seulement suppression.
- Entraîne-toi à écrire `.gitignore` avant de coder (réflexe Pro).
- Ces notions resservent dans **Kubernetes (RBAC, Secrets)**, **AWS (IAM, Secrets Manager)**, **CI/CD** (Blocs 11, 13).