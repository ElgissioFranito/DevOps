# Exercice 6 — Trois pannes, trois métiers, un plan de reprise

> **Objectifs** : choisir une architecture de disponibilité en **chiffrant** RTO/RPO, diagnostiquer un lag, et rédiger un plan de reprise.
> **Durée** : ~1h · **Prérequis** : Leçons 1-5 (le Leçon 4 et son PITR reviendront !).
> **Livrable** : `notes-exercice-06.md` dans ce dossier.

## Rappel des configurations possibles

| Configuration | RPO | RTO |
|---|---|---|
| A. Backup nocturne seul (Leçon 4) | jusqu'à 24 h | heures (restauration manuelle) |
| B. Réplication asynchrone | secondes | court (promotion + redirection) |
| C. Réplication synchrone + failover auto | 0 | minutes |

---

## Scénario 1 — La boutique du 29 novembre

**Contexte** : e-commerce, 2 000 commandes/jour (10× ce jour-là), le **disque du primary meurt à 14h02** le jour de pointe. Le métier dit : « chaque commande non enregistrée = un client perdu ; et on ne peut pas perdre plus de 5 minutes de commandes. »

1. **Traduis** l'exigence métier en RTO et RPO (attention : « on ne peut pas perdre plus de 5 min de commandes » → RPO = ?).
2. Élimine les configurations **impossibles** (qu'est-ce qui viole le RPO ? Qu'est-ce qui viole le RTO ?).
3. **Choisis** (A, B ou C) et **justifie en 3 phrases**.
4. **Liste les 6 étapes** de la bascule (détecter → ... → re-cloner) et indique **qui** (humain ou outil) fait chacune dans ta configuration.

## Scénario 2 — L'intranet de 30 collaborateurs

**Contexte** : intranet RH, notes de frais et congés. Le métier dit : « on peut vivre une demi-journée sans l'outil, tant qu'on ne perd rien. » Budget : petit.

1. Chiffre RTO (demi-journée ok ?) et RPO (rien ?).
2. Le RPO « rien » semble exiger le synchrone... **mais** vérifie : à quelle fréquence les écritures arrivent-elles vraiment ? (un intranet RH écrit ~50 fois/jour).
3. **Choisis** et justifie — l'astuce ici : le **couple backup + asynchrone** peut suffire si tu le rédiges honnêtement. Que manque-t-il pour que le « rien perdu » soit vrai en pratique ? (indice : la Leçon 4, l'archivage WAL.)
4. **Écris la phrase** que tu mettrais dans le plan de reprise pour ce métier.

## Scénario 3 — Le rapport du lundi qui tue la base

**Contexte** : tous les lundis 9h, un `SELECT` d'agrégation de 10 millions de lignes tourne. Depuis qu'on l'a déplacé **sur le réplica**, les utilisateurs de l'application disent : « j'ai enregistré mon profil et il n'apparaît pas ! »

1. **Diagnostique** : quelle requête de la Leçon (vue en 3.2) lances-tu sur le primary, et que regardes-tu dedans ?
2. Explique le **phénomène** (pense à la séquence « l'utilisateur écrit sur le primary, puis relit... où ? »).
3. **Corrige l'architecture** : quelles requêtes passent par le primary, lesquelles par le réplica ? Formule la **règle** en une phrase (« les écritures X et leur relecture Y... »).
4. **Note le réflexe de surveillance** : quel seuil de lag te ferait alerter, et pourquoi ?

## Synthèse — Le plan de reprise (l'anti- improvisation)

Pour le scénario de ton choix, rédige le plan (une demi-page) avec :

1. **RTO** et **RPO** annoncés (avec l'unité !).
2. La **configuration retenue** (et pourquoi, en 3 lignes).
3. Les **étapes de bascule** numérotées (avec responsable : outil ou humain).
4. La **fréquence de test** du plan (et pourquoi une bascule jamais répétée échouera).
5. Le **filet anti-erreur-humaine** : rappelle pourquoi le backup + PITR (Leçon 4) reste indispensable **avec** la réplication.

## Question bonus (pour les curieux)

Le métier du scénario 1 ajoute : « et si le **datacenter entier** tombe ? » — quelle idée (du bloc 6, le managé) répond à cette question ? En une phrase.

> 🧭 **Astuce anti-frustration** : si un scénario te semble « impossible à trancher », c'est que le **métier n'a pas encore dit ses chiffres** — c'est la vraie leçon de l'exercice : le rôle du DevOps est de **faire émerger** RTO/RPO, pas de les deviner.