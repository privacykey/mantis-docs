# Operational notes

- **API keys are stored as SHA-256 hashes.** Keys are random 192-bit keys with the `mantis_live_` prefix (matches GitHub's secret-scanning format, so a leaked key in a public repo will trigger their alerting). Bcrypt/argon2 is not used here — slow hashing only helps against low-entropy passwords; for full-entropy API keys, SHA-256 is the standard (GitHub PATs, Stripe restricted keys, etc.).
- **Disabled or expired keys** still return the configured response — they just don't record a hit or fire notifications. This prevents an attacker from probing for valid key IDs by looking for differential responses.
- **No long-running background workers.** Notifications dispatch via Next.js `after()`, retries via a Postgres-backed queue (no Redis required) — both run in the request lifecycle, so the same code runs on Vercel Functions, Railway, Fly, and self-host Docker without any worker config.
