# mantis docs

The full documentation for [`privacykey/mantis`](https://github.com/privacykey/mantis) — a self-hostable mantis key service. This folder is staged for extraction into its own repo at [`privacykey/mantis-docs`](https://github.com/privacykey/mantis-docs).

## Getting started

- [Getting started](./getting-started.md) — CLI install → first key, in five steps
- [Trying it locally](./trying-locally.md) — Docker evaluation, local-dev setup for contributors, benchmarks
- [Use cases](./use-cases.md) — defensive, detective, operational, and adversarial-research patterns

## Deployment

- [Overview & chooser](./deployment/README.md) — picking between local, tunnelled, and PaaS options
- [Local Docker](./deployment/docker-local.md) (option A)
- [Docker + Tailscale](./deployment/tailscale.md) (option B)
- [Docker + Cloudflare Tunnel](./deployment/cloudflare.md) (option C)
- [Railway](./deployment/railway.md) (option E1)
- [Fly.io](./deployment/fly.md) (option E2)
- [Render](./deployment/render.md) (option E3)
- [Edge limits](./deployment/edge-limits.md) — rate limiting / DDoS / WAF guidance
- [Backups](./deployment/backups.md) — Postgres backup strategies
- [mantis-edge worker](./edge-deployment.md) — Cloudflare Worker (stateless) variant

## Reference

- [HTTP API](./api.md) — endpoints, response kinds, webhook payload shape
- [Configuration](./configuration.md) — required and optional env vars
- [Updating](./updating.md) — update commands per component (server, CLI, edge worker, IoT helper)

## Features

- [File keys](./file-keys.md) — `.docx` / `.xlsx` / `.pptx` / `.pdf` / `.svg` / `.html` / `.md` / `.eml` / `.ics` / `.vcf`
- [Honey directory](./honey-directory.md) — 9-file `.zip` bundle for shared-drive deployment
- [Host-event keys](./host-events.md) — shell / login / boot / wake / network installers + web embeds + smart-home + the `X-Mantis-*` header reference
- [Uptime Kuma integration](./uptime-kuma.md) — fan-out via Kuma's 80+ notification channels
- [Reliability](./reliability.md) — hit dedup, retry queue, UA / bot parsing

## Operating

- [Single-user model](./single-user.md) — admin / non-admin behaviour
- [Operational notes](./operational-notes.md) — key hashing, disabled-key responses, worker model
- [Dev inbox](./dev-inbox.md) — built-in webhook capture for local dev

## Recipes

- [Self-hosted apps](./self-hosted-apps.md) — per-app recipes (Immich, Paperless, Joplin, Vaultwarden, dashboards, code hosts)

## Architecture

- [Project layout](./architecture.md) — directory map of the source tree
