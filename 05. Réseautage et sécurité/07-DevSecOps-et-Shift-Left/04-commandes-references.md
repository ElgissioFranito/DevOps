# Référence rapide — Leçon 7 : DevSecOps & Shift-Left

> Bloc 5 · Leçon 7 — Aide-mémoire.

## Concepts
- **Shift-left** : contrôler la sécurité le plus tôt possible (dès le code).
- **DevSecOps** : sécurité intégrée au développement + exploitation.
- **CVE** : identifiant public d'une faille connue.
- **Patch** : correctif qui ferme une faille.
- **Sévérité** : Critical / High / Medium / Low.

## Les 5 familles de scan
| Scan | Sur quoi | Équivalent concret |
|------|----------|--------------------|
| **SAST** | Code source (sans exécuter) | SonarQube, Semgrep, CodeQL |
| **DAST** | App en fonctionnement | OWASP ZAP, Burp |
| **Dependency** | Bibliothèques (`pom.xml`,`package.json`) | `npm audit`, Dependabot, Trivy |
| **Container** | Image Docker | Trivy, Snyk, Grype |
| **Secret** | Secrets dans Git | gitleaks, trufflehog, GitGuardian |

## Pipeline cible
```
Code → SAST → Dependency scan → Build → Container scan → Deploy
```
→ **Bloquer** si vulnérabilité **critique** détectée.

## Réflexes
- Scanner tôt, souvent, à chaque étape du code vers la prod.
- Ne jamais committer de secret.
- Patcher vite les CVE actives (leçon de Log4j).