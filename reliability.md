---
title: "Reliability"
description: "Understand hit deduplication, notification retries, worker mode, and bot parsing."
---

## Hit dedup

Each key has a `dedupe_window_seconds` (default **60**). Repeat hits to the same key within that window are still recorded (so forensics are intact), but they are marked `is_duplicate: true` and don't fire notifications. Set to `0` to disable.

## Notification retry queue

Notifications are inserted into a `notifications` table on hit. A background worker claims pending rows (`FOR UPDATE SKIP LOCKED` — safe for multiple instances), attempts delivery with a 5s timeout, and on failure schedules the next attempt with exponential backoff. Status (`pending` / `in_flight` / `succeeded` / `failed` / `aborted`) is surfaced in the dashboard, CLI (`mantis hits <id>`), and API (`GET /api/keys/<id>/hits`).

**Worker mode** (default for non-Vercel deployments):
- `instrumentation.ts` starts the worker on boot. Disable with `RUN_NOTIFY_WORKER=0`.

**Cron mode** (Vercel and other serverless):
- Worker is skipped automatically when `VERCEL` env is set.
- Configure Vercel Cron (or external scheduler) to hit `POST /api/cron/notifications` every minute.
- Set `CRON_SECRET`; the endpoint requires `Authorization: Bearer $CRON_SECRET` and returns `401` when the secret is unset.

```json
// vercel.json
{
  "crons": [{ "path": "/api/cron/notifications", "schedule": "* * * * *" }]
}
```

Both worker mode and cron mode run the same retention sweep. If
`MANTIS_HIT_RETENTION_DAYS`, `MANTIS_NOTIFICATION_RETENTION_DAYS`,
`MANTIS_AUDIT_RETENTION_DAYS`, or `MANTIS_SESSION_RETENTION_DAYS` are set,
old rows are purged during the hourly worker loop or the cron invocation.

## User-Agent + bot detection

Hits are enriched with parsed UA (`ua_browser`, `ua_browser_version`, `ua_os`, `ua_device`) via `ua-parser-js`, plus a `bot_label` for known crawlers and HTTP clients (googlebot, curl, python-requests, headless-chrome, etc.). Bots aren't suppressed — they're labeled so you can decide.
