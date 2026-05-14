# Trying it locally

Two flavours of "running mantis on your laptop": the **Docker quickstart** is for kicking the tyres before you provision a real server; the **local dev** flow is for contributors who want to edit the source.

## Docker quickstart (evaluation)

Run the stack against your laptop with Docker:

```bash
git clone <this repo> mantis && cd mantis
docker compose up -d
docker compose logs mantis | grep "bootstrap API key" -A1
```

The first boot prints a bootstrap API key. Copy it; it won't be shown again.

Then drive the CLI as usual, just pointing at localhost:

```bash
mantis --key mantis_live_... login --url http://localhost:3000

# The dev image ships an inbox at /inbox; webhook captures land there with
# zero external setup.
mantis new "first mantis" -w http://localhost:3000/inbox/demo
open http://localhost:3000/inbox
mantis hits last --follow
```

When you're ready to move from "trying it" to "running it", pick a real deployment from [Deployment](./deployment/README.md) and `mantis --profile prod login --url https://...` against it. The Docker localhost setup is for evaluation only — don't rely on it for canaries that need to fire when you're not at your machine.

## Local dev (for contributors)

```bash
npm install
cp .env.example .env
# edit .env: at minimum set DATABASE_URL

# Start postgres separately (docker run -p 5432:5432 ... or use Neon/Supabase)
npm run db:migrate
npm run dev
```

First request prints a bootstrap API key to stdout.

## Performance checks

Benchmarks live under [`bench/`](../bench/README.md) and are intentionally local, not CI-gated:

```bash
npm run bench:cli                  # CLI startup/rendering overhead
npm run bench:edge                 # mantis-edge worker, requires `npx wrangler dev` running
MANTIS_BENCH_KEY=mantis_live_... npm run bench:server
```
