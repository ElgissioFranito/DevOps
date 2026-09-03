# Leçon 4 — Du Git à la production

> **Bloc 1 · Bases du SDLC** — Leçon 4 sur 4 (synthèse du bloc)
> Cette dernière leçon relie **tout ce que tu as appris** : cycle de vie (L1), backlog (L2), tests & environnements (L3). Tu vas décrire le **parcours complet** d'une application, du code dans Git jusqu'au serveur de production, avec les échanges concrets (Git, build, tests, artifact, déploiement).

---

#### 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Expliquer** le parcours d'une application : `Git → Build → Tests → Artifact → Deployment → Production`.
2. **Replacer** chacune des notions vues en L1, L2, L3 dans ce parcours.
3. **Décrire** ce qu'on fait concrètement à chaque étape avec les commandes de référence (Spring Boot / Maven, variante NestJS / npm).
4. **Expliquer** pourquoi on déploie un **artifact identique** de staging en production.
5. **Présenter** ce parcours à voix haute, sans notes — le critère final du bloc.

---

#### 2. Explication simple

##### Le « pourquoi » : réunir toutes les pièces

Dans la roadmap, le critère pour valider le bloc 1 est : *« Tu peux expliquer sans hésiter comment ton code passe de ton ordinateur jusqu'au serveur de production. »*

Cette leçon est donc la **synthèse**. Chaque étape a déjà été étudiée :

```text
Git          (bloc 04 – contrôle de version)   ← ton code vit ici
 ↓
Build        (L3)          ← code source → artifact
 ↓
Tests        (L3)          ← unitaire / intégration / E2E
 ↓
Artifact     (L3)          ← le livrable déployable
 ↓
Deployment   (L2-L3)       ← envoyer l'artifact sur un environnement
 ↓
Production   (L1-L3)       ← les vrais utilisateurs
```

##### Le « comment » : le parcours pas à pas

1. **Git** : je pousse mon code (dans une branche) vers un dépôt Git central. Le code est le **point d'entrée** : tout part de lui.
2. **Build** : depuis le code, une commande automatise la transformation en artifact (ex. `mvn clean package` → `.jar`).
3. **Tests** : avant ou pendant le build, la montée de tests (unitaires, intégration, E2E) valide la qualité. En staging on fait les dernières vérifications.
4. **Artifact** : le résultat du build, **versionné et reproductible**. C'est *cet objet précis* qu'on va déployer, pas « le code » qu'on reconstruirait au dernier moment.
5. **Déploiement** : on installe l'artifact sur staging, on valide, puis **le même artifact** est promu en production.
6. **Production + Monitoring** : les vrais utilisateurs y accèdent. On surveille (L1 boucle de feedback).

##### Un point crucial : « le même artifact »

Le grand principe DevOps moderne : **on déploie en production l'exact même artifact qui a été validé en staging**. On ne **re-construit** pas le code sur la machine de prod — sinon le résultat pourrait être différent (« ça marchait sur mon PC ») et l'incertitude revient.

> 💡 **Analogie** : c'est comme la recette d'un plat livré. Tu ne refais pas la cuisine dans chacun des restaurants de service : tu livres le plat **préparé à l'avance et validé** par dégustation en cuisine d'essai.

##### La différence entre développement et production

- **Développement** : sur mon ordinateur, code, tests rapides, tout peut casser.
- **Production** : sur un serveur partagé, artifact validé, vrais utilisateurs, vraies données → on est **prudent** : on déploie l'artifact qui a déjà fonctionné.

---

#### 3. Exemples concrets

##### Exemple 1 — Le parcours complet (Spring Boot / Maven)

```bash
# 1) GIT — je pousse mon code
cd mon-app
git add .
git commit -m "Ajout du panier"
git push origin main

# 2) BUILD — transforme le code en artifact (.jar)
mvn clean package
# → target/mon-app-1.0.jar   (l'ARTIFACT)

# 3) TESTS — inclus dans le build (Maven exécute les tests avant de "package")
mvn test

# 4) DEPLOY en STAGING (démo : on copie l'artifact)
scp target/mon-app-1.0.jar user@staging:/opt/mon-app/mon-app.jar
ssh user@staging "cd /opt/mon-app && ./mon-app.jar"   # lance en staging

# 5) VÉRIFICATION staging
curl http://staging.exemple.com/health/ping   # → PONG si OK

# 6) DEPLOY en PRODUCTION (LE MÊME artifact)
scp target/mon-app-1.0.jar user@prod:/opt/mon-app/mon-app.jar
ssh user@prod "cd /opt/mon-app && ./mon-app.jar"

# 7) MONITORING
curl http://exemple.com/health/ping   # → PONG (les vrais utilisateurs servis)
```

> ⚠️ Les commandes `scp`/`ssh` manuelles ci-dessus sont **pédagogiques** : dans la réalité moderne, ce déploiement est **automatisé** (pipeline CI/CD, bloc 11). L'objectif ici : comprendre l'**ordre logique** des étapes.

##### Exemple 2 — La même chose pour une app Node / NestJS

```bash
git add . && git commit -m "Panier" && git push origin main   # GIT
npm ci && npm test                                             # INSTALL + TESTS
npm run build                                                  # BUILD → dossier dist/
npx tar -czf mon-app.tgz dist                                  # ARTIFACT (archivé)
# déploie l'artifact en staging, valide, puis le même en prod
```

##### Exemple 3 — Le schéma mental à retenir (à savoir redessiner)

```text
GIT ──► BUILD ──► TESTS ──► ARTIFACT ──► DEPLOY(STAGING) ──► DEPLOY(PROD) ──► MONITOR
 └────── code        │          │               │
                     └────── montée ──┘          └──── le MÊME artifact ────┘
```

---

#### 4. Bonnes pratiques modernes (2025-2026)

1. **Tout part de Git** : le code est versionné, le déploiement se fait à partir de Git (idéalement au bloc 13 GitOps).
2. **Déployer le même artifact** de staging vers production (reproductibilité).
3. **Automatiser Git → Build → Tests → Deploy** dans une **pipeline CI/CD** (bloc 11) pour ne rien faire à la main.
4. **Rendre le build reproductible** : mêmes versions de dépendances, builds fixes (lockfiles, versions épinglées).
5. **Ajouter un contrôle de santé (healthcheck)** dans staging et prod pour vérifier que l'artifact démarre bien.
6. **Chaque déploiement est versionné** : on sait toujours quelle version tourne en prod (adressable/rollback).

> 🧠 **Jargon** : **Pipeline CI/CD** = enchaînement automatisé (intégration continue / déploiement continu) qui fait Git→Build→Test→Deploy tout seul. **Rollback** = revenir à une version précédente si un déploiement pose problème.

---

#### 5. Pièges à éviter

| ❌ Anti-pattern | ⚠️ Pourquoi c'est dangereux | ✅ Version correcte |
|----------------|------------------------------|----------------------|
| **Re-construire** le code sur le serveur de prod | Résultat différent de celui testé → « ça marche chez moi » mais casse en prod. | Déployer l'**artifact unique** validé en staging. |
| Déployer du code **sans Git** (en direct) | On ne sait ni quoi, ni quand, ni comment revenir en arrière. | Tout passer par Git ; le déploiement vient d'une version enregistrée. |
| Sauter les tests avant de déployer | Bugs subis par les vrais utilisateurs (retour sur L3). | Tester (unitaire → E2E) avant de promouvoir. |
| Déployer « à la main » à chaque fois | Erreurs humaines, déploiements incohérents, non reproductibles. | Automatiser la pipeline (bloc 11). |
| Pas de version sur l'artifact | Impossible de savoir quelle version tourne ni de la retrouver. | Versionner chaque artifact (ex. `mon-app-1.0.jar`, tag Git, image taguée). |

---

#### 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`** et la correction dans **`03-correction.md`**.

**Énoncé court** : pour ta fonctionnalité (ex. : le panier), produis un document qui **explique le parcours Git → Production**, en réutilisant tout ce que tu as appris :

1. **Redessine le schéma** complet du parcours (Git → Build → Tests → Artifact → Deploy staging → Deploy prod → Monitoring).
2. Pour **chaque étape**, écris **1-2 phrases** : *qu'est-ce qu'on fait* et *avec quoi* (commande/outil).
3. **Explicite** le point clé : pourquoi on déploie **le même artifact** en prod.
4. **Réponds** à la question finale du bloc (à voix haute) :
   > *« Comment ton code passe de ton ordinateur au serveur de production ? »* — en 5-8 phrases, comme si tu l'expliquais à quelqu'un.

---

#### 7. Correction détaillée de l'exercice

> La correction complète pas-à-pas est dans **`03-correction.md`**. Essentiel du raisonnement attendu :

- Relier **toutes** les notions : déploiement part de Git, build produit un artifact, tests valident, le **même** artifact part en staging puis prod, monitoring boucle la boucle.
- Insister sur la **reproductibilité** : on ne recompile jamais en prod.
- Le discours « à voix haute » doit être fluide et tenir en quelques phrases claires, sans blocage.

---

#### 8. Checklist de validation

Coche chaque case que tu réussis :

- [ ] Je peux **redessiner** le parcours complet Git → Production, dans l'ordre.
- [ ] Je sais **replacer** Git, build, tests, artifact, staging et prod dans ce parcours.
- [ ] Je sais **expliquer** pourquoi on déploie le même artifact (pas de recompilation).
- [ ] Je peux **donner** les commandes de référence (Maven et/ou npm) pour build et tests.
- [ ] Je peux **expliquer à voix haute** (sans notes) comment mon code arrive en production.
- [ ] J'ai complété l'exercice et vérifié avec `03-correction.md`.

> 🎉 **Fin maîtrisée du bloc 1 si** après cette leçon, tu expliques le parcours sans hésiter — tu valides ainsi le critère « Bloc acquis » de la roadmap.

---

*Bloc 1 terminé ✅. Suite logique :* Bloc 02 — **Système d'exploitation — Linux**.