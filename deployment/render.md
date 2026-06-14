---
title: "Render"
description: "Deploy Mantis on Render with a Blueprint and managed service."
sidebarTitle: "Render"
---

```bash
cp deploy/render.yaml.example render.yaml
# commit and push; then create a Render Blueprint pointing at the repo
```

The blueprint declares `MANTIS_API_KEY_PEPPER` as a required secret. Generate it
with `openssl rand -base64 32` when Render prompts for the value, and keep it
stable for the life of the database.

Free tier spins down after 15 min idle → first mantis trigger after idle takes 30–60s. For real mantis use, upgrade to Starter ($7/mo) or use Railway/Fly.

For app-layer URL/rate limits, use a Cloudflare-proxied custom domain; see **[edge limits](/deployment/edge-limits#render)**.
