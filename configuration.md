# Configuration

See `.env.example` for the full list. Required:

- `DATABASE_URL` — Postgres connection string
- `PUBLIC_BASE_URL` — used to construct key URLs

Optional:

- `SMTP_URL`, `SMTP_FROM` — enables email notifications
- `BOOTSTRAP_API_KEY` — pre-seed the first API key (for IaC). If unset, one is minted and printed on first boot.
- `AUTO_MIGRATE=1` — apply pending migrations on app boot (use on container deploys, not Vercel)
- `MANTIS_PUBLIC_PATH` — defaults to `/c`. Changing this only affects the URLs the API advertises; to actually serve from a different path, put a reverse proxy in front and rewrite to `/c/<id>`.
- `LOG_LEVEL` — `trace|debug|info|warn|error` (default `info`)
- `MANTIS_DUPLICATE_LOG_LIMIT` — duplicate hit rows to store per dedupe window after the first hit (default `10`; `0` stores only the first).
- `MANTIS_MAX_STORED_REQUEST_FIELD_CHARS` — cap for stored `user-agent`, `referer`, and individual header values (default `16384`).
- `MANTIS_MAX_STORED_HEADER_SNAPSHOT_CHARS` — cap for the total stored header snapshot (default `65536`).
- `ENABLE_DEV_INBOX` — `1` to enable the unauthenticated `/inbox` webhook capture. Default: disabled.
