# Correction détaillée — Du Git à la production

> **Bloc 1 · Leçon 4** — Correction pas-à-pas de `02-exercice.md`. Nous corrigeons avec l'exemple du **panier d'achat** (Spring Boot, variante NestJS).

---

## ✅ Étape 1 — Schéma complet (au moins ça)

```text
GIT ──► BUILD ──► TESTS ──► ARTIFACT ──► DEPLOY(STAGING) ──► DEPLOY(PROD) ──► MONITOR
 ```

**À enrichir avec les notions L1/L2/L3** : ce parcours est la partie « Code → Build → Test → Deploy » du cycle (L1) ; chaque fonctionnalité vient du backlog (L2) ; les tests et environnements sont ceux de la L3 ; le monitoring remonte le feedback au début du cycle (L1).

---

## ✅ Étape 2 — Explication de chaque étape (réponses types)

1. **Git** : « Je pousse mon code (branche, puis fusion sur `main`) vers le dépôt Git central. Git versionne **quoi** et **quand** ; il permet aussi de revenir en arrière. » Commande : `git add / commit / push`.
2. **Build** : « Je transforme le code source en livrable exécutable avec une commande de build. » Avec Maven : `mvn clean package` → `.jar` ; avec npm : `npm run build` → `dist/`.
3. **Tests** : « Je vérifie la qualité : tests unitaires (rapides, isolés), intégration, et E2E sur staging. » Avec `mvn test` ou `npm test` (et dans la CI, bloc 11).
4. **Artifact** : « Le résultat du build, versionné et reproductible (ex. `mon-app-1.0.jar`), est l'objet déployable. On le range (repository d'artifacts). »
5. **Déploiement staging** : « J'installe cet artifact sur staging — la copie fidèle de la prod avec de fausses données — je valide via un healthcheck (ex. `curl .../health/ping`). »
6. **Déploiement production** : « Je déploie **le même artifact** en production. Les vrais utilisateurs sont servis ; je vérifie à nouveau le healthcheck. »
7. **Monitoring** : « Je surveille (erreurs, temps de réponse) pour détecter un problème et fermer la boucle de feedback du cycle (retour au développement). »

---

## ✅ Étape 3 — Le point clé du « même artifact »

> « En production, on ne re-compile **pas** le code. On reconstruire produirait un artifact peut-être **différent** de celui testé (compilateur, versions, environnement) — c'est le piège du "ça marche chez moi". Au contraire, on **déploie l'exact artifact** qui a passé les tests en staging : on transporte un objet déjà validé, on n'en recrée pas un. C'est ce qui rend la mise en production **prévisible et sûre**. »

**Pourquoi c'est la bonne réponse** : elle donne le *pourquoi* (reproductibilité), nomme le piège (« ça marche chez moi ») et le bénéfice (prévisibilité / sûreté).

---

## ✅ Étape 4 — Discours final (exemple de 7 phrases fluides)

> « Mon code part de Git : je le pousse vers un dépôt central, c'est le point de départ de tout. Ensuite le pipeline le construit : la commande de build transforme le code source en un artifact déployable, par exemple un .jar _. Le build exécute aussi les tests : unitaires d'abord, puis intégration, pour valider que ça marche. Une fois les tests passés, j'obtiens un artifact propre et versionné. Je le déploie d'abord en staging, qui est une copie de la production, et je vérifie qu'il démarre correctement. Puis je promus le **même artifact** en production, où les vrais utilisateurs l'utilisent. Enfin, je surveille l'application en production pour repérer tout problème et faire remonter le besoin au début du cycle. »

**Ce qu'on attend de toi** : l'ordre (Git → build → test → artifact → staging → prod → monitoring) et l'idée du **même artifact**. Le vocabulaire exact est secondaire, la cohérence est essentielle.

---

## ✅ Étape 5 — Auto-vérification (réponses)

1. **Toutes les étapes dans l'ordre ?** → Oui si ma phrase enchaîne Git→Build→Test→Artifact→Staging→Prod→Monitoring sans en sauter une.
2. **Quoi + avec quoi ?** → Oui si j'ai dit « `mvn clean package` → `.jar` », « `curl /health` », etc., pas juste le mot « build ».
3. **Même artifact justifié ?** → Oui si j'ai expliqué qu'on **ne recompile pas** en prod pour rester reproductible.

---

## 📝 Checklist de validation (récapitulatif + conseils)

- [ ] Je peux **dessiner** Git → Build → Tests → Artifact → Staging → Prod → Monitoring dans l'ordre.
- [ ] Je peux **donner** les commandes clés (Maven et/ou npm) pour build et tests.
- [ ] Je sais **justifier** le déploiement du même artifact.
- [ ] Je peux **expliquer à voix haute**, sans notes, le parcours complet (5-8 phrases fluides).
- [ ] Je relie ce parcours aux Leçons 1, 2 et 3 (cycle, backlog, tests/environnements).

### 💡 Conseils pour la suite

- **Entraîne-toi à voix haute** plusieurs fois par semaine jusqu'à la fluidité — c'est exactement le critère de validation du bloc.
- Ce schéma **Git→Prod** est le fil rouge de toute la roadmap : il réapparaîtra à chaque bloc (Git 04, CI/CD 11, GitOps 13…).
- Tu as maintenant les **fondations conceptuelles**. Le prochain bloc (02 – Linux) te donnera le socle *pratique* pour manœuvrer ces étapes concrètement.

---

*🎉 Bloc 1 terminé et validé si tu sais expliquer le parcours sans hésiter. Suite :* Bloc 02 — **Système d'exploitation — Linux**.