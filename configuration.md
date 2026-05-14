---
title: "Configuration"
description: "Required and optional environment variables for running Mantis."
---

See `.env.example` in the main Mantis repo for the canonical, commented list.
These are the variables operators usually need to understand.

## Required

- `DATABASE_URL` — Postgres connection string.

## URLs

- `PUBLIC_BASE_URL` — public origin used when Mantis generates trigger, status, and Wallet callback URLs. Defaults to `http://localhost:3000`.
- `MANTIS_PUBLIC_PATH` — trigger path prefix, default `/c`. Changing this changes generated URLs and the public-only host allowlist; if you choose a different path, put a reverse proxy in front that rewrites it to `/c/<id>` on the app.

## Boot and runtime

- `BOOTSTRAP_API_KEY` — pre-seed the first admin API key. If unset and no API keys exist, Mantis mints one and prints it on first boot.
- `AUTO_MIGRATE=1` — apply pending migrations on app boot. Useful for long-running container deploys; avoid for Vercel/serverless.
- `RUN_NOTIFY_WORKER=0|1` — notification worker switch. It runs by default outside Vercel; set `0` when using cron mode instead.
- `CRON_SECRET` — required to enable `GET`/`POST /api/cron/notifications`. The endpoint fails closed with 401 when unset.
- `LOG_LEVEL` — `trace|debug|info|warn|error`, default `info`.

## Notification destinations

- `SMTP_URL` — enables email notification destinations. Example: `smtp://user:pass@host:587`.
- `SMTP_FROM` — email sender, default `Mantis <mantis@localhost>`.
- `ALLOW_PRIVATE_WEBHOOKS=1` — allow webhook/Slack/Discord/Teams targets that resolve to private, loopback, link-local, or cloud-metadata IPs. Default is blocked as an SSRF guard.

## Proxy and public-only hosts

- `TRUST_PROXY_HEADERS=1` — trust `cf-connecting-ip`, `x-vercel-forwarded-for`, `x-real-ip`, and `x-forwarded-for` for forensic IP logging. Set only behind a proxy that strips and re-injects those headers. Auto-enabled on Vercel and in non-production.
- `PUBLIC_ONLY_HOSTS` — comma/space-separated hostnames that should expose only public routes: trigger URLs, status URLs, and Wallet callbacks.
- `DASHBOARD_HOSTS` — hostnames allowed to serve the dashboard and management API. When `PUBLIC_ONLY_HOSTS` is set, unknown hosts fail closed to public-only unless explicitly listed here.
- `PUBLIC_ONLY_ALLOW_HEALTH=1` — allow `/api/health` on public-only hosts.
- `PUBLIC_ONLY_ALLOW_INBOX=1` — allow `/inbox` and `/api/inbox` on public-only hosts. Use only for deliberate dev boxes.

## Hit storage

- `MANTIS_DUPLICATE_LOG_LIMIT` — duplicate hit rows to store per dedupe window after the first hit. Default `10`; `0` stores only the first hit in the window.
- `MANTIS_MAX_STORED_REQUEST_FIELD_CHARS` — cap for stored `user-agent`, `referer`, and individual header values. Default `16384`.
- `MANTIS_MAX_STORED_HEADER_SNAPSHOT_CHARS` — cap for the total stored header snapshot. Default `65536`.

## Retention

Unset retention variables mean retain forever. The notify worker sweeps hourly;
cron mode runs the same sweep through `/api/cron/notifications`.

- `MANTIS_HIT_RETENTION_DAYS` — delete hits older than N days; notifications cascade.
- `MANTIS_NOTIFICATION_RETENTION_DAYS` — delete settled notifications older than N days.
- `MANTIS_AUDIT_RETENTION_DAYS` — delete append-only audit events older than N days.
- `MANTIS_SESSION_RETENTION_DAYS` — delete revoked or expired dashboard sessions older than N days. Active sessions are not affected.

## Dev inbox

- `ENABLE_DEV_INBOX=1` — enable the unauthenticated in-memory `/inbox` webhook capture and `/api/inbox` JSON API. Leave off in production.

## Docker and tunnels

- `MANTIS_BIND_HOST` — compose host bind address, default `127.0.0.1`.
- `MANTIS_HOST_PORT` — compose host port, default `3000`.
- `TS_AUTHKEY`, `TS_HOSTNAME`, `TS_PRIVATE_HOSTNAME`, `TS_PUBLIC_HOSTNAME`, `TS_EXTRA_ARGS` — Tailscale sidecar configuration.
- `CLOUDFLARE_TUNNEL_TOKEN` — Cloudflare Tunnel sidecar token.

## Apple Wallet

Apple Wallet can be configured with env vars or from the admin dashboard at
`/settings/wallet`. Env vars take precedence over the DB-stored dashboard config.

Required to enable `.pkpass` generation:

- `APPLE_PASS_CERT_PATH` — Pass Type ID `.p12` certificate.
- `APPLE_PASS_CERT_PASS` — password for the `.p12`.
- `APPLE_PASS_TEAM_ID` — 10-character Apple Team ID.
- `APPLE_PASS_TYPE_ID` — Pass Type ID, for example `pass.com.example.mantis`.
- `APPLE_PASS_AUTH_SECRET` — random secret used to derive per-pass Wallet auth tokens.

Optional:

- `APPLE_PASS_WWDR_PATH` — Apple WWDR intermediate PEM override. Mantis also bundles a WWDR cert.
- `APPLE_PASS_ICON_PATH` — custom 58x58 PNG icon.
- `APPLE_PASS_LOGO_PATH` — custom 160x50 PNG logo.
- `APPLE_PASS_ORG_NAME` — pass organization name, default `Mantis`.
- `APPLE_PASS_APNS_KEY_PATH` — APNs `.p8` auth key for pass-update pushes.
- `APPLE_PASS_APNS_KEY_ID` — APNs key ID.
- `APPLE_PASS_APNS_SANDBOX=1` — use APNs sandbox instead of production.
