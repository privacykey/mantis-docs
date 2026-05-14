---
title: "Operational notes"
description: "Implementation notes for key hashing, disabled-key behavior, and worker mode."
---

- **API keys and dashboard sessions are stored as SHA-256 hashes.** API keys are random 192-bit keys with the `mantis_live_` prefix (matches GitHub's secret-scanning format, so a leaked key in a public repo will trigger their alerting). Dashboard cookies carry opaque `mantis_sess_...` tokens; the database stores only their hash.
- **Disabled or expired keys** still return the configured response — they just don't record a hit or fire notifications. This prevents an attacker from probing for valid key IDs by looking for differential responses.
- **Notifications use a Postgres-backed queue.** Public trigger requests enqueue notification rows after the response path; a worker claims pending rows with `FOR UPDATE SKIP LOCKED`, sends them, and retries failures with backoff. The worker starts automatically on non-Vercel deployments. On Vercel or other serverless hosts, use `/api/cron/notifications` with `CRON_SECRET`.
