---
title: "Project layout"
description: "Directory map of the Mantis source tree and major application modules."
sidebarTitle: "Project layout"
---

```
src/                         # Next.js server: dashboard, API, public triggers
  app/
    page.tsx                 # / redirects to /keys or /login
    login/                   # API-key login form; mints httpOnly sessions
    logout/                  # POST clears the session cookie
    (app)/                   # authenticated dashboard route group
      layout.tsx             # shared nav: keys, new, admin-only wallet settings
      keys/
        page.tsx             # key list, enable/disable controls
        new/                 # create form
        [id]/                # detail, hits, downloads, installers, destinations
      settings/wallet/       # admin Apple Wallet / PassKit config
    api/
      keys/...               # authenticated key CRUD, hits, downloads, installers
      api-keys/...           # authenticated API-key listing, minting, revocation
      hits/recent/           # CLI watch feed across accessible keys
      audit/                 # admin audit log
      health/                # liveness + DB readiness
      cron/notifications/    # serverless notification/retention worker endpoint
      inbox/                 # dev inbox JSON API
      wallet/v1/...          # Apple Wallet web-service callbacks
    c/[publicId]/            # public trigger endpoint
    status/[publicId]/       # public monitor endpoint for Uptime Kuma
    inbox/                   # dev webhook capture + viewer
  db/                        # Drizzle schema and SQL migrations
  lib/
    auth.ts                  # Bearer API key auth and per-key ownership checks
    session.ts               # dashboard session cookies backed by hashed tokens
    env.ts                   # URL, SMTP, bootstrap, and Wallet env resolution
    keys.ts                  # public ID and API serialization helpers
    response.ts              # gif / empty / json / redirect / html trigger output
    docs/                    # generated bait artifacts: Office, PDF, Wallet, NFC, etc.
    installers/              # host, web, IoT, NFC, and Wallet installer generation
    notify/                  # destinations, activation pings, queue, sender worker
    monitor.ts               # latch/window status logic
    public-only-hosts.ts     # split public/dashboard host routing
    retention.ts             # optional row-level cleanup
  proxy.ts                   # Host/path guard for public-only deployments
  instrumentation.ts         # boot hook: migrations, bootstrap key, notify worker

cli/                         # @mantis/cli terminal client
  src/commands/              # one file per CLI command
  src/lib/                   # API client, keychain config, output, installers
mantis-edge/                 # stateless Cloudflare Worker backend
iot-helper/                  # optional LAN/log watcher for IoT-style triggers
bench/                       # local performance checks
tests/                       # Vitest coverage for app and worker behavior
deploy/                      # Fly and Render deployment examples
docker/                      # compose sidecar assets
docker-compose.yml           # local, Tailscale, and Cloudflare Tunnel profiles
drizzle.config.ts
pnpm-workspace.yaml
```
