# Leçon 7 — DevSecOps et Shift-Left

> **Bloc 5 · Réseautage et sécurité** — Leçon 7 sur 7
> 🧭 **Pont depuis les Leçons 1-6** : tu sais protéger le réseau (pare-feu, TLS), exposer proprement (proxy) et gérer les accès/secrets. Mais **la sécurité la plus efficace se joue tôt, dès le code** : on l'appelle le **shift-left** (déplacer le plus tôt possible). Cette leçon t'ouvre au **DevSecOps** : intégrer la sécurité tout au long du cycle de vie, avant que le code aille en production.
> 👉 C'est la dernière pierre du bloc et le pont naturel vers les blocs Docker (9) et CI/CD (11), qui mettront ces idées en pratique.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Expliquer** ce qu'est le **shift-left** et en quoi il est plus efficace (et moins coûteux) que de sécuriser seulement en production.
2. **Définir** **DevSecOps** et son objectif.
3. **Distinguer** les grandes familles de scans : **SAST**, **DAST**, **dependency scanning**, **container scanning**, **secret scanning**.
4. **Expliquer** ce qu'est une **CVE** et un **patch**, et lire un rapport de vulnérabilité (sévérité).
5. **Positionner** ces scans dans un pipeline CI/CD (et empêcher le déploiement si une vulnérabilité critique est détectée) — le concret arrivant au Bloc 11.

---

## 2. Explication simple

### 2.1 Le « pourquoi » : sécuriser le plus tôt possible (shift-left)

Deux façons de penser la sécurité :

```
Code → Production → problème découvert (cher à réparer)

Code → Scan dès le dev → Tests → Build → Production  (on détecte tôt, on répare vite)
```

> 💡 **Analogie** : corriger une faute de plomberie **pendant** la construction de la maison est trivial. La découvrir **après** l'emménagement, c'est percer les murs. Moins on avance dans le cycle de vie, moins c'est cher et risqué de corriger.

**Shift-left** = faire ces contrôles le **plus tôt possible** (pendant le développement), pas attendre la production.

### 2.2 DevSecOps (le « quoi »)

**DevSecOps = Development + Security + Operations.** L'idée : intégrer la sécurité **dans** le pipeline de développement dès le départ, au lieu d'en faire une étape séparée tardive.

```
Code → Git → CI → [Security Scan] → Build → [Docker] → Deploy → Production
```

> ℹ️ **Note (à garder en tête)** : cette leçon mentionne Docker et CI/CD que tu verras aux blocs 9 et 11. Retiens surtout les **principes** ici (scanner tôt, ne jamais committer de secret) ; la mise en œuvre concrète viendra dans ces blocs.

### 2.3 Les scans, par niveau (le « comment »)

| Scan | Quoi | Où / comment |
|------|------|--------------|
| **SAST** (Static Application Security Testing) | Analyse le **code source** sans l'exécuter | Très tôt, dès le dev/commit |
| **DAST** (Dynamic...) | Teste l'**application en fonctionnement** (noir boîte) | Sur une app démarrée, plus tard |
| **Dependency scanning** | Analyse les **bibliothèques/dépendances** (ex. `pom.xml`) | Dès le build, à chaque dépendance |
| **Container scanning** | Analyse une **image Docker** à la recherche de vulnérabilités | Au build d'image (Bloc 9) |
| **Secret scanning** | Cherche des **secrets** égarés dans Git | À chaque push / dans l'historique |

---

## 2.4 CVE, vulnérabilité et patch (le « quoi » et le « quand »)

- **CVE** (Common Vulnerabilities and Exposures) : **identifiant public unique** d'une vulnérabilité connue (ex. `CVE-2021-44228`, le fameux Log4j).
- **Vulnérabilité** : une **faille** dans un logiciel qui peut être exploitée.
- **Patch** : un correctif (mise à jour) qui ferme la faille.
- **Dépendance vulnérable** : une bibliothèque que ton projet utilise et qui contient une faille (`pom.xml`, `package.json`…).
- **Container vulnérable** : une image Docker dont une couche/logiciel a une faille.

> 💡 **Analogie** : un CVE est une **fiche d'avis de recherche** publique sur une faille ; une dépendance vulnérable, c'est une pièce de ta maison reconnue défectueuse ; un patch, c'est le remplacement de cette pièce.

Sévérité : un rapport indique souvent **Critical / High / Medium / Low** (critique / élevée / moyenne / faible). Un pipeline « sérieux » **bloque le déploiement** si une vulnérabilité **critique** est détectée.

### 2.5 Le « quand » et le pipeline type

```
git push
   ↓
Tests unitaires
   ↓
SAST                 ← analyse du code
   ↓
Dependency scan      ← analyse des dépendances
   ↓
Docker build
   ↓
Container scan       ← analyse de l'image
   ↓
Deploy               ← (stoppé si vulnérabilité critique)
```

> 💡 **Pourquoi pas tout en un seul scandage ?** Chaque niveau de scan voit des choses que les autres ne voient pas (le code vs les librairies vs l'image finale) : c'est pourquoi on les **cumule** le long du pipeline.

---

## 📖 Mini-glossaire (à consulter avant les exemples)

> Définitions d'une ligne pour ne jamais être perdu(e).

- **Shift-left** : déplacer les contrôles (dont sécurité) le **plus tôt** possible dans le cycle de vie.
- **DevSecOps** : intégrer la sécurité **dans** le développement et l'exploitation (Dev + Sec + Ops).
- **SAST** (Static Application Security Testing) : scan du **code source** sans l'exécuter.
- **DAST** (Dynamic Application Security Testing) : scan d'une **application en fonctionnement**.
- **Dependency scanning** : scan des **bibliothèques** utilisées.
- **Container scanning** : analyse une **image Docker** (conteneur) pour des failles.
- **Secret scanning** : recherche de secrets égarés dans le code/Git.
- **CVE** (Common Vulnerabilities and Exposures) : identifiant public d'une **vulnérabilité connue**.
- **Patch** : correctif/mise à jour qui ferme une faille.
- **Sévérité** : gravité d'une vulnérabilité (Critical/High/Medium/Low).
- **Pipeline** : enchaînement automatisé des étapes jusqu'à la production (Bloc 11, CI/CD).
- **Dépendance** : une bibliothèque externe utilisée par ton projet.
- **`pom.xml` / `package.json`** : fichiers listant les dépendances (Maven/Java, Node) — vus au Bloc 3.
- **Docker / `docker build`** : construire une image conteneurisée (Bloc 9) ; le scan d'image s'y applique.
- **Log4j** : bibliothèque Java célèbre pour la faille CVE-2021-44228 (exemple historique).
- **Git push** : envoyé ton code vers le dépôt (Bloc 4) ; déclenche souvent le pipeline.

---

### 🧪 À faire maintenant (5 min) — lire un vrai rapport de vulnérabilités

> Objectif : voir une vraie alerte de sécurité (CVE) en toutes sécurité. Option Node (recommandée si tu as Node) ou option grep.

**Option A — scan de dépendances (`npm audit`)**
```bash
mkdir audit-demo && cd audit-demo && npm init -y
npm install lodash@4.17.20      # une vieille version → volontairement vulnérable
npm audit                        # affiche une/des CVE avec sévérité
npx npm audit --json | head -40  # version détaillée
```

**Option B — recherche de secrets (dépôt de test)**
```bash
mkdir secret-demo && cd secret-demo && echo "API_KEY=AAAA" > config.js
grep -rn "API_KEY\|password" . --exclude-dir=.git
```

**Ce que tu dois observer / écrire dans ta tête** :
- `npm audit` liste une **CVE** (ex. `CVE-...`), sa **sévérité** (critical/high/medium) et la **version corrigée** (le patch).
- Le `grep` trouve les mots-clés de secrets → c'est ce qu'automatise un vrai **secret scanner**.
- **En industrie** : une alerte **critical** **bloque le pipeline** tant que le patch n'est pas appliqué.

> ⚠️ `lodash@4.17.20` sert à *démontrer une faille* dans un dossier de test — on ne l'utilise jamais dans un vrai projet. Ne **commit** pas ce dépôt de démo.

---

## 3. Exemples concrets (principes, exécutables aux Blocs 9/11)

### 3.1 Visualiser la chaîne de scans (schéma)
```
Code → SAST → Dependency Scan → Build → Container Scan → Deploy
```
Chaque étape garde la production si une alerte critique surgit.

### 3.2 Exemples d'outils (par type de scan)

| Type | Outils (survol) |
|------|-----------------|
| SAST | SonarQube, Semgrep, CodeQL (GitHub) |
| DAST | OWASP ZAP, Burp Suite |
| Dependency | `npm audit`, GitHub Dependabot, Trivy (souvent en YAML GitLab/GitHub) |
| Container | Trivy, Snyk, Grype |
| Secret | gitleaks, GitGuardian, trufflehog |

> 🔵 **À ne pas retenir par cœur** : ce sont des noms à **reconnaître**. Tu les retrouveras en vrai au Bloc 11 (CI/CD) et dans les outils.

### 3.3 (Démo conceptuelle) Ce que « bloquer le déploiement » signifie

Imagine le pipeline CI/CD (Bloc 11) :
```yaml
# Extrait simplifié d'un pipeline (concept)
- étape: dependency_scan
- si: vulnérabilité.critique_détectée == oui
  alors: STOP   # on n'essaie même pas de déployer
- sinon: continue vers Deploy
```
Le but : **empêcher** qu'une version vulnérable atteigne la production.

---

## 4. Bonnes pratiques modernes (2025-2026)

- **Scanner tôt et souvent** : SAST dès le commit, pas seulement avant la mise en prod.
- **Cumuler les scans** : chaque type (code, dépendances, image) couvre une surface différente.
- **Blocage sur criticité** : échouer le pipeline sur une vulnérabilité **critique** (Critical) ; faire un tri sur les non-bloquantes.
- **Zéro secret dans Git** : secret scanning à chaque push (rappel Leçon 6).
- **Tirer les alertes à jour** (Dependabot / Renovate) pour patcher vite, avant exploitation (l'exemple Log4j l'a montré brutalement).
- **Patcher rapidement les CVE actives en production** (leçon retenue de Log4j).
- **Automatiser** : la sécurité manuelle ne tient pas la cadence DevOps.

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi | ✅ Version correcte |
|----------------|----------|---------------------|
| Attendre la prod pour scanner | Corriger tard = cher et risqué | Shift-left : scanner dès le dev |
| Un seul type de scan | On rate les failles des dépendances/images | Cumuler SAST + depend + container |
| Ignorer les vulnérabilités moyennes | Les attaquants s'y attaquent vite | Trier par sévérité mais patcher |
| Committer un secret en espérant le supprimer après | L'historique Git le garde | Secret scanning + rotation |
| Déployer malgré une CVE critique | Surface d'attaque ouverte en prod | Bloquer le pipeline |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`**, la correction dans **`03-correction.md`**.

**Énoncé court** : sur un projet de test, lance un scan de dépendances avec `npm audit` (si Node) ou un scan de secrets local (gitleaks/trufflehog) sur un petit dépôt ; lis le rapport (sévérité, CVE) et entraîne-toi à classer les scans (SAST/DAST/dependency/container/secret) sur des exemples donnés.

---

## 7. Correction détaillée de l'exercice

> La correction complète pas-à-pas est dans **`03-correction.md`**. Le raisonnement :
> - un scanner de **dépendances** (`npm audit`) liste des CVE avec **sévérité** : on apprend à lire le rapport ;
> - un **secret scanner** détecte les mots clés (`API_KEY=`, mots de passe) égarés : on comprend en quoi c'est utile en continu ;
> - le questionnaire consolide **quelle famille de scan** pour quelle surface.

---

## 8. Checklist de validation

- [ ] J'explique le shift-left et pourquoi c'est moins coûteux.
- [ ] J'explique ce qu'est DevSecOps.
- [ ] Je distingue SAST, DAST, dependency, container, secret scanning.
- [ ] Je définis une CVE et un patch, et je lis une sévérité.
- [ ] Je positionne ces scans dans un pipeline (et je verrouille sur vulnérabilité critique).
- [ ] Je peux dire où la sécurité s'intègre avant la production.

---

🧭 **Fin du bloc 5** — Relis la checklist globale dans `00-Introduction-Bloc.md`. Tu sais maintenant : où circule une requête et à quels niveaux elle peut être bloquée (le critère « bloc acquis » de la roadmap), et comment intégrer la sécurité de l'accès au code. La suite logique de la roadmap : le **Bloc 6 — Cloud Providers**, où tout ce réseau/sécurité s'expose à grande échelle.

---

*Prochaine étape :* Bloc 6 — **Cloud Providers** (hors de ce dossier).