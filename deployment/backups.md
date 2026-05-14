---
title: "Backups and retention"
description: "Back up and restore the Postgres database that stores Mantis data."
sidebarTitle: "Backups"
---

Mantis stores everything in Postgres. Backup = `pg_dump` the database.

Two reasons to back up:

- **Disaster recovery** — restore on a new host if the server dies.
- **Schema rollback** — migrations are forward-only. If an upgrade damages data or you need the old shape back, only a backup recovers it cleanly.

## Local Postgres (docker compose)

```bash
docker compose exec -T postgres pg_dump -U mantis -d mantis -Fc > "mantis-$(date +%Y%m%d).dump"
```

`-Fc` = custom format, smaller and restore-flexible.

Restore on a fresh container:

```bash
docker compose down -v postgres   # wipes the volume
docker compose up -d postgres
sleep 5
docker compose exec -T postgres pg_restore -U mantis -d mantis --no-owner < mantis-20260513.dump
```

## Managed Postgres (Neon / Supabase / Railway / Fly Postgres)

Each provider has its own snapshot mechanism. Don't rely on theirs alone — they all have edge cases (Neon's free branches expire, Railway's snapshots disappear with a project delete). Schedule your own `pg_dump` to S3 / B2 / GCS in parallel.

```bash
# Cron entry, runs daily at 03:17
17 3 * * * pg_dump "$DATABASE_URL" -Fc | aws s3 cp - s3://my-backups/mantis-$(date +\%F).dump
```

## What's in there

| Table | Backup-worthy? |
|---|---|
| `api_keys` | Yes — losing this means re-bootstrapping & re-distributing keys |
| `keys` | Yes — losing this loses every configured mantis URL |
| `hits` | Yes (but bounded by retention) — forensic history |
| `notifications` | Yes (bounded by retention) — delivery state, in-flight retries |
| `notification_destinations` | Yes — channel config + activation status |
| `audit_events` | Yes — operator and key-management history |
| `sessions` | Usually — dashboard login sessions; can be dropped if you are comfortable logging in again |
| `wallet_config` | Yes — Pass Type ID cert + password (sensitive!) |
| `wallet_registrations` | Yes if using Apple Wallet — device push tokens for pass update notifications |

`wallet_config` contains the .p12 password in plaintext, and notification
tables can contain webhook targets and signing secrets. Dump files should be
encrypted at rest (`gpg --symmetric`, or rely on bucket-level KMS).

## Retention

Hit / notification growth is the table to watch. With heavy use a single high-traffic key can add 10k+ rows/day.

Built-in retention (see `.env.example`):

```bash
MANTIS_HIT_RETENTION_DAYS=90           # delete hits + their notifications
MANTIS_NOTIFICATION_RETENTION_DAYS=30  # delete settled notifications
MANTIS_AUDIT_RETENTION_DAYS=365        # delete audit events
MANTIS_SESSION_RETENTION_DAYS=7        # delete expired/revoked sessions
```

The notify worker sweeps hourly. In serverless cron mode, the same sweep runs
through `/api/cron/notifications`. All unset = retain forever — appropriate for
low-traffic deployments.

Manual cleanup if you don't want a fixed retention:

```sql
-- Delete a specific noisy key's hits older than a week
DELETE FROM hits
WHERE key_id = '<uuid>' AND occurred_at < now() - interval '7 days';

-- Compact disk after large deletes
VACUUM FULL hits;  -- holds an exclusive lock; offline-only
-- ...or in normal operation:
VACUUM ANALYZE hits;
```

## Restoring a single key

If you accidentally `DELETE FROM keys WHERE id = ...`, restoring just that row from a dump:

```bash
pg_restore -a -t keys -t hits -t notifications --data-only \
  -d mantis-restore mantis-20260513.dump
# then INSERT … SELECT into the live DB
```

Easier: keep a full `mantis-restore` DB around for the most recent dump and copy rows over as needed.
