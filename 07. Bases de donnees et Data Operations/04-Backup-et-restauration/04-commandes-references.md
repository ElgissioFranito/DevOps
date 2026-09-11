# Aide-mémoire — Leçon 4 : backup et restauration

> **Bloc 7 · Leçon 4** — Table des commandes à garder à côté. Tout est local et gratuit.

---

## 1. Sauvegarder (la routine)

```bash
mkdir -p backups                                        # dossier des sauvegardes (relatif au projet)

# LA BASE — format custom compressé, date dans le nom
sudo -u postgres pg_dump -Fc -d bibliotheque -f "backups/bibliotheque-$(date +%F).dump"
# -Fc : format custom (compressé, restaurable table par table) ; -d : la base ; -f : le fichier destination

# LA BASE — format plain (SQL lisible, pour petites bases / Git)
sudo -u postgres pg_dump -d bibliotheque -f "backups/bibliotheque-$(date +%F).sql"

# LES RÔLES ET DROITS GLOBAUX (indispensable — vit HORS de la base)
sudo -u postgres pg_dumpall --globals-only -f "backups/globals-$(date +%F).sql"

# TOUT le cluster (rôles + toutes les bases) — pour les migrations d'installation
sudo -u postgres pg_dumpall -f "backups/cluster-$(date +%F).sql"

ls -lh backups                                          # -l détaille ; -h tailles lisibles
```

## 2. Vérifier un backup (avant d'en avoir besoin)

```bash
pg_restore -l "backups/bibliotheque-$(date +%F).dump" | head -20   # -l : lister le CONTENU de l'archive
# → repérer : TABLE..., TABLE DATA..., ACL... (les droits)

# Le test qui compte : restaurer À CÔTÉ puis compter
sudo -u postgres psql -c 'CREATE DATABASE test_restore;'
sudo -u postgres pg_restore -d test_restore "backups/bibliotheque-$(date +%F).dump"
sudo -u postgres psql -d test_restore -c 'SELECT count(*) FROM membres;'
sudo -u postgres psql -c 'DROP DATABASE test_restore;'     # nettoyage
```

## 3. Restaurer

```bash
# FORMAT CUSTOM (pg_dump -Fc)
sudo -u postgres pg_restore -d bibliotheque_restaure "backups/bibliotheque-2026-09-11.dump"
# -d : la base cible (existant et vide, ou avec --clean : DROP puis CREATE des objets)

# Une SEULE table depuis un dump custom (le superpouvoir du format custom)
sudo -u postgres pg_restore -d bibliotheque_restaure -t membres "backups/bibliotheque-2026-09-11.dump"
# -t : ne restaurer que CETTE table

# FORMAT PLAIN (pg_dump / .sql) : se rejoue comme un script
sudo -u postgres psql -d bibliotheque_restaure -f "backups/bibliotheque-2026-09-11.sql"
# -f : exécuter le fichier SQL

# Les rôles : se rejouent comme un script SQL
sudo -u postgres psql -f "backups/globals-2026-09-11.sql"
```

## 4. Le PITR (mécanisme — mots-clés à connaître)

```bash
# La « photo » physique (fichiers + WAL) :
sudo -u postgres pg_basebackup -D "backups/base-$(date +%F)" -Fp -Xs -P
# -D : dossier destination ; -Fp : fichiers en clair ; -Xs : WAL en flux (streaming) ; -P : progression
```

```text
PITR = photo (pg_basebackup) + carnet (WAL archivés, via archive_mode) + cible (recovery_target_time)
Restauration = photo → rejouer le carnet → S'ARRÊTER à la cible (ex. 14:02)
Chez un fournisseur (RDS — Bloc 6) : automatique ; TU gères la rétention et les tests.
```

## 5. Automatiser et appliquer la 3-2-1

```bash
sudo -u postgres crontab -e      # les tâches planifiées du compte postgres (cron — Bloc 2)
# 30 2 * * *  pg_dump -Fc bibliotheque > backups/bibliotheque-$(date +\%F).dump
# chaque jour 02h30 ; \% : % échappé (sinon cron coupe la ligne)
```

```text
La stratégie 3-2-1 :
  3 copies (l'originale + 2 backups)  ×  2 supports (disque + stockage objet)  ×  1 hors site (S3 — Bloc 6)
Rétention d'exemple : 7 quotidiens + 4 hebdo + 12 mensuels (puis purge automatique)
Chiffrement : les dumps contiennent TOUTES les données → chiffrés (S3 au repos / gpg — Bloc 5)
RPO (ce qu'on accepte de perdre) → décide la fréquence (approfondi au Bloc 8 — Disaster Recovery)
```

## 6. La check-list « je peux dormir » (résumé de la leçon)

```
[ ] pg_dump -Fc chaque nuit (cron, 02h30)          → la photo
[ ] pg_dumpall --globals-only chaque nuit          → les rôles
[ ] 2e support + 1 hors site chiffré (S3)          → le 3-2-1
[ ] rétention définie et purgée automatiquement    → le disque ne déborde pas (Leçon 3)
[ ] test de restauration MENSUEL (à côté + compter) → « un backup non testé n'est pas un backup »
[ ] backup AVANT chaque migration/DELETE en masse  → le filet avant le saut (Leçon 5)
```
