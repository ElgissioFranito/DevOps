# Exercice 8 — Projet récapitulatif : la base `bibliotheque` en production

> **Objectifs** : assembler les 7 leçons en un **service vivant** + un **runbook** qui le prouve.
> **Durée** : ~2h (en une seule passe, comme une vraie mise en prod) · **Prérequis** : Leçons 1-7 faites.
> **Livrables** : `runbook-bibliotheque.md` + `notes-exercice-08.md` dans ce dossier.

## Le contexte (le scénario de mise en prod)

La bibliothèque municipale veut **mettre en ligne** son catalogue : 5 bibliothécaires qui écrivent, des lecteurs qui consultent, une appli Spring Boot devant, une base PostgreSQL derrière. **Tu es l'admin.** Le directeur exige : *« si la base tombe, on la récupère — et je veux le voir. »*

## Étape 1 — Le socle (Leçons 1 et 2 : ranger + sécuriser)

1. Recrée proprement la base `bibliotheque_prod` (schéma + types + clés étrangères — Leçon 1).
2. Crée les rôles avec le moindre privilège (Leçon 2) :
   - `app_biblio` : lecture/écriture sur les tables applicatives (PAS de DROP, PAS de SUPERUSER).
   - `lecteur_biblio` : SELECT seul.
3. Vérifie `\du` et prouve par le test que chaque rôle peut ce qu'il doit — **et rien de plus**.
4. **Note dans le runbook** (§ 2) : chaque rôle, ses droits, et **pourquoi**.

## Étape 2 — L'observation (Leçon 3 : mesurer et régler)

1. Active le journal des requêtes lentes (`log_min_duration_statement = 500ms` — Leçon 3).
2. Charge le catalogue (au moins 500 lignes réalistes) et mesure : `EXPLAIN ANALYZE` + taux de cache (`pg_stat_database`).
3. Crée **au moins un index** justifié par une mesure réelle.
4. **Note dans le runbook** (§ 3) : les 3 requêtes de santé + **leurs seuils** d'alerte.

## Étape 3 — Le filet (Leçon 4 : survivre à la perte)

1. **Backup complet** : `pg_dump -Fc` (la base) + `pg_dumpall --globals-only` (les rôles) — horodatés.
2. **Simule la panne** (chez toi, sans risque) : `DROP TABLE membres;` — note le silence exact.
3. **Restaure à côté**, vérifie par les comptages, puis **remets en service** (`DROP DATABASE` → `CREATE` → `pg_restore`).
4. **Vérification applicative** (le moment « aha » de la Leçon 4) : reconnecte-toi avec `app_biblio` et `lecteur_biblio` — **le service fonctionne-t-il ?**
5. **Note dans le runbook** (§ 4-5) : la fréquence des backups, les 2 dumps, le 3-2-1, et la **procédure pas à pas de restauration** que tu viens de vivre.

## Étape 4 — L'évolution (Leçon 5 : migrer sans improviser)

1. **Backup d'abord** (le rituel — non négociable).
2. Écris et applique **une migration Flyway** (`Vn__...`) : ajoute une colonne utile (ex. `date_retour_prevue` à `emprunts`), avec les `GRANT` embarqués.
3. `flyway info` + `validate` : documente l'état final.
4. **Note dans le runbook** (§ 6) : la procédure de migration + le rituel de backup.

## Étape 5 — La panne du serveur (Leçon 6 : décider, pas juste subir)

Le directeur demande : *« et si le SERVEUR tombe ? »*
1. **Chiffre** le RTO (durée d'interruption acceptable) et le RPO (données perdues acceptables) **pour ce projet** — avec l'unité.
2. **Choisis** : backup + PITR suffit-il, ou faut-il une réplication ? Justifie en 3 lignes (coût, métier, budget).
3. **Note dans le runbook** (§ 7) : RTO/RPO, configuration retenue, étapes de bascule, fréquence de test.

## Étape 6 — La performance (Leçon 7 : le cache, sans fragilité)

1. Installe Redis, cache **une** requête coûteuse (la liste des livres populaires) avec TTL.
2. Vis et corrige le stale data (invalidation explicite au `UPDATE`).
3. Prouve la règle d'or (Redis arrêté → l'application continue).
4. **Note dans le runbook** (§ 8) : la grille par donnée (quoi / TTL / invalidation).

## Étape 7 — Le récit (le runbook complet)

1. Termine `runbook-bibliotheque.md` en 9 points (gabarit de `04-commandes-references.md`).
2. Rédige les **2 scénarios d'incidents narrés** (§ 9) : « trop de clients » et « plus rien ne s'écrit » — chacun avec **symptôme → diagnostic → correction** (tu les as tous vécus dans le bloc).
3. Écris la réponse de 5 lignes à « comment éviter une perte de données » (section 7 de `01-lecon.md`).
4. Range le tout dans Git (Bloc 4) — le runbook est versionné avec le code.

> 🧭 **Astuce anti-frustration** : fais le projet dans l'ordre des étapes — chaque étape dépose une preuve dans le runbook. À la fin, le runbook **est** le projet.