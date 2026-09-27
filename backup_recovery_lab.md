# PostgreSQL Backup, Recovery and Replication Lab

## Step 1: Take and Verify a Logical Backup

```bash
mkdir -p ~/backups
pg_dump -Fc -f ~/backups/bootcamp.dump bootcamp
pg_restore --list ~/backups/bootcamp.dump | head
createdb bootcamp_check
pg_restore -d bootcamp_check ~/backups/bootcamp.dump
```

## Step 2: Enable WAL Archiving

Add the following settings to `postgresql.conf`:

```conf
wal_level = replica
archive_mode = on
archive_command = 'cp %p /home/$USER/backups/wal/%f'
```

Create the WAL backup directory:

```bash
mkdir -p ~/backups/wal
```

Restart PostgreSQL:

```bash
sudo systemctl restart postgresql
```

Take a base backup:

```bash
pg_basebackup -D ~/backups/base -Ft -z -Xs -P
```

## Step 3: Simulate a Disaster and Recover

Record the current time:

```sql
SELECT now();
```

Delete the student data:

```sql
DELETE FROM students;
```

Stop PostgreSQL:

```bash
sudo systemctl stop postgresql
```

Configure recovery in `postgresql.conf`:

```conf
restore_command = 'cp ~/backups/wal/%f %p'
recovery_target_time = 'YYYY-MM-DD HH:MM:SS'
```

Replace `YYYY-MM-DD HH:MM:SS` with the time recorded before the deletion.

Start PostgreSQL:

```bash
sudo systemctl start postgresql
```

Verify the recovery:

```sql
SELECT count(*) FROM students;
```

## Step 4: Set Up a Streaming Standby

Create a replication role on the primary server:

```sql
CREATE ROLE replicator
WITH REPLICATION LOGIN PASSWORD 'reppass';
```

Add this line to `pg_hba.conf`:

```conf
host replication replicator 127.0.0.1/32 md5
```

Build the standby server:

```bash
pg_basebackup -h 127.0.0.1 -U replicator -D ~/standby -R -P
```

## Step 5: Watch Replication Health

Run this query on the primary server:

```sql
SELECT application_name,
       state,
       pg_wal_lsn_diff(sent_lsn, replay_lsn) AS lag_bytes
FROM pg_stat_replication;
```

## Wrap-Up

This lab demonstrates:

* Logical PostgreSQL backups
* Backup verification
* WAL archiving
* Base backups
* Point-in-Time Recovery (PITR)
* Disaster recovery
* Streaming replication
* Replication health monitoring


