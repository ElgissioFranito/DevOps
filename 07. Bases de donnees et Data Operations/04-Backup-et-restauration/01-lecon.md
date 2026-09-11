# Leçon 4 — Backup et restauration : protéger les données

> **Bloc 7 · Bases de données & Data Operations** — Leçon 4 sur 8
> 🧭 **Pont depuis la Leçon 3** : ta base est **bien gardée** (Leçon 2) et **bien réglée** (Leçon 3). Mais aucune permission ni configuration ne protège d'un **disque qui meurt**, d'un **malware** qui chiffre les fichiers, ou d'un `DELETE` sans `WHERE` passé hors transaction. La roadmap exige ici : *« backup complet, incrémental, restauration, point-in-time recovery, stratégie »* — et son critère de validation du bloc est clair : **« expliquer comment éviter une perte de données »**. C'est la leçon la plus importante du bloc : sauvegarder, restaurer, et **tester**.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Expliquer** les 3 causes de perte de données et pourquoi *« un backup non testé n'est pas un backup »*.
2. **Sauvegarder** une base complète avec `pg_dump` (et les **rôles** avec `pg_dumpall` — le piège classique).
3. **Restaurer** un backup avec `pg_restore` et **vérifier** la restauration par des comptages.
4. **Définir** le **WAL** (le journal de bord du SGBD), l'incrémental, et le **PITR** (restauration à un instant précis).
5. **Appliquer** la stratégie **3-2-1** et décider de la fréquence selon ce qu'on accepte de perdre.
6. **Vivre** une panne simulée et s'en remettre (exercice : tu vas réellement casser et réparer ta base — chez toi, sans risque).

---

## 2. Explication simple

### 2.1 Le « pourquoi » : comment on perd des données

Trois causes couvrent pratiquement toutes les pertes :

```
1. L'erreur humaine    → « j'ai lancé le DELETE sans WHERE en production »        (la plus fréquente)
2. La panne matérielle → le disque rend l'âme : plus rien à lire, même pas les erreurs
3. L'attaque           → un malware chiffre les fichiers et demande une rançon
```

> 💡 **Analogie** : le backup est une **assurance-vie**. Tu paies (un peu d'espace, un peu de temps) tous les jours pour quelque chose que tu espères ne **jamais** utiliser — et le jour où tu en as besoin, tu es le seul à te réjouir d'avoir payé.

Et la phrase à graver :

> ⚠️ **Un backup non testé n'est pas un backup.** Un backup qu'on n'a **jamais** restauré peut être vide, corrompu, incomplet — tu le découvriras au pire moment. Le test de restauration fait **partie** de la sauvegarde, comme le contrôle du parachute fait partie du saut.

**Le « quand »** : les backups se préparent **avant** l'incident, se programment **régulièrement** (automatisation — `cron`, le planificateur de tâches de Linux, vu au Bloc 2), et se font **systématiquement avant toute opération risquée** (une migration de schéma — la Leçon 5 t'y prépare).

### 2.2 Le « comment » (partie 1) : le backup logique — `pg_dump`

**`pg_dump`** crée une **copie logique** : un fichier qui contient tout ce qu'il faut pour **reconstruire** la base — les tables, les données, les index, les droits sur les objets. Deux qualités décisives :

- **cohérence** : la copie représente la base **exactement à l'instant du début** du dump, même si des gens écrivent pendant (pas de fichier « à moitié modifié ») ;
- **portabilité** : la même archive se restaure sur une autre machine, une autre version, un autre cloud.

Deux **formats** à connaître :

| Format | Commande | Restauré avec | Qualités |
|---|---|---|---|
| **Plain** (SQL lisible) | `pg_dump -Fp` (défaut) | `psql -f fichier.sql` | lisible, versionnable dans Git (petites bases) |
| **Custom** (compressé) | `pg_dump -Fc` | `pg_restore -d base fichier` | **compressé**, et on peut **restaurer une seule table** si besoin — le format recommandé |

> 💡 **Analogie** : le format plain, c'est la recette **écrite en clair** (tu peux la lire, la corriger à la main) ; le format custom, c'est la recette **plastifiée et compressée dans une pochette** (plus compacte, plus rapide à rejouer, et tu peux n'extraire qu'une page).

Et le **piège classique** que tu vas vivre en exercice : `pg_dump` sauvegarde **une base**, pas les **rôles** (les utilisateurs, leurs mots de passe, les droits sur les bases). Pour cela, il existe **`pg_dumpall`** (le dump « global » : rôles et droits du **cluster** — l'ensemble géré par ton installation PostgreSQL). Oublier `pg_dumpall`, c'est restaurer une base **impossible à utiliser** : plus aucun utilisateur autorisé à se connecter.

### 2.3 Le « comment » (partie 2) : le journal de bord — WAL, incrémental et PITR

Le backup logique est une **photo** : il fige un instant. Pour rattraper ce qui s'est passé **après** la photo, il faut le **journal de bord**.

**Le WAL** (Write-Ahead Log, « journal écrit à l'avance ») : PostgreSQL note **chaque modification** dans un journal **avant** de l'appliquer aux données. Choix de fiabilité : si le serveur tombe en pleine écriture, le journal permet de **rejouer** ce qui était noté et de retrouver un état propre.

> 💡 **Analogie** : le caissier **note chaque opération dans le carnet** avant de toucher au tiroir-caisse. Si on l'interrompt, on relit le carnet : on sait exactement où il en était. Le carnet, c'est le WAL.

**L'incrémental** se comprend alors naturellement :

```
Backup complet (la photo) du dimanche
        ↓
Les WAL archivés de la semaine (le carnet) = l'incrémental : on ne stocke QUE les changements
        ↓
Restauration = photo de dimanche + carnet rejoué jusqu'à l'instant voulu
```

**Le PITR** (Point-In-Time Recovery, « restauration à un instant précis ») : cette combinaison permet de dire *« restaure-moi la base telle qu'à 14h02 »* — utile quand on a supprimé des données par erreur **à 14h03**. Les outils : `pg_basebackup` (une copie **physique** des fichiers de la base + les WAL nécessaires) puis un **recovery_target_time** (la cible temporelle de la restauration).

> 🔁 **Pont vers le Bloc 6 (RDS)** : sur une base **managée**, tout ce mécanisme est **automatisé** — c'est exactement ce que faisaient les « backups automatiques » et le « point-in-time recovery » de RDS (Leçon 5 du Bloc 6). Ici tu comprends **sous le capot** ; avec RDS en pratique, tu restes responsable de la **rétention** (combien de jours on garde) et de la **vérification**.

### 2.4 Le « quand » et la stratégie : 3-2-1

La règle **3-2-1** — la stratégie industrielle :

```
3   copies des données   (l'originale + 2 sauvegardes)
2   supports différents  (ex. disque local + stockage objet S3, vu au Bloc 6)
1   copie HORS SITE      (dans un autre lieu/région — le vol ou l'incendie ne doit pas tout emporter)
```

> 💡 **Analogie** : ta thèse de fin d'études, tu la gardes sur le PC, sur un disque externe, et **une copie dans le nuage**. Le PC qui brûle n'emporte que la première copie.

Et la **fréquence** se décide avec une question honnête : *« combien d'heures de données accepte-je de perdre ? »* La réponse s'appelle le **RPO** (Recovery Point Objective, le point de reprise — approfondi au Bloc 8 avec le Disaster Recovery). Sauvegarder toutes les heures = perdre au pire **1 heure** de données. Sauvegarder chaque nuit = perdre au pire **1 jour**.

**Le « quand » en résumé** :

```
Automatique   → cron (chaque nuit/heure)              : la routine
Avant chaque  → migration de schéma (Leçon 5),        : le filet AVANT l'opération risquée
              → modification risquée (DELETE en masse)
Périodique    → test de restauration (mensuel)        : « un backup non testé n'est pas un backup »
```

---

## 📖 Vocabulaire / Abréviations

- **Backup** (sauvegarde) : une copie permettant de **restaurer** les données.
- **Restore** (restauration) : remettre les données depuis un backup.
- **`pg_dump`** : l'outil de sauvegarde **d'une base** (format plain ou custom).
- **`pg_dumpall`** : l'outil de sauvegarde des objets **globaux** (rôles, droits sur les bases).
- **`pg_restore`** : l'outil de restauration d'un dump au **format custom** (`-Fc`).
- **`pg_basebackup`** : sauvegarde **physique** (les fichiers de la base + les WAL nécessaires).
- **Dump logique / dump physique** : la recette qui reconstruit / la copie brute des fichiers de données.
- **Cohérence** : la copie représente un **instant précis**, pas un état « à moitié modifié ».
- **Format plain (`-Fp`)** : SQL en clair, lisible, restauré avec `psql -f`.
- **Format custom (`-Fc`)** : archive **compressée**, restaurée avec `pg_restore` (peut n'extraire qu'une table).
- **WAL** (Write-Ahead Log) : le journal de bord où PostgreSQL **note chaque modification avant** de l'appliquer.
- **Archivage WAL** : conserver les fichiers de journal pour rejouer l'histoire après un backup.
- **Backup incrémental** : ne stocker **que les changements** depuis le dernier backup (ici : les WAL).
- **PITR** (Point-In-Time Recovery) : restaurer la base **à un instant précis** (photo + carnet rejoué).
- **`recovery_target_time`** : la **cible temporelle** d'une restauration PITR.
- **Stratégie 3-2-1** : 3 copies, 2 supports, 1 hors site.
- **Rétention** : combien de temps / combien de copies on garde.
- **RPO** (Recovery Point Objective) : la quantité de données qu'on **accepte de perdre** (approfondi au Bloc 8).
- **`cron`** : le planificateur de tâches de Linux (Bloc 2).
- **Hors site** : stocké dans un **autre lieu/région** (le nuage — Bloc 6).
- **Cluster PostgreSQL** : l'ensemble géré par une installation du SGBD (les bases + les rôles) — distinct de Kubernetes (Bloc 10).
- **Malware** : logiciel malveillant (ex. ransomware = malware qui **chiffre** les fichiers pour extorquer).
- **ACL** (Access Control List) : la liste des droits sur un objet (le résultat des `GRANT` de la Leçon 2).

---

## 3. Exemples concrets

> 🔁 On enchaîne : le vocabulaire est posé ; voici **les commandes exactes**, commentées ligne par ligne. Tout est local et gratuit — et l'exercice te fera **vivre** la panne.

### 3.1 Sauvegarder (backup complet + rôles)

```bash
mkdir -p backups        # le dossier des sauvegardes, RELATIF au projet (-p : ne râle pas s'il existe déjà)

# LA BASE (format custom, compressé, avec la DATE dans le nom)
sudo -u postgres pg_dump -Fc -d bibliotheque -f "backups/bibliotheque-$(date +%F).dump"
# pg_dump : sauvegarde UNE base ; -Fc : format custom compressé ; -d : la base à copier ;
# -f : le fichier destination ; $(date +%F) : la DATE du jour (AAAA-MM-JJ) insérée par le shell

# LES RÔLES (le dump global — sans lui, la base restaurée est inutilisable)
sudo -u postgres pg_dumpall --globals-only -f "backups/globals-$(date +%F).sql"
# pg_dumpall : les objets GLOBAUX du cluster ; --globals-only : uniquement rôles et droits globaux

ls -lh backups          # -l : liste détaillée ; -h : tailles lisibles (K, M...)
```

> 💡 **Pourquoi `sudo -u postgres` ?** En local, l'administrateur se connecte via l'authentification **peer** (Leçon 2) : le dump doit tourner depuis le compte système `postgres`. Sur un serveur distant, on ajouterait `-h adresse -U utilisateur` (et le mot de passe serait demandé — jamais collé dans la commande, voir les pièges).

### 3.2 Vérifier le backup (SANS le restaurer)

```bash
pg_restore -l "backups/bibliotheque-$(date +%F).dump" | head -20
# -l : LISTE le contenu de l'archive (tables, index, ACL...) ; head -20 : garde 20 lignes
```

Extrait attendu (les lignes exactes varient) :

```
3245; 1259 16456 TABLE membres postgres
3249; 1259 16447 TABLE livres postgres
3258; 0 16456 TABLE DATA membres postgres
...
3264; 0 0 ACL SCHEMA "public" postgres       ← ACL = les DROITS (Access Control List) : les GRANT de la Leçon 2 sont dedans
```

> 💡 **Ce contrôle de 5 secondes** est le premier « test » du backup : un fichier de 0 octet, ou une liste sans tes tables, se repère **maintenant**, pas le jour de la panne.

### 3.3 Restaurer à côté (le test de restauration)

```bash
# 1. Une base de réception NEUVE (on ne restaure jamais d'abord « en production »)
sudo -u postgres psql -c 'CREATE DATABASE bibliotheque_restaure;'
# psql -c : exécute UNE commande SQL puis quitte

# 2. La restauration proprement dite
sudo -u postgres pg_restore -d bibliotheque_restaure "backups/bibliotheque-$(date +%F).dump"
# -d : la base cible

# 3. LA VÉRIFICATION par les comptages (la preuve, pas l'impression)
sudo -u postgres psql -d bibliotheque_restaure -c 'SELECT count(*) FROM membres;'
sudo -u postgres psql -d bibliotheque -c 'SELECT count(*) FROM livres;'
# → les deux comptages doivent COÏNCIDER avec les valeurs avant la panne
```

### 3.4 La mise en service après une vraie panne

```bash
# La base est détruite/illisible ? On la reconstruit :
sudo -u postgres psql -c 'DROP DATABASE bibliotheque;'     # efface la base cassée
sudo -u postgres psql -c 'CREATE DATABASE bibliotheque;'   # la recrée VIDE
sudo -u postgres pg_restore -d bibliotheque "backups/bibliotheque-$(date +%F).dump"

# Puis on ré-applique ce que le dump de base ne contient PAS :
# les droits CONNECT sur la base → depuis TON script versionné de la Leçon 2 :
sudo -u postgres psql -d bibliotheque -c "GRANT CONNECT ON DATABASE bibliotheque TO app_biblio;"
sudo -u postgres psql -d bibliotheque -c "GRANT CONNECT ON DATABASE bibliotheque TO lecteur_biblio;"
# (les GRANT de TABLES, eux, reviennent AVEC le dump — les ACL sont dedans, cf. 3.2)
```

```bash
# LA VRAIE VÉRIFICATION : les clients (pas l'admin) doivent fonctionner
psql -h localhost -U app_biblio -d bibliotheque      # l'application : INSERT/SELECT doivent marcher
psql -h localhost -U lecteur_biblio -d bibliotheque  # l'analyste : SELECT oui, DELETE refusé (Leçon 2)
```

> 💡 **Le moment «aha»** : après cette séquence, tu as vécu la différence entre *« la base est restaurée »* (l'admin peut lire) et *« le service est rendu »* (l'application et l'analyste travaillent). C'est le **service** qu'on restaure, pas la base.

### 3.5 L'incrémental et le PITR (sous le capot — à connaître)

```bash
# Le backup PHYSIQUE (la « photo » des fichiers + les WAL nécessaires) :
sudo -u postgres pg_basebackup -D "backups/base-$(date +%F)" -Fp -Xs -P
# -D : le dossier destination ; -Fp : format « plain » (de vrais fichiers, pas une archive) ;
# -Xs : récupère les WAL en FLUX (streaming) pendant la sauvegarde ; -P : affiche la progression
```

```
Le mécanisme PITR (résumé en une image) :

photo (pg_basebackup du 08:00)  +  carnet (WAL archivés)  +  cible (recovery_target_time = 14:02)
        ↓ rejouer le carnet jusqu'à la cible
la base telle qu'à 14h02  →  l'écriture erronée de 14h03 n'a jamais existé
```

> 💡 En auto-hébergé, le PITR demande la configuration de l'**archivage WAL** (`archive_mode` + un dossier de destination). **Pour le bloc, retiens le mécanisme et les mots-clés** — la mise en place complète est un sujet de production avancée. Chez un fournisseur (RDS, Bloc 6), tu coches une case et c'est **automatique** : c'est exactement le prix du « managé ».

### 3.6 Automatiser avec `cron`

```bash
sudo -u postgres crontab -e    # édite les tâches planifiées DU compte postgres (cron — vu au Bloc 2)
# 30 2 * * *  pg_dump -Fc bibliotheque > backups/bibliotheque-$(date +\%F).dump
# 30 2 * * * : chaque jour à 02h30 (minute heure jour-du-mois mois jour-de-semaine) ;
# \% : le % est ÉCHAPPÉ dans cron (sinon il coupe la ligne) ;
# idée à adapter : aussi « cd backups && ... » pour écrire au bon endroit, et un test de restauration MENSUEL
```

> 💡 **Pourquoi 02h30 ?** La fenêtre de moindre trafic (Bloc 2 : les fenêtres de maintenance). Un backup consomme du disque et du CPU — on le décale quand personne ne regarde.

---

## 4. Bonnes pratiques modernes (2025-2026)

1. **Format custom (`-Fc`)** par défaut : compressé, rapide, et il permet d'extraire **une seule table** si besoin.
2. **`pg_dumpall --globals-only` à chaque ronde de backup** : les rôles et droits globaux vivent HORS de la base — le piège que tu vas vivre en exercice.
3. **Le backup porte la DATE** dans son nom (`$(date +%F)`) : les fichiers se parlent d'eux-mêmes, la rétention se gère sans deviner.
4. **Stratégie 3-2-1** : 3 copies, 2 supports, 1 **hors site** — la copie externe est typiquement le **stockage objet S3** (Bloc 6) chiffré.
5. **Chiffrer les sauvegardes** : un dump contient **toutes** les données (utilisateurs, emails...) — un backup volé est une fuite de données. (Le chiffrement au repos est fourni par S3 ; en local, `gpg` — l'outil de chiffrement vu au Bloc 5.)
6. **Définir la rétention** (ex. 7 quotidiens + 4 hebdomadaires + 12 mensuels) et **l'automatiser** : un disque qui garde tout finit plein (Leçon 3 !).
7. **Tester la restauration régulièrement** (mensuel, automatisable) : restaurer dans une base de test et **compter** — la seule preuve acceptée.
8. **Backup AVANT chaque opération risquée** : migration de schéma (**Leçon 5**), `DELETE` en masse, changement de version majeure.
9. **Sur base managée (RDS)** : vérifier la **rétention** configurée, tester la restauration quand même, et garder un **export logique indépendant du fournisseur** (anti-verrouillage).

---

## 5. Pièges à éviter

### Piège 1 — Le backup jamais testé

```
❌ MAUVAIS : le script tourne chaque nuit, personne n'a JAMAIS restauré
   → le jour de la panne : fichier corrompu depuis 6 mois, personne ne le sait.

✅ CORRECT : restaurer dans une base de TEST, compter, comparer — régulièrement
   (l'étape 3.3 ci-dessus, automatisée une fois par mois).
```

**Pourquoi** : c'est l'équivalent du parachute jamais plié par un professionnel. Le test fait partie de la sauvegarde.

### Piège 2 — Le backup sur le même disque que les données

```
❌ MAUVAIS : pg_dump -f /même/disque/...  (une seule « copie » = pas une copie)
✅ CORRECT : 3-2-1 — disque local + support différent + hors site (S3)
```

**Pourquoi** : le scénario le plus fréquent de perte **totale** : le disque meurt — données ET sauvegardes ensemble.

### Piège 3 — Oublier `pg_dumpall` (les rôles)

```
❌ MAUVAIS : pg_dump seul → la base restaurée refuse les connexions (plus de rôles connus)
✅ CORRECT : pg_dump + pg_dumpall --globals-only (tu le VIVRAS à l'étape 4 de l'exercice)
```

**Pourquoi** : la base se restaure parfaitement, puis « password authentication failed » — et tu cherches pourquoi : les **rôles** vivaient à côté.

### Piège 4 — Le mot de passe dans la ligne de commande

```bash
# ❌ MAUVAIS : visible dans l'historique du shell ET dans la liste des processus
pg_dump -U app_biblio --password=MotDePasse123 -d bibliotheque ...

# ✅ CORRECT : la saisie cachée, ou le fichier ~/.pgpass (dans TON dossier utilisateur)
psql -h localhost -U app_biblio -d bibliotheque      # le mot de passe est DEMANDÉ (caché)
# ~/.pgpass : un fichier de ton dossier utilisateur (lignes host:port:base:utilisateur:motdepasse,
# droits 0600 = lisible par toi seul — chmod 0600 vu au Bloc 2), lu automatiquement par les outils PostgreSQL
```

**Pourquoi** : tout ce qui est passé en argument est **lisible** par les autres utilisateurs de la machine (commande `ps` — la liste des processus).

### Piège 5 — Restaurer PAR-DESSUS la base active

```
❌ MAUVAIS : pg_restore -d production (l'application tourne, les erreurs se multiplient, à moitié restauré)
✅ CORRECT : restaurer À CÔTÉ (base de test / instance neuve) → vérifier → basculer l'application
```

**Pourquoi** : tu ne remplaces pas le moteur pendant que la voiture roule. La bascule est **un choix explicite**, pas un effet de bord.

### Piège 6 — Croire que le backup d'« hier soir » suffit toujours

```
❌ MAUVAIS : backup quotidien pour une application où on accepte de perdre 15 minutes
✅ CORRECT : la fréquence découle du RPO (ce qu'on accepte de perdre) — ici : horaire + WAL
```

**Pourquoi** : la fréquence est un **choix métier chiffré** (le RPO), pas une habitude copiée.

---

## 6. Exercice pratique

> 🔁 **Comment s'articulent les fichiers** : la théorie est terminée, passons à la pratique — et celle-ci est **mémorable** : tu vas réellement casser ta base, puis la réparer. L'exercice complet est dans **`02-exercice.md`** (à faire **avant** la correction).

En résumé, tu vas — **en local, sans risque** :

1. préparer la table `membres` (5 lignes) — les données à protéger ;
2. **sauvegarder** : `pg_dump -Fc` (la base) + `pg_dumpall --globals-only` (les rôles) + vérification du contenu (`pg_restore -l` et la mention `ACL`) ;
3. **simuler la panne** : d'abord dans une transaction (sans risque), puis pour de vrai (`DROP TABLE`) ;
4. **restaurer à côté** (`bibliotheque_restaure`) et **vérifier par les comptages** ;
5. **remettre en service** : `DROP DATABASE` → `CREATE` → `pg_restore` → ré-appliquer les `GRANT CONNECT` (ton script versionné de la Leçon 2) → **tester avec les rôles clients** ;
6. **réfléchir** : pourquoi `pg_dumpall`, pourquoi restaurer à côté, ta fréquence/RPO, et où ranger les copies (3-2-1).

Livrable : `notes-exercice-04.md`.

---

## 7. Correction détaillée de l'exercice

La correction complète (sorties attendues, le piège des `GRANT CONNECT` que tu vas vivre, la lecture de `pg_restore -l`) est dans **`03-correction.md`**, qui réécrit la checklist finale et donne des conseils.

---

## 8. Checklist de validation

- [ ] Je peux nommer les 3 causes de perte de données et expliquer pourquoi *« un backup non testé n'est pas un backup »*.
- [ ] Je sais sauvegarder une base avec `pg_dump -Fc` (date dans le nom) **et** les rôles avec `pg_dumpall --globals-only`.
- [ ] Je sais **vérifier un backup** sans le restaurer (`pg_restore -l`, tailles avec `ls -lh`).
- [ ] Je sais restaurer **à côté** (base de test), **vérifier par les comptages**, puis remettre en service (recreate + `pg_restore` + `GRANT CONNECT`).
- [ ] Je peux expliquer le **WAL** (le carnet du caissier), l'**incrémental** (le carnet archivé) et le **PITR** (photo + carnet rejoué jusqu'à une heure cible).
- [ ] Je connais la stratégie **3-2-1** et je sais décider une fréquence à partir d'un **RPO**.
- [ ] Je sais quand faire un backup **au-delà** du planning : avant toute opération risquée (migration — Leçon 5).
- [ ] Je sais pourquoi on ne met **jamais** un mot de passe dans une ligne de commande.

---

> 🧭 **Prochaine étape** : tu sais maintenant **protéger** les données. La **Leçon 5** (migrations de schéma et de données) aborde l'opération qui fait justement **peur** aux équipes sans backup : **modifier la structure** de la base en production — proprement, versionné dans Git, avec un plan de retour en arrière (le **rollback**). Tu comprendras alors pourquoi chaque migration sérieuse commence par... un backup.
