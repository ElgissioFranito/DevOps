# Leçon 7 — Contrôle d'accès et secrets

> **Bloc 5 · Réseautage et sécurité** — Leçon 7 sur 8
> 🧭 **Pont depuis la Leçon 6** : tu sais **exposer** une app proprement (proxy) et la protéger en réseau (pare-feu, TLS, VPN). Mais l'accès à un système, c'est aussi **qui a le droit de faire quoi** (utilisateurs, rôles) et **où l'on cache les mots de passe/clés** (les *secrets*). Cette leçon t'ouvre à ces deux piliers, présents partout (Kubernetes, IAM cloud, Git).

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Distinguer** authentification (qui es-tu ?) et autorisation (que peux-tu faire ?).
2. **Expliquer** les modèles **RBAC** (par rôles) et **ABAC** (par attributs), et savoir quand utiliser l'un ou l'autre.
3. **Appliquer** le principe de **moindre privilège** (n'accorder que le nécessaire).
4. **Expliquer** pourquoi on ne met **jamais** un secret dans Git, et citer les alternatives (variables d'environnement, secrets CI/CD, Vault, AWS Secrets Manager, Kubernetes Secrets).
5. **Mettre en pratique** : manipuler un `.env` local et comprendre la différence avec un vrai gestionnaire de secrets.
6. (survol) Citer **LDAP** (annuaire d'utilisateurs) sans creuser.

---

## 2. Explication simple

### 2.1 Le « pourquoi » : qui, et avec quels droits ?

Une application n'est pas seule : elle touche une base, appelle des API, s'exécute avec des comptes. Il faut répondre à deux questions distinctes :
- **Authentification** : *qui es-tu ?* (tu t'identifies)
- **Autorisation** : *que peux-tu faire ?* (tes droits)

> 💡 **Analogie** : l'**aéroport**. L'auth, c'est le contrôle des papiers (ta carte d'identité, on vérifie **qui tu es**). L'autorisation, c'est ton **titre d'embarquement** : même identifié, tu n'as pas accès au cockpit si ton billet ne le permet pas (on vérifie **ce que tu peux faire**).

### 2.2 RBAC vs ABAC (le « quoi »)

Deux façons standard de décider des droits :

**RBAC (Role-Based Access Control, contrôle d'accès par rôles)** — on assigne un **rôle** à un utilisateur, et chaque rôle a des permissions fixes.

```
Utilisateur "toto" ── rôle "admin"   ── peut lire/écrire/supprimer
Utilisateur "lili" ── rôle "lecteur" ── peut seulement lire
```

Simple, prévisible, facile à auditer → c'est le modèle par défaut de Kubernetes, des IAM cloud, de ton MySQL, etc.

**ABAC (Attribute-Based Access Control, par attributs)** — les droits dépendent d'**attributs** évalués dynamiquement (qui, quoi, quand, où), avec des règles conditionnelles.

```
Règle : autoriser l'accès SI
    département = "RH"          ET
    heure ∈ [8h-18h]            ET
    ressource.confidentialité = "interne"
```

Plus fin et flexible, mais plus complexe à mettre en place et à auditer.

> 📖 **Quand choisir ?** RBAC suffit pour 90 % des cas (des rôles définis suffisent) ; ABAC quand il faut des règles fines conditionnelles (départements, horaires, niveaux de confidentialité).

### 2.3 Le moindre privilège (le « comment »)

**Moindre privilège** = ne donner à chaque compte/processus **que** ce dont il a besoin pour travailler, et rien de plus. On l'a déjà vu avec les ports (Leçon 3) et les utilisateurs Linux (Bloc 2) : c'est le même principe appliqué aux droits.

> ⚠️ **Un secret en plus** : les clés de la Leçon 4, les mots de passe de bases, les API keys… tout ce qui permet d'accéder à un service est un **secret** à protéger.

---

## 2.4 Le secret et l'erreur fatale : le mettre dans Git

Le réflexe d'un débutant est de mettre la config (avec le mot de passe) dans le code, puis de *committer*. C'est la pire chose qui soit : une fois dans l'historique Git, le secret **y reste à jamais**, même si tu le retires ensuite.

> 💡 **Analogie** : mettre son mot de passe dans le code, c'est coller son **code de carte bancaire sur la serrure** de l'entrée : c'est lisible par quiconque récupère le code (collaborateur, dépôt public, leak).

### Les bonnes façons de gérer un secret

| Méthode | Quoi | Quand |
|---------|------|-------|
| **Variable d'environnement** | `DB_PASSWORD=...` fourni au moment du lancement, pas dans le code | Dans un conteneur/script CI local |
| **Fichier `.env`** (jamais commité) | stocke les variables localement, ignoré par Git | Développement local |
| **Secrets CI/CD** (GitHub Actions, GitLab) | stockés dans l'outil, injectés pendant le pipeline | Pipelines (Bloc 11) |
| **Vault / AWS Secrets Manager / Kubernetes Secrets** | coffres-forts spécialisés pour la prod, avec rotation et audit | Production, échelle |

> 🔵 **À mentionner, définir, ne pas creuser — LDAP** : protocole d'annuaire centralisé d'utilisateurs/groupes (souvent couplé à Active Directory) pour authentifier les comptes d'une organisation.

---

## 📖 Mini-glossaire (à consulter avant les exemples)

> Définitions d'une ligne pour ne jamais être perdu(e).

- **Authentification** : vérifier **l'identité** d'un utilisateur (qui es-tu ?).
- **Autorisation** : vérifier **les droits** d'un utilisateur (que peux-tu faire ?).
- **RBAC** (Role-Based Access Control) : gestion des droits par **rôles** fixes.
- **ABAC** (Attribute-Based Access Control) : gestion des droits par **attributs** et règles conditionnelles.
- **Moindre privilège** : ne donner que ce qui est nécessaire, rien de plus.
- **Secret** : toute valeur sensible (mot de passe, clé API, certificat privé, token) qui permet d'accéder à un service.
- **Variable d'environnement** : valeur passée à un processus au lancement, hors du code.
- **`.env`** : fichier local de variables (à ne **jamais** committer).
- **`.gitignore`** : fichier Git listant ce qu'on ne versionne jamais (vu Bloc 4).
- **API key / token** : une clé/token qui identifie et autorise un appel à une API.
- **Rotation de secret** : le fait de **changer régulièrement** un secret (et révoquer l'ancien).
- **Audit** : traçabilité des actions (qui a fait quoi).
- **CI/CD** : intégration/déploiement continus (Bloc 11) — là où vivront les secrets de pipeline.
- **Vault / AWS Secrets Manager / Kubernetes Secrets** : coffres à secrets de production (survol).
- **LDAP / Active Directory** : annuaire centralisé d'utilisateurs (survol).

---
### 🧪 À faire maintenant (5 min) — protéger un projet avant d'écrire une ligne

> Objectif : créer le **réflexe « secret mis à l'abri AVANT de committer »**. Tu utilises un dossier de test.

```bash
mkdir projet-secret && cd projet-secret
git init

# 1) crée un .env (mot de passe FAUX, jamais un vrai !)
printf 'DB_PASSWORD=FAUX\nAPI_KEY=FAUX\n' > .env

# 2) AJOUTE un .gitignore avant tout commit
printf '.env\n*.key\n*.pem\n' > .gitignore

# 3) vérifie que .env est ignoré
git status --short
# .gitignore apparaît, mais PAS .env  →  c'est parfait
git check-ignore .env && echo "OK : .env est protégé"
```

**Ce que tu dois observer / écrire dans ta tête** :
- Le `git status` **ne montre pas** `.env` : Git l'ignore, grâce au `.gitignore` déjà en place.
- `git check-ignore .env` confirme **quelle règle** protège le fichier.
- **Ordre crucial** : on met le `.gitignore` **avant** d'ajouter quoi que ce soit — sinon on risque de committer le secret par mégarde.

---
## 3. Exemples concrets

### 3.1 Un `.env` local (à ne jamais committer)

```bash
# Fichier .env (à la racine du projet)
DB_HOST=localhost
DB_PORT=5432
DB_PASSWORD=P@ssw0rd-super-secret
API_KEY=sk-...
```

```bash
# On le charge depuis un script
set -a                # exporte chaque variable suivante
source .env           # charge le fichier .env
set +a                # arrête l'export automatique
echo "$DB_PASSWORD"   # peut afficher la variable (en pratique, ne pas logger)
```

### 3.2 Empêcher Git de les versionner (`.gitignore`)

```bash
# .gitignore
.env
*.key
*.pem
```

```bash
git add .gitignore
git commit -m "ignore secrets"
```
> 💡 **Rappel Bloc 4** : `.gitignore` dit à Git « ne jamais track ces fichiers ». Sans lui, tu risques de committer ton `.env` par erreur.

### 3.3 Passer un secret via une variable d'environnement

```bash
DB_PASSWORD="Toto123!" python3 mon-script.py   # valeur fournie au lancement
```

### 3.4 (Démo conceptuelle) Variables en CI/CD

Dans GitHub Actions / GitLab CI, on stocke le secret dans l'interface puis on le référence (on verra tout ça au Bloc 11) :
```yaml
env:
  DB_PASSWORD: ${{ secrets.DB_PASSWORD }}   # injecté par l'outil, jamais écrit en clair dans le code
```

---

## 4. Bonnes pratiques modernes (2025-2026)

- **Zéro secret dans le code** : ni en clair, ni même commenté — l'historique Git les garde à jamais.
- **Toujours un `.gitignore`** avant de committer, incluant `.env` et les fichiers de clés (`*.key`, `*.pem`).
- **Moindre privilège** partout : comptes, rôles, ports, permissions (rappel Bloc 2/3).
- **RBAC par défaut** ; n'envisager ABAC que si de vraies règles fines sont nécessaires.
- **Rotation des secrets** : changer régulièrement et révoquer ce qui est compromis.
- En prod : utiliser un **coffre à secrets** (Vault, AWS Secrets Manager, Kubernetes Secrets) avec **audit**.
- Ne jamais afficher un secret dans les logs.

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi | ✅ Version correcte |
|----------------|----------|---------------------|
| Commit du `.env` | Le secret vit dans Git pour toujours | `.gitignore` AVANT tout commit + rotation |
| Mot de passe en dur dans le code | Leak via le dépôt | Variable d'environnement / coffre |
| Le même compte admin partout | Catastrophe si compromis, pas d'audit | RBAC + comptes dédiés + moindre privilège |
| Donner tous les droits « pour simplifier » | Surface d'attaque énorme | Accorder le strict nécessaire |
| Révéler un secret dans les logs | Consultable par tous | Ne jamais logger les secrets |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`**, la correction dans **`03-correction.md`**.

**Énoncé court** : crée un projet (au bloc 4) avec un `.env` contenant de faux secrets, ajoute un `.gitignore`, vérifie avec `git status` que le `.env` n'est pas suivi, charge-le dans un script, et réponds à un quiz sur RBAC vs ABAC et moindre privilège.

---

## 7. Correction détaillée de l'exercice

> La correction complète pas-à-pas est dans **`03-correction.md`**. Le raisonnement :
> - le `.env` doit être créé **avec** son `.gitignore` pour ne jamais fuiter,
> - on vérifie avec `git status` / `git check-ignore` plutôt qu'en le committant,
> - on comprend que RBAC suffit souvent, et qu'un secret ne se partage jamais en clair.

---

## 8. Checklist de validation

- [ ] Je distingue authentification et autorisation.
- [ ] J'explique RBAC et ABAC, et je choisis selon le besoin.
- [ ] J'applique le principe de moindre privilège.
- [ ] Je n'ai aucun secret dans mon code/historique Git.
- [ ] J'utilise `.gitignore` pour protéger `.env` et clés.
- [ ] Je cite les bonnes solutions de secrets (env, CI/CD, Vault, AWS/K8s).

---

🧭 **Pont vers la suite** — Nous avons sécurisé le réseau, l'exposition et les droits. **La sécurité dans le code** (détecter les vulnérabilités tôt) et sa place dans le cycle de vie (DevSecOps) est le dernier pilier de ce bloc : c'est la Leçon 8.

---

*Prochaine étape :* Leçon 8 — **DevSecOps et Shift-Left** dans `08-DevSecOps-et-Shift-Left`.