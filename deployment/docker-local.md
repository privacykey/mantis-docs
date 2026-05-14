# A. Local Docker (no tunnel)

```bash
git clone <this repo> mantis && cd mantis
docker compose up -d
docker compose logs mantis | grep "bootstrap API key" -A1
```

Mantis is bound to `127.0.0.1:3000` by default. Reachable from the host only. To expose to LAN: set `MANTIS_BIND_HOST=0.0.0.0` in `.env`.

The Docker localhost setup is for **evaluation only** — don't rely on it for canaries that need to fire when you're not at your machine.
