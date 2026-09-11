# Correction — Leçon 8 : DevSecOps et Shift-Left

> **Bloc 5 · Leçon 8** — Correction pas à pas.

---

## Étape 1 — (Option A) Lire un rapport de dépendances

```bash
mkdir sast-demo && cd sast-demo && npm init -y
npm install lodash@4.17.20
npm audit --json
```

**Explication** : `npm audit` compare tes dépendances à la base de CVE connues. `--json` donne une sortie lisible pour un script. Le rapport liste : la **CVE**, sa **sévérité** (critical/high/medium/low), le **paquet** vulnérable et la **version corrigée**.

**À observer** :
- une ligne `critical` ou `high` sur `lodash` ;
- l'identifiant `CVE-...` ;
- la recommandation de mise à jour → c'est le **patch**.

> ⚠️ En industrie, si une vulnérabilité **critique** est détectée, le pipeline **bloque le déploiement** tant que le patch n'est pas appliqué.

## Étape 2 — (Option B) Chercher des secrets

```bash
grep -rn "password\|API_KEY\|secret" --exclude-dir=.git .
```
**Explication** : `-r` = récursif, `-n` = avec numéro de ligne, `--exclude-dir=.git` = ne pas fouiller l'historique. On repère les mots-clés sensibles. En production, un outil dédié (**gitleaks**, **trufflehog**, **GitGuardian**) fait ça automatiquement à chaque `git push`.

## Étape 3 — Classer les scans

1. Injection SQL dans le code → **SAST** (analyse du code source).
2. Librairie `log4j` vulnérable → **dependency scanning**.
3. Image Docker → **container scanning**.
4. Application lancée face à des attaques → **DAST** (app en fonctionnement).
5. Clé API dans Git → **secret scanning**.

## Étape 4 — Réflexion

> **Pourquoi avant la production (shift-left) ?** Corriger tôt est moins cher et moins risqué (analogie plomberie de la leçon). Une faille en prod est déjà exploitée/exploitable et force une urgence ; détectée au commit, elle est corrigée avant qu'elle ne parte.

---

## Checklist de validation (leçon 7)

- [ ] J'explique le shift-left et ses bénéfices.
- [ ] Je définis DevSecOps.
- [ ] Je classe un cas dans la bonne famille de scan.
- [ ] Je lis une CVE et sa sévérité dans un rapport.
- [ ] Je sais que le pipeline bloque sur vulnérabilité critique.

---

## 🧠 Conseils pour la suite

- **Garde la liste d'outils** (SAST/DAST/…) : elle revient au Bloc 11 (CI/CD).
- **Ne commit jamais de secret** ; si c'est fait, rotation immédiate (Leçon 7).
- La sécurité automatique est un des frères d'œuvre du DevOps : c'est le socle que tu mettras en pipeline plus tard.