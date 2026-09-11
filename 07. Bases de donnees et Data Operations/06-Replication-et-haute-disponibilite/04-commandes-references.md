# Commandes & références 6 — Réplication et haute disponibilité

## 1. Les 5 requêtes de diagnostic (à connaître par cœur)

```sql
-- 1. « Je suis quoi, moi ? » (sur n'importe quel serveur)
SELECT pg_is_in_recovery();          -- t = réplica · f = primary

-- 2. « Qui me suit et avec quel retard ? » (sur le primary)
SELECT application_name, state, sync_state,
       pg_wal_lsn_diff(pg_current_wal_lsn(), replay_lsn) AS lag_octets
FROM pg_stat_replication;

-- 3. « Où en est le journal ? » (sur le réplica)
SELECT pg_last_wal_replay_lsn();     -- la position rejouée

-- 4. « Quel est mon retard en secondes ? » (sur le réplica)
SELECT now() - pg_last_xact_replay_timestamp() AS retard;

-- 5. « Où le primary en est-il ? » (sur le primary)
SELECT pg_current_wal_lsn();         -- la position d'écriture
```

Lecture : `state = streaming` (flux vivant) · `sync_state = sync/async` (le mode) · `lag_octets = 0` (à jour) · un retard **croissant** = alerte.

## 2. Choisir le mode de réplication (synchronous_commit)

```sql
-- SYNCHRONE : le primary attend la confirmation du réplica nommé (RPO = 0)
ALTER SYSTEM SET synchronous_standby_names = 'replica_1';
SELECT pg_reload_conf();

-- ASYNCHRONE : retour au défaut (performance, RPO = secondes)
ALTER SYSTEM RESET synchronous_standby_names;
SELECT pg_reload_conf();
```

> ⚠️ En synchrone, si **aucun** réplica nommé ne répond, les écritures **bloquent** : le synchrone exige un réplica **en bonne santé** — c'est le contrat.

## 3. La promotion (la bascule manuelle)

```sql
-- Sur le RÉPLICA :
SELECT pg_promote();      -- « tu es le chef maintenant » (accepte les écritures)
SELECT pg_is_in_recovery();   -- → f : confirmé
```

Après promotion : **rediriger l'application** (chaîne de connexion / DNS / proxy) — puis **re-cloner l'ancien primary** (il a manqué le flux : on ne le « réintègre » jamais tel quel) :

```bash
sudo -u postgres pg_basebackup -h <nouveau-primary> -D /var/lib/postgresql/16/main \
     -U replicateur -R -P --wal-method=stream
# Le même outil que le backup physique de la Leçon 4 — rôles inversés (recrutement d'un sous-chef)
```

## 4. Le compte du réplication-user (la sécurité d'abord — Leçon 2)

```sql
-- Sur le primary : un rôle DÉDIÉ à la réplication (jamais le superuser !)
CREATE ROLE replicateur WITH REPLICATION LOGIN PASSWORD 'fort_et_unique';
GRANT ... ;   -- (REPLICATION est un attribut, pas un droit d'objet)
```

```conf
# postgresql.conf (côté primary)
listen_addresses = '*'            # sinon personne ne peut se connecter depuis le réplica
wal_level = replica               # le niveau de journal suffisant (défaut)
max_wal_senders = 3               # combien de réplicas peuvent se connecter
```

```conf
# postgresql.conf (côté réplica)
hot_standby = on                  # accepter les LECTURES pendant la copie
```

## 5. Les outils de failover automatique (pour situer)

| Outil | Architecture | Qui décide ? |
|---|---|---|
| **Patroni** | grappe de nœuds (3+ recommandés) | l'élection de leader (quorum — étcd/Consul) |
| **pg_auto_failover** | primary + réplica + **moniteur** | le moniteur (un tiers qui observe) |

Règle transversale : **jamais de promotion sans majorité/quorum** — c'est la protection anti *split-brain*.

## 6. TP Docker optionnel (à jouer après le Bloc 9)

```bash
mkdir -p ~/tp-replication && cd ~/tp-replication
docker network create repl-net

# Le primary
docker run -d --name primary --network repl-net \
  -e POSTGRES_PASSWORD=tp -e POSTGRES_USER=admin postgres:16
docker exec -it primary psql -U admin -d postgres \
  -c "CREATE TABLE notes (id serial PRIMARY KEY, texte text);" \
  -c "INSERT INTO notes (texte) VALUES ('premiere note');"

# Le réplica : clone physique (pg_basebackup) + suivi du flux (l'option -R)
docker run -d --name replica --network repl-net \
  -e POSTGRES_PASSWORD=tp -e POSTGRES_USER=admin -e PGUSER=admin \
  postgres:16 bash -c "
    rm -rf /var/lib/postgresql/data/* &&
    pg_basebackup -h primary -U admin -D /var/lib/postgresql/data \
      -R -P --wal-method=stream &&
    chmod 700 /var/lib/postgresql/data"

# Vérifier les rôles, écrire ici, lire là, mesurer le lag, promouvoir
docker exec -it primary psql -U admin -d postgres -c "SELECT pg_is_in_recovery();"  -- f
docker exec -it replica  psql -U admin -d postgres -c "SELECT pg_is_in_recovery();"  -- t
docker exec -it primary psql -U admin -d postgres \
  -c "INSERT INTO notes (texte) VALUES ('ecrite sur le primary');"
docker exec -it replica  psql -U admin -d postgres -c "SELECT * FROM notes;"
docker stop primary && docker exec -it replica psql -U admin -d postgres -c "SELECT pg_promote();"
```

Observations attendues : la ligne apparaît **sur le réplica** (le flux) ; après `stop primary` + `pg_promote()`, le réplica **accepte les écritures** — et l'ancien primary, au redémarrage, est **en retard** (il devra être re-cloné). Nettoyage : `docker rm -f primary replica && docker network rm repl-net`.

## 7. RTO/RPO — le tableau de décision (à garder en tête)

| Besoin métier | Configuration | RPO | RTO |
|---|---|---|---|
| On peut perdre la nuit | Backup nocturne (Leçon 4) | 24 h | heures |
| On ne perd RIEN, mais 4 h d'arrêt ok | Backup + **archivage WAL** (PITR) | ≈ 0 | heures |
| On ne perd RIEN et on revient vite | Réplication **synchrone** + failover auto | 0 | minutes |
| Rapports lourds sans ralentir | + un réplica de **lecture** | — | — |

## 8. Liens croisés du bloc

| Besoin | Où |
|---|---|
| WAL, `pg_basebackup`, PITR (les fondations) | `04-Backup-et-restauration/01-lecon.md` |
| Rôle dédié, moindre privilège | `02-Utilisateurs-roles-permissions-et-connexions/01-lecon.md` |
| Multi-AZ / Read Replicas (le managé) | Bloc 6 — RDS |
| Docker (pour le TP optionnel) | Bloc 9 — Conteneurs |