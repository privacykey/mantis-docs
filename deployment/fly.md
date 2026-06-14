---
title: "Fly.io"
description: "Deploy Mantis to Fly.io with Postgres and public edge guidance."
---

```bash
cp deploy/fly.toml.example fly.toml
# edit `app =` to a unique name; edit PUBLIC_BASE_URL after first deploy
fly launch --no-deploy --copy-config
fly postgres create --name mantis-db
fly postgres attach mantis-db
fly secrets set \
  PUBLIC_BASE_URL=https://<app>.fly.dev \
  MANTIS_API_KEY_PEPPER="$(openssl rand -base64 32)" \
  BOOTSTRAP_API_KEY=mantis_live_...
fly deploy
```

Fly concurrency protects the VM, not the public URL. For app-layer URL/rate limits, use a Cloudflare-proxied custom domain; see **[edge limits](/deployment/edge-limits#flyio)**.
