# Correction détaillée — Le cycle de vie d'une application (SDLC)

> **Bloc 1 · Leçon 1** — Correction pas-à-pas de `02-exercice.md`.

---

## 🎯 Rappel de l'exercice

Choisir une fonctionnalité simple et la dérouler sur les 8 cases du cycle (besoin → monitoring/maintenance), puis produire un schéma et répondre à 3 questions d'auto-vérification.

**Nous corrigeons avec l'exemple du panier d'achat.** Si tu as choisi une autre fonctionnalité, le **raisonnement** doit être identique — seuls les détails changent.

---

## ✅ Correction des 8 cases du cycle (exemple : panier d'achat)

### 1. Besoin

**Attendu :** le problème résolu et pour qui.

> « Les clients d'un site e-commerce veulent regrouper plusieurs produits avant de passer commande, afin de payer une seule fois. »

**Pourquoi c'est bien :** on exprime le problème (payer plusieurs commandes séparées est pénible), le public (clients e-commerce) et le bénéfice (payer une seule fois).

### 2. Analyse du besoin

**Attendu :** les attentes précises (qui, quoi, pourquoi).

> « En tant que client, je veux ajouter des produits à un panier, en modifier les quantités, supprimer une ligne, et voir le total, afin de commander en une fois. »

**Pourquoi c'est bien :** on liste les fonctions précises (ajouter, modifier, supprimer, total). C'est ici que sont détectées les **ambiguïtés** : peut-on commander un panier vide ? plusieurs fois ? etc.

### 3. Conception

**Attendu :** une solution technique grossière.

> « Une page de panier à l'écran, une structure de données `Panier`, une table en base `panier_lignes` (produit, quantité, prix), et une règle : un panier vide ne peut pas être commandé. »

**Pourquoi c'est bien :** on ne code pas encore, on décide des **briques** (écran, données, base, règles métier). C'est l'équivalent des « plans ».

### 4. Développement

**Attendu :** ce qu'on code concrètement.

> « On implémente l'ajout d'un produit au panier (crée la ligne ou augmente la quantité), le calcul du total (somme prix × quantité), et la suppression d'une ligne. »

**Pourquoi c'est bien :** on décrit des **comportements codables**, pas encore le code détaillé.

### 5. Tests

**Attendu :** ce qu'on vérifie pour être sûr que ça marche.

> « On vérifie : panier vide + 1 produit = 1 ligne, ajouter 2 fois le même produit double la quantité, le total = somme correcte, suppression = ligne supprimée, impossible de commander un panier vide. »

**Pourquoi c'est bien :** ce sont des **cas de test concrets et vérifiables**, dont certains négatifs (panier vide). Chaque cas protège contre une régression (voir Leçon 3).

### 6. Intégration

**Attendu :** comment le code s'assemble avec le reste.

> « On branche le panier à la page produit (le bouton « Ajouter »), à la page de commande, et à la base de données. On vérifie que l'ensemble fonctionne ensemble (test d'intégration). »

**Pourquoi c'est bien :** le code du panier est **inutile seul** ; il faut le raccorder aux autres briques et vérifier l'ensemble cohérent.

### 7. Déploiement

**Attendu :** comment la version arrive en production.

> « On compile le projet, on construit un artifact (un fichier déployable), puis on l'installe sur le serveur de production en remplaçant l'ancienne version en toute sécurité. »

**Pourquoi c'est bien :** on distingue **développement** (sur le PC) et **production** (sur le serveur). L'artifact est le « livrable » déployable (jargon vu dans la roadmap).

### 8. Production + Monitoring + Maintenance

**Attendu :** comment on surveille tout ça.

> « Les vrais clients utilisent le panier. On surveille les erreurs d'ajout au panier, le temps de réponse de la page, et le taux de paniers abandonnés. En cas d'anomalie, on corrige (maintenance) et on relance le cycle. »

**Pourquoi c'est bien :** le monitoring **ferme la boucle** du cycle (le feedback revient au début) et déclenche la maintenance.

---

## ✅ Schéma attendu (texte)

```text
BESOIN          Les clients veulent commander plusieurs produits en une fois.
  ↓
ANALYSE         Ajouter, modifier, supprimer, total. Commande impossible si vide.
  ↓
CONCEPTION      Une page panier + table panier_lignes + règle métier.
  ↓
DÉVELOPPEMENT   Code de l'ajout, du total, de la suppression.
  ↓
TESTS           Panier vide+1=1 ligne, doublon=double qté, total ok, delete ok, vide refusé.
  ↓
INTÉGRATION     Raccord page produit + page commande + base de données.
  ↓
DÉPLOIEMENT     Build → artifact → installé sur le serveur de production.
  ↓
PRODUCTION      Les vrais clients utilisent le panier.
  ↓
MONITORING      Erreurs, temps de réponse, paniers abandonnés → maintenance.
```

---

## ✅ Correction des 3 questions d'auto-vérification

1. **À quelle étape saute-t-on si on code directement sans écouter l'utilisateur ?**
   → L'**analyse du besoin** (et donc la **conception** correcte). On risque de construire une fonctionnalité que personne ne veut ou qui ne répond pas au vrai problème.

2. **Pourquoi les tests viennent-ils avant le déploiement ?**
   → Pour **éviter de casser la production** et de faire payer l'erreur à tous les vrais utilisateurs. Corriger un bug en développement coûte ~100× moins cher qu'en production.

3. **Que se passe-t-il si on néglige le monitoring après la mise en production ?**
   → L'application peut planter ou ralentir **sans que personne ne le voie** : les utilisateurs subissent le problème en silence, et on ne sait même pas qu'il faut corriger. Le monitoring est ce qui **ferme la boucle** du cycle.

---

## 📝 Checklist de validation (récapitulatif + conseils)

> Coche ce que tu maîtrises réellement.

- [ ] Je sais **définir le SDLC** et citer ses étapes dans l'ordre.
- [ ] Je sais **distinguer** SDLC (parcours) et DevOps (façon de l'exécuter efficacement).
- [ ] Je sais **dérouler** mes propres fonctionnalités sur les 8 cases du cycle.
- [ ] Je sais **justifier** pourquoi analyse et tests ne se sautent pas.
- [ ] Je sais **expliquer** le rôle du monitoring pour boucler le cycle.
- [ ] J'ai produit **mon schéma textuel** et il est cohérent avec la correction ci-dessus.

### 💡 Conseils pour la suite

- **Refais l'exercice** avec une fonctionnalité différente dans quelques jours pour ancrer le réflexe.
- Garde ton schéma : il servira de **fil conducteur** pour la Leçon 4 (« Du Git à la production »).
- Ne passe pas à la suite tant que tu ne peux pas **parler** de ton cycle à voix haute sans notes.

---

*Prochaine étape :* Leçon 2 — **Planification et backlog** → dossier `02-Planification-et-Backlog/`.