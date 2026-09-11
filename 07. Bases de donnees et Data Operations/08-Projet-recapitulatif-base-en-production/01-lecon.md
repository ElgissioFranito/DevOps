# Leçon 8 — Projet récapitulatif : la base en production

> **Bloc 7 · Bases de données & Data Operations** — Leçon 8 sur 8 (la synthèse)
> 🧭 **Pont depuis la Leçon 7** : ta base sait maintenant être **sécurisée** (2), **réglée** (3), **sauvegardée et restaurée** (4), **migrée proprement** (5), **disponible** (6) et **rapide** (7). Il ne manque qu'une chose : la **preuve assemblée**. Le critère de validation de la roadmap pour ce bloc : *« mettre une base PostgreSQL en production, la sauvegarder, la restaurer, effectuer une migration et expliquer comment éviter une perte de données »*. Ce projet est **la transformation des connaissances en livrable** : le **runbook** — le carnet de conduite d'un service, comme le Gabarit des projets récapitulatifs des blocs 2 et 4.

---

## 1. Objectifs d'apprentissage

À la fin de ce projet, tu seras capable de :

1. **Assembler** les 7 leçons en une mise en production **cohérente et documentée**.
2. **Écrire un runbook** : le carnet de conduite (architecture, rôles, sauvegarde, restauration, bascule, cache) qu'un collègue pourrait **suivre sans toi**.
3. **Vivre le scénario complet** de l'incident : panne → sauvegarde → restauration → remise en service → **vérification applicative**.
4. **Justifier** tes choix par **les chiffres** : RTO/RPO (Leçon 6), prix du miss (Leçon 7), seuil du journal (Leçon 3).
5. **Expliquer comment éviter une perte de données** — en une réponse de 5 lignes, prouvée par ce que tu as vécu.

---

## 2. Explication simple : la carte du voyage

Le bloc est un voyage en 7 étapes — le projet les **relit toutes** :

```
Leçon 1 : le SGBD et PostgreSQL        (je sais ranger et interroger)
Leçon 2 : rôles, permissions, connexions (je sais QUI peut toucher à quoi — le moindre privilège)
Leçon 3 : configuration et ressources   (je sais mesurer et régler — la boucle observer → régler → vérifier)
Leçon 4 : backup et restauration        (je sais survivre à la perte — et je l'ai vécue)
Leçon 5 : migrations versionnées        (je sais faire évoluer le schéma sans improviser)
Leçon 6 : réplication et HA             (je sais répondre à la panne du SERVEUR — avec RTO/RPO)
Leçon 7 : cache applicatif Redis        (je sais rendre le service RAPIDE — sans le rendre fragile)
```

### 2.1 L'architecture finale (le fil rouge assemblé)

```
                    Internet
                       ↓
              Load Balancer (Bloc 5/6)
                       ↓
        Application Spring Boot (le rôle app_biblio)
          ↓                        ↓
    cache Redis (les réponses        → PostgreSQL PRIMARY (guichet 5432)
    fréquentes — post-it)                 ├── backup pg_dump chaque nuit (cron)
                                          ├── archivage WAL (le carnet → PITR)
                                          ├── migrations Flyway versionnées dans Git
                                          └── RÉPLICA (le sous-chef — lectures/bascule)
```

**Pourquoi chaque brique existe** (la question de validation du bloc 6, qui revient ici) : l'application n'a **jamais** le compte admin (Leçon 2) ; les réponses fréquentes ne repassent **pas** par la base (Leçon 7) ; la base est **réglée et observée** (Leçon 3) ; toute perte est **récupérable** (Leçon 4) ; le schéma est **reproductible** (Leçon 5) ; la panne du serveur est **répondue en minutes** (Leçon 6).

### 2.2 Le « comment » : le runbook, la preuve écrite

Un **runbook** (littéralement « livre de conduite ») est le document qui décrit **comment le service tourne** et **que faire quand il tourne mal**. Le format attendu du projet (l'aide-mémoire `04-commandes-references.md` t'en donne le gabarit complet) :

```
runbook-bibliotheque.md
  1. Architecture (le diagramme + pourquoi chaque brique)
  2. Qui accède à quoi (rôles + droits + connexions)
  3. L'observation (les requêtes de santé + les seuils)
  4. La sauvegarde (la fréquence, les 2 dumps, le 3-2-1)
  5. La restauration (la procédure PAS À PAS, testée)
  6. Les migrations (la procédure + le rituel de backup)
  7. Le plan de reprise (RTO/RPO + la bascule + le test)
  8. Le cache (la grille par donnée + la règle d'or)
  9. Les incidents (2 scénarios narrés : « trop de clients », « plus rien ne s'écrit »)
```

> 💡 **Analogie** : le runbook est au service ce que le **carnet de vol** est au pilote : ce qui transforme « je sais piloter » en « je peux prouver comment, et quelqu'un d'autre peut le refaire ».

### 2.3 Le « quand » : la mise en production, puis la vie

La mise en production (ce projet) n'est pas une fin : c'est **le début de la vie du service** — la sauvegarde continue (cron), les migrations à chaque évolution, les tests de restauration mensuels, les bascules trimestrielles. Le runbook **vit avec le service** ; c'est aussi le pont naturel vers le bloc suivant : l'**IaC** (Bloc 8) rendra tout ce carnet **exécutable en code** (Terraform).

---

## 📖 Vocabulaire / Abréviations

- **Runbook** : le carnet de conduite d'un service (architecture, procédures, incidents) — qu'un collègue peut suivre **sans toi**.
- **Mise en production** (ou « mise en prod ») : rendre le service **réellement accessible** aux utilisateurs.
- **Vérification applicative** : tester avec les **clients** du service (les rôles applicatifs), pas seulement avec l'admin — « la base restaurée » ≠ « le service rendu » (Leçon 4).
- **Procédure pas à pas** : les étapes **numérotées** d'une opération — on la suit, on n'improvise pas à 3 h du matin.
- **Seuil** : la valeur d'un indicateur qui déclenche l'alerte (ex. : taux de cache < 95 %, lag > 10 Mo, journal > 500 ms).
- **Incident** : l'événement qui interrompt ou dégrade le service — narré, pas juste ressenti.
- **Post-mortem** : le récit de l'incident **après coup** (que s'est-il passé, pourquoi, ce qu'on corrige) — une pratique du Bloc 12.
- **Défaillance unique** (SPOF) : le composant unique dont la panne arrête tout (la Leçon 6 t'a appris à ne pas en créer au cache — Leçon 7).
- **Test mensuel de restauration** : le réflexe de la Leçon 4 — *« un backup non testé n'est pas un backup »*, **planifié**.
- **Test trimestriel de bascule** : le réflexe de la Leçon 6 — la chaîne détecter→élire→promouvoir→rediriger, **répétée**.
- **IaC** (*Infrastructure as Code*, Bloc 8) : rendre l'infrastructure **exécutable en code** (Terraform) — le pont de la fin du bloc.

---

## 3. Mise en pratique

> 🔁 **Comment s'articulent les fichiers de cette leçon** : la théorie est terminée — c'est maintenant que tu **assembles**. L'énoncé complet du projet est dans **`02-exercice.md`** (à faire en **une seule passe** — c'est le livrable final du bloc). La correction → `03-correction.md` (le runbook type + la grille d'auto-évaluation). Et le gabarit copiable → `04-commandes-references.md` (le squelette du runbook à remplir).

---

## 4. Bonnes pratiques modernes (2025-2026)

1. **Le runbook est le produit** : pas la base toute seule — le **carnet de conduite**. Ce qu'un collègue peut suivre sans toi, c'est la vraie mise en production.
2. **Chaque brique a sa raison** : dans le diagramme, chaque ligne répond à une question de validation d'une leçon. Le bloc 6 l'a enseigné (pourquoi chaque service existe) — applique-le à chaque composant de la base.
3. **Prouver, pas devenir** : le projet exige des **vérifications applicatives** (les rôles clients), pas seulement « l'admin peut lire ». C'est la leçon clé du Leçon 4 vécue une fois de plus.
4. **Les chiffres décident** : RTO/RPO (Leçon 6), prix du miss (Leçon 7), seuils du journal (Leçon 3) — les choix dans le runbook sont **chiffrés et justifiés**, pas des impressions.
5. **Le runbook vit** : les dates de test (restauration mensuelle, bascule trimestrielle) sont **planifiées** dans le document — une promesse de maintien.
6. **Ce qui est écrit est versionné** : le runbook se range dans Git avec le code (Bloc 4) — une modification de l'architecture **commence** par une modification du runbook.

---

## 5. Pièges à éviter

### Piège 1 — Livrer « la base » sans le carnet

```
❌ MAUVAIS : un dump + un README de 3 lignes   →  « voilà, la base est prête »
   → un collègue ne sait pas QUI a le droit, ni que faire à 3 h du matin.

✅ CORRECT : le runbook en 9 points (gabarit 04) + les vérifications applicatives
   → le service peut être **maintenu** par quelqu'un d'autre.
```

**Pourquoi** : livrer sans runbook, c'est livrer une **dépendance à toi** — exactement le SPOF humain.

### Piège 2 — Le backup dans le runbook, mais jamais testé

```
❌ MAUVAIS : « une restauration est prévue chaque mois » — dans le texte, pas dans le calendrier
✅ CORRECT : la date du **prochain** test de restauration est écrite dans le runbook
   (et l'expérience de restauration du Leçon 4 a déjà été vécue).
```

**Pourquoi** : la Leçon 4 l'a prouvé — *« un backup non testé n'est pas un backup »*. Le runbook qui ne **planifie** pas le test est un souhait, pas un plan.

### Piège 3 — Justifier sans chiffres

```
❌ MAUVAIS : « on a choisi la réplication synchrone pour être tranquille »
✅ CORRECT : « synchrone — RPO = 0 exigé (paiement) ; RTO = 60 s (failover auto) »
```

**Pourquoi** : c'est le principe de la Leçon 6 — les chiffres décident, pas l'impression. Le runbook qui ne chiffre pas ses choix est un **avis**, pas une décision.

### Piège 4 — L'admin partout dans le runbook

```
❌ MAUVAIS : toutes les procédures sont « sudo -u postgres »
✅ CORRECT : chaque procédure utilise le **rôle dédié** (Leçon 2)
   — et le runbook dit OÙ le compte admin est rangé (Leçon 2 : coffre de secrets).
```

---

## 6. Structure (comment tout s'emboîte — la carte du bloc)

```
GIT (Bloc 4 : le versionnement)
 └── runbook-bibliotheque.md          ← LE LIVRABLE FINAL (ce projet)
       ├── Leçon 2 : rôles + droits + connexions (la sécurité)
       ├── Leçon 3 : seuils + observation (la santé)
       ├── Leçon 4 : dumps + PITR + test de survie (la perte)
       ├── Leçon 5 : migrations Flyway (l'évolution)
       ├── Leçon 6 : réplication + RTO/RPO + bascule (la panne)
       └── Leçon 7 : grille cache + règle d'or (la performance)

PRODUCTION (ce que le rend possible)
       └── Bloc 8 (IaC) : ce même carnet, exécutable en code (Terraform)
```

---

## 7. Checklist de validation (LE critère du bloc)

À la fin de ce projet, tu dois pouvoir cocher **chaque** ligne :

- [ ] J'ai **écrit** un runbook de la base `bibliotheque` en 9 points (gabarit de `04-commandes-references.md`).
- [ ] Le runbook explique **pourquoi chaque brique** existe (la question de validation du bloc 6).
- [ ] Les accès utilisent les **rôles dédiés** (`app_biblio`, `lecteur_biblio`) — **jamais** l'admin en production.
- [ ] Les choix sont **chiffrés** : RTO/RPO (Leçon 6), TTL et prix du miss (Leçon 7), seuils (Leçon 3).
- [ ] J'ai **vécu le scénario complet** : panne → sauvegarde → restauration → remise en service → **vérification applicative** (les rôles clients, pas seulement l'admin).
- [ ] J'ai **effectué une migration** versionnée (Flyway `Vn__...`) avec le rituel de backup d'avant-migration.
- [ ] Le runbook **planifie** les tests à venir (restauration mensuelle, bascule trimestrielle).
- [ ] Je peux **expliquer comment éviter une perte de données** en 5 lignes, prouvées par ce que j'ai vécu.

---

> 🧭 **La réponse à la question de validation** (à rédiger dans ton runbook) :

```
« Une perte de données est évitée par COUCHES :
  1. Le REFUS PAR DÉFAUT (Leçon 2) : seuls les rôles nécessaires écrivent — pas de DELETE accidentel par un compte trop large.
  2. Le WAL + ARCHIVAGE (Leçon 4) : chaque écriture est journalisée AVANT d'être validée — survivre à un crash, restaurer à la seconde (PITR).
  3. Le BACKUP TESTÉ (Leçon 4) : deux dumps (base + rôles), testés par restauration réelle, copiés hors-site (3-2-1).
  4. La RÉPLICATION (Leçon 6) : le sous-chef prend le relais en minutes — la panne du serveur ne tue pas le service.
  Aucune couche ne suffit seule (la réplication copie les erreurs, le backup seul ne rend pas le service) — c'est leur EMPILEMENT qui supprime la perte. »
```

---

## 8. Prochaine étape

> 🎉 **Le Bloc 7 est complet** : tu sais créer, sécuriser, régler, sauvegarder, migrer, répliquer et accélérer une base de données PostgreSQL — et tu peux le **prouver** par un runbook vécu. La roadmap considère le bloc acquis si tu peux « mettre une base PostgreSQL en production, la sauvegarder, la restaurer, effectuer une migration et expliquer comment éviter une perte de données » — c'est ce que tu viens de démontrer.

> 🧭 **Pont vers le bloc suivant** — Le **Bloc 8 — Infrastructure as Code (IaC)** : jusqu'ici, tu as tout construit **à la main** (commandes, fichiers, cron). Le bloc suivant transforme ce carnet en **code exécutable** : **Terraform** créera le réseau, la machine, le stockage, la base managée (RDS) et les accès — **à partir du même runbook que tu viens d'écrire**. Ce que tu as décrit en mots au Bloc 7 devient du code au Bloc 8. Tu es prêt.