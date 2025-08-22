# Citus Cluster Backup Strategy with pgBackRest

This guide summarizes **best practices for backing up a Citus PostgreSQL cluster using pgBackRest**, including full, incremental, and WAL backups, object storage integration, and retention management.

---

## 1. Backup Types and Schedule

| Backup Type   | Frequency                   | Notes                                                                               |
| ------------- | --------------------------- | ----------------------------------------------------------------------------------- |
| Full Backup   | Weekly (e.g., Sunday 2 AM)  | Complete snapshot of all nodes. Baseline for incremental backups.                   |
| Incremental   | Daily (Mon–Sat 2 AM)        | Only changed files since last backup. Faster and storage-efficient.                 |
| WAL Archiving | Continuous / near real-time | Push all WAL segments to object storage. Enables **Point-In-Time Recovery (PITR)**. |

**Recommendation:**

* First backup is always full.
* Use incremental backups after the full backup.
* Stream WAL continuously to enable PITR.

---

## 2. Node Strategy for Citus

### Coordinator Node

* Backup metadata and configuration (stanza) + WAL.
* Essential for reconstructing shard mapping.

### Worker Nodes

* Backup each worker's local shards and WAL.
* Can be done in parallel.
* Push backups to a centralized object storage repository.

### Consistency

* Option 1: Pause writes briefly → backup coordinator → backup workers.
* Option 2: Use snapshot isolation via `pg_dump` or filesystem-level snapshot.

---

## 3. Retention Policy

```ini
retention-full=2
retention-diff=7
```

* Keep last 2 full backups.
* Keep last 7 incremental/differential backups.
* Keep enough WAL segments to cover the PITR window.

---

## 4. Object Storage Integration

* Use S3, MinIO, GCS, or Azure Blob.
* Organize bucket like:

```
/citus_cluster/
  backup/
    YYYYMMDD-HHMMSSF/  # full
    YYYYMMDD-HHMMSSI/  # incremental
  archive/              # WAL files
```

* Benefits: durable offsite storage, supports replication, lifecycle policies.

---

## 5. Monitoring & Testing

* Automated health checks: `pgbackrest info` to verify backup success.
* Alert on failures.
* Test restores regularly:

  * Full restore to staging environment monthly.
  * PITR restore randomly to ensure WAL recovery works.

---

## 6. Recovery Workflow

### Scenario: Data loss on Wednesday

1. Restore last full backup (Sunday).
2. Apply incremental backups in chronological order (Mon → Tue → Wed).
3. Apply WAL segments for point-in-time recovery if needed.

**Object Storage Layout Example:**

```
backup/
 ├─ 20250820-020000F/   # Full (Sunday)
 ├─ 20250821-020000I/   # Incremental (Mon)
 ├─ 20250822-020000I/   # Incremental (Tue)
 ├─ 20250823-020000I/   # Incremental (Wed)
archive/
 ├─ WAL segments streamed continuously
```

* Order of restoration matters: Full → Incrementals → WAL.

---

## 7. Handling Failed Backups

* Failed backups are logged in `pgbackrest.info`.
* They do not affect retention of previous successful backups.
* Retention only removes **old successful backups** according to the policy.
* Monitor using:

```bash
pgbackrest info --stanza=citus_cluster
```

---

## 8. Summary of Best Practices

1. First backup always **full**, then incremental.
2. **WAL streaming** for continuous recovery.
3. **Backup all nodes** (coordinator + workers).
4. **Parallel backups** on workers for large datasets.
5. **Store backups in durable object storage**.
6. **Automate and monitor** backups; test restores regularly.
7. **Retention policy** balances storage usage and recovery requirements.

---

## 9. Sample Cron Schedule

```yaml
# Full backup on Sunday
cron:
  name: "pgBackRest Full Backup"
  user: postgres
  weekday: 0
  hour: 2
  minute: 0
  job: "pgbackrest --stanza=citus_cluster --type=full backup"

# Incremental backup Mon–Sat
cron:
  name: "pgBackRest Incremental Backup"
  user: postgres
  weekday: 1-6
  hour: 2
  minute: 0
  job: "pgbackrest --stanza=citus_cluster --type=incr backup"
```

* This schedule ensures full weekly backups with daily incrementals.
* WAL streaming handles point-in-time recovery between backups.

---

**Note:** For a Citus cluster, ensure **coordinator metadata** and **worker shards** are backed up consistently, either via snapshot isolation or coordinated backup orchestration.
