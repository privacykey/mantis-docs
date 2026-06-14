---
title: "Local Docker"
description: "Run Mantis locally with Docker for evaluation or development."
sidebarTitle: "Local Docker"
---

```bash
git clone <this repo> mantis && cd mantis
cp .env.example .env
pepper="$(openssl rand -base64 32)"
sed -i.bak "s|^MANTIS_API_KEY_PEPPER=.*|MANTIS_API_KEY_PEPPER=$pepper|" .env
rm .env.bak
docker compose up -d
docker compose logs mantis | grep "bootstrap API key" -A1
```

Mantis is bound to `127.0.0.1:3000` by default. Reachable from the host only. To expose to LAN: set `MANTIS_BIND_HOST=0.0.0.0` in `.env`.

`MANTIS_API_KEY_PEPPER` is required. Generate it once and keep it stable for
that database; rotating it invalidates existing API keys.

The Docker localhost setup is for **evaluation only** — don't rely on it for canaries that need to fire when you're not at your machine.
