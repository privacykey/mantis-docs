# Dev inbox

A built-in webhook capture, only enabled with `ENABLE_DEV_INBOX=1`:

- `ANY http://localhost:3000/inbox/<any-path>` — captures the request into a 100-entry in-memory ring buffer
- `GET /inbox` — live-updating viewer (polls every 1s, pretty-prints JSON bodies)
- `GET /api/inbox` — same data as JSON; supports `?slug=foo&since=<id>`
- `DELETE /api/inbox` — clear the buffer

Use it as your webhook target while developing, instead of webhook.site. State is in-memory only — it resets when the server restarts. Do not expose it in production.
