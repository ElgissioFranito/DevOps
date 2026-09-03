# Correction détaillée — Tests et environnements

> **Bloc 1 · Leçon 3** — Correction pas-à-pas de `02-exercice.md`. Nous corrigeons avec l'exemple du **panier d'achat** (une app Java Spring Boot, variante Node/NestJS).

---

## ✅ Étape 1 — Tes 5 tests (exemple)

### Tests unitaires (isolés, rapides)

```text
TU-1 → Vérifier que l'ajout d'un produit crée une ligne dans le panier.
      Attendu : « panier vide + 1 café = 1 ligne ».

TU-2 → Vérifier le calcul du total avec quantités.
      Attendu : « 2 cafés à 5 € → total = 10 € ».

TU-3 → Vérifier la suppression d'une ligne et la mise à jour du total.
      Attendu : « après suppression, le total retombe à 0 € ».
```

**Pourquoi c'est unitaire ?** Chaque test isole une **seule unité** (le panier) sans base ni réseau. Très rapide, exécutable sur le PC du développeur.

### Test d'intégration

```text
TI-1 → Vérifier que le panier interagit correctement avec la base de données
       (enregistrer une ligne, la relire, la supprimer).
      Attendu : « après ajout puis relecture depuis la base, on retrouve la ligne ».
```

**Pourquoi c'est de l'intégration ?** On fait travailler **plusieurs briques ensemble** (le code du panier + la base). Plus long, se fait dans un environnement partagé (dev ou staging).

### Test E2E

```text
E2E-1 → Parcours complet : un utilisateur ouvre la page produit, clique
        « Ajouter au panier », voit le panier avec le bon total, puis valide.
      Attendu : « du clic au résultat affiché, tout fonctionne de bout en bout ».
```

**Pourquoi c'est E2E ?** On teste le **parcours réel de l'utilisateur** dans l'application la plus proche de la production → staging.

---

## ✅ Étape 2 — Tests ↔ environnements

```text
Niveau test        Environnement naturel   Pourquoi
Unitaire           PC du développeur / dev  Rapide, isolé, pas besoin d'app complète.
Intégration        dev / staging             Nécessite plusieurs briques en place (base, API).
E2E                staging                   Nécessite l'app complète la plus réaliste.
Production         (aucun test "vrai")       On ne teste pas chez les vrais utilisateurs.
```

**Pourquoi** : les tests rapides restent dans les environnements de développement ; les tests qui touchent au système complet (intégration, E2E) montent vers staging, où l'app ressemble à la prod. **Production n'est pas un terrain de test.**

---

## ✅ Étape 3 — Le build de ta fonctionnalité

```text
Code source        Commande              Artifact produit
Spring Boot        mvn clean package     target/application.jar
Node / NestJS      npm run build         dist/ (dossier de fichiers web)
```

- **Build** = l'action : `mvn clean package` / `npm run build` (compiler, assembler, optimiser).
- **Artifact** = le résultat : le `.jar` reproductible, ou le dossier `dist/`.

> 💡 **Attention** : ne pas confondre **build** (verbe/action, transforme le code) et **artifact** (nom, le résultat produit). C'est un piège fréquent.

---

## ✅ Étape 4 — Pourquoi staging avant la production ?

> « Le staging est une **répétition générale** de la production. S'il n'existait pas, on découvrirait sur le vif les problèmes d'intégration, de configuration ou de versions — chez les vrais utilisateurs, avec de vraies données, sans filet. En passant d'abord par staging (le plus fidèle possible à la prod), on valide l'application et son déploiement dans un environnement privé : le risque de casser la production est fortement réduit. Seule ce qui est validé en staging est ensuite promu en production. »

**Pourquoi c'est la bonne réponse** : elle relie staging au principe de **réduction du risque** (Leçon 3), mentionne « le plus proche possible de la production » et explique l'impact négatif du saut direct en prod.

---

## ✅ Étape 5 — Auto-vérification (réponses types)

1. **Mon test unitaire est-il isolé ?** → Oui si le test unitaire du panier n'appelle ni base ni réseau. Si tu as appelé la base dans un « unitaire », c'est en réalité un test d'intégration.
2. **Mon E2E décrit-il un parcours complet ?** → Oui si c'est « l'utilisateur clique → voit le résultat ». Un E2E n'est pas une seule fonction isolée.
3. **Ai-je distingué build et artifact ?** → build = l'action (commande), artifact = le résultat (.jar / dist/).

---

## 📝 Checklist de validation (récapitulatif + conseils)

- [ ] Je sais **distinguer** test unitaire / intégration / E2E, avec un exemple de chacun.
- [ ] Je sais **placer** chaque type de test dans son environnement (PC/dev, staging, prod).
- [ ] Je sais **définir** build et artifact, avec un exemple Maven et un exemple npm.
- [ ] Je sais **expliquer** le rôle du staging pour réduire le risque en production.
- [ ] J'ai complété l'exercice et vérifié mes réponses avec cette correction.

### 💡 Conseils pour la suite

- **Munition de vocabulaire** : test unitaire, intégration, E2E, staging, build, artifact — sers-t'en à voix haute pour t'assurer que tu les maîtrises.
- Ces concepts seront **remis en pratique** au bloc **04 (Git)** et surtout au bloc **11 (CI/CD)**, où la montée des tests et le build seront automatisés.
- Tu vas maintenant **relier tout ce que tu as appris** dans la Leçon 4 : le parcours complet, du dépôt Git jusqu'à la production.

---

*Prochaine étape :* Leçon 4 — **Du Git à la production** → dossier `04-Du-Git-a-la-Production/`.