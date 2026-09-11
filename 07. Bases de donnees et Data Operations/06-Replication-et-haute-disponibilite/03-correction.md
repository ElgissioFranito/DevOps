# Correction 6 — Trois pannes, trois métiers, un plan de reprise

> Compare **ton raisonnement** à celui-ci. Il n'y a pas une réponse unique — il y a des **justifications** qui tiennent ou pas. C'est l'exercice des chiffres.

## Scénario 1 — La boutique du 29 novembre

1. **Traduction en chiffres** : « on ne peut pas perdre plus de 5 min de commandes » → **RPO = 5 min** (max). « Chaque commande perdue = un client perdu » et le jour de pointe → l'interruption doit être **courte** : on fixera **RTO ≈ 15 min** (à valider avec le métier).
2. **Élimination** :
   - **A (backup nocturne)** : RPO = jusqu'à 24 h → **viole** le RPO de 5 min. Éliminée seule (mais elle reste un **complément**, voir 5).
   - **B (asynchrone)** : RPO = secondes → **respecte** 5 min ; RTO = court (promotion manuelle ~15 min avec un astreint qui connaît la procédure) → limite mais jouable.
   - **C (synchrone + failover auto)** : RPO = 0, RTO = minutes → **respecte** tout.
3. **Choix attendu** : **C** pour un jour de pointe où chaque commande compte — et c'est un métier qui **paie** la latence d'écriture (le synchrone) parce que le RPO = 0 l'exige. Réponse honnête acceptée aussi : **B + failover semi-auto** si le RPO réel du métier est « 1 minute » et non « zéro » — tant que **les chiffres sont posés avec le métier**.
4. **Les étapes de bascule** : détecter (outil/moniteur) → élire (outil, quorum) → promouvoir (outil — `pg_promote()`) → rediriger (proxy/DNS — selon ta config : outil ou humain) → vérifier l'application (humain) → re-cloner l'ancien primary en réplica (humain, le lendemain).

## Scénario 2 — L'intranet de 30 collaborateurs

1. **Chiffres** : « demi-journée ok » → **RTO = 4 h**. « On ne perd rien » → **RPO = 0**... **en apparence**.
2. **Le vrai rythme d'écriture** : ~50 écritures/jour. Entre deux écritures, il ne se passe **rien** : l'exigence « RPO = 0 » se ramène en pratique à « ne pas perdre la **dernière écriture** » — ce que l'**archivage WAL continu** (Leçon 4 : `archive_mode = on`) garantit **sans réplication synchrone** : chaque écriture est archivée au coffre à la seconde près (PITR).
3. **Choix attendu** : **A renforcé** — backup nocturne + **archivage WAL** (PITR). RPO ≈ 0 (on rejoue le journal jusqu'à l'erreur), RTO = heures (acceptable), **budget minimal** (un seul serveur + stockage du coffre).
   - Le piège du raisonnement paresseux : « RPO = 0 ⇒ synchrone obligatoire ». Le synchrone protège la **disponibilité** aussi — or ici le métier accepte 4 h. **Le PITR (Leçon 4) est la solution du RPO sans payer le prix du synchrone.**
4. **La phrase du plan** : *« En cas de panne du serveur, l'intranet est indisponible au maximum 4 h (RTO) ; aucune donnée validée n'est perdue (RPO ≈ 0) grâce à l'archivage WAL continu et au backup nocturne testé. »*

## Scénario 3 — Le rapport du lundi

1. **Diagnostic** (sur le primary) :

```sql
SELECT application_name, state, sync_state,
       pg_wal_lsn_diff(pg_current_wal_lsn(), replay_lsn) AS lag_octets
FROM pg_stat_replication;
```

   On regarde **`lag_octets`** : pendant l'agrégation de 10 M lignes, le réplica est **saturé en lecture** — il rejoue le journal au ralenti → le lag **grimpe** (parfois des centaines de Mo, soit des **secondes à minutes** de retard).
2. **Le phénomène** : l'utilisateur écrit son profil sur le **primary**, l'application le **relit sur le réplica** (qui a un retard de X secondes) → « je ne vois pas ma modification ». Ce n'est pas une perte : c'est une **relecture au mauvais endroit**.
3. **La règle à formuler** : *« les écritures et leur relecture immédiate passent par le primary ; le réplica ne reçoit que les lectures tolérantes au retard (rapports, dashboards, exports). »*
4. **Le seuil d'alerte** : un lag qui **croît de façon durable** (ex. > 10 Mo ou > 5 s) mérite une alerte — pas un pic d'une seconde. Un lag croissant = un réplica qui ne suit plus (saturé) = un sous-chef qui dormirait pendant le service.

## Synthèse — Le plan de reprise attendu (exemple : scénario 1)

```text
PLAN DE REPRISE — boutique (29 nov)
RTO : 15 min   |   RPO : 5 min (visé 0 en synchrone)
Configuration : réplication SYNCHRONE + failover auto (pg_auto_failover)
Étapes de bascule :
  1. Détection de la panne .......... pg_auto_failover (moniteur) ~10 s
  2. Élection du nouveau primary .... quorum (le moniteur décide)
  3. Promotion ...................... pg_promote() automatique
  4. Redirection de l'application ... proxy/DNS (automatisé)
  5. Vérification applicative ....... humain (checklist de 3 tests)
  6. Re-clonage de l'ancien ......... humain, le lendemain (pg_basebackup)
Test du plan : chaque TRIMESTRE, bascule simulée en conditions contrôlées
Filet anti-erreur-humaine : backup nocturne + PITR (le DROP de 14h02 se réplique aussi !)
```

## Question bonus

« Si le datacenter entier tombe » → la réponse managée du **bloc 6** : répliquer **entre régions** (Multi-Region / Read Replica cross-region chez AWS RDS) — la copie vit dans un **autre lieu géographique**, pas seulement une autre zone. En une phrase : *« le Multi-AZ protège d'une panne de machine ; le multi-région protège d'une panne de lieu. »*

## Grille d'auto-évaluation

| Critère | C'est réussi si... |
|---|---|
| Traduction métier → chiffres | Tu as posé RTO **et** RPO avec une **unité** (min, h) — jamais « rapide » ou « rien » |
| Élimination | Tu as **rejeté** les configs qui violent les chiffres (pas juste « préféré » une autre) |
| Choix justifié | Ta justification **cite les chiffres** et le **coût** (latence, budget, humain) |
| Bascule | Les 6 étapes sont là, avec le responsable de chacune (outil/humain) |
| Le piège du lag | Tu as formulé la règle « écriture + relecture = primary » |
| Le pont Leçon 4 | Tu as rappelé que la réplication **copie aussi les erreurs** — le PITR reste le filet |
| Le bonus | Multi-zone vs multi-région : machine vs **lieu** |