---
title: "HTTP API"
description: "Reference for key management endpoints, trigger responses, and webhook payloads."
sidebarTitle: "HTTP API"
---

The CLI is the recommended interface, but everything it does is exposed over a plain JSON HTTP API. Useful for scripts in languages without a Mantis client, or for testing the trigger path with `curl`:

```bash
export MANTIS=http://localhost:3000
export KEY=mantis_live_...   # `mantis login` output, or read from the bootstrap log

# Mint a key
curl -sX POST $MANTIS/api/keys \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d '{"memo":"first mantis","destinations":[{"channel":"webhook","target":"https://webhook.site/<your-id>"}]}'
# → { "id":"...", "public_id":"...", "url":"http://localhost:3000/c/...", ... }

# Trigger it
curl -i $MANTIS/c/<public_id>
# → 200 OK with a 1×1 transparent GIF

# Inspect hits
curl -s $MANTIS/api/keys/<id>/hits -H "Authorization: Bearer $KEY" | jq
```

## Endpoint reference

Management endpoints are API-key authenticated with `Authorization: Bearer mantis_live_...`.
Dashboard helper endpoints that need browser access also accept the `mantis_session`
httpOnly cookie. Public trigger, status, health, wallet, and dev-inbox routes
are called out explicitly below.

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/api/keys` | Create a key. Bearer auth. Body: `{ memo, response_kind?, response_payload?, destinations?, expires_at?, dedupe_window_seconds?, monitor_mode?, monitor_window_seconds? }` |
| `GET` | `/api/keys?limit=&cursor=` | List accessible keys. Bearer auth. Admin keys see all; non-admin keys see rows they created. |
| `GET` | `/api/keys/:id` | Get one accessible key. Bearer auth. |
| `PATCH` | `/api/keys/:id` | Update memo, response, expiry, dedupe, monitor mode/window, `disabled`, or replace destinations. Bearer auth. |
| `DELETE` | `/api/keys/:id` | Hard-delete a key and cascading hits/notifications. Bearer auth. |
| `GET` | `/api/keys/:id/hits?limit=&cursor=` | Paginated hit log for one key. Bearer auth. |
| `GET` | `/api/hits/recent?since=<iso>&cursor=<iso>&key_id=<id>&limit=<n>` | Recent hit feed across accessible keys, used by CLI watch mode. Bearer auth. |
| `GET` | `/api/keys/:id/download?format=<format>` | Download a generated artifact. Bearer or session auth. Formats: `docx`, `xlsx`, `pptx`, `pdf`, `folder`, `nfc-label`, `apple-wallet`, `svg`, `html`, `md`, `eml`, `ics`, `vcf`. |
| `GET` | `/api/keys/:id/install?type=<type>[&hostname=example.com][&format=json]` | Generated installer snippet for host, web, NFC, and IoT events. Bearer or session auth. |
| `POST` | `/api/keys/:id/reset` | Reset a key's latched monitor state. Bearer or session auth. |
| `POST` | `/api/keys/:id/destinations/:destinationId/signing-secret` | Reveal a webhook destination's plaintext HMAC signing secret. Bearer or session auth. Audited. |
| `POST` | `/api/keys/:id/destinations/:destinationId/rotate-secret` | Rotate a webhook destination's HMAC signing secret and return the new secret once. Bearer or session auth. Audited. |
| `GET` | `/api/api-keys` | List API keys. Bearer auth. Hashes are never returned; non-admin keys see only themselves. |
| `POST` | `/api/api-keys` | Mint a new API key. Bearer auth. Body: `{ name, is_admin? }`; plaintext key returned once. Only admins can mint admin keys. |
| `DELETE` | `/api/api-keys/:id` | Revoke an API key. Bearer auth. Self-revoke is allowed; revoking others requires admin. |
| `GET` | `/api/audit?limit=&cursor=&since=&event_type=&actor=` | Admin-only audit log. Bearer or session auth. |
| `GET` `HEAD` | `/api/health` | **Public unless gated by your proxy.** Liveness + `SELECT 1` readiness. 200 = app and DB ok, 503 = DB failure. |
| `GET` `POST` | `/api/cron/notifications?max=<n>` | Notification retry and retention worker endpoint for serverless deployments. Requires `Authorization: Bearer $CRON_SECRET`; returns 401 if `CRON_SECRET` is unset. |
| `GET` `HEAD` | `/status/:public_id` | **Public.** Uptime-monitor status endpoint. 200 = ok, 503 = tripped, 404 = not monitored / disabled / expired / unknown. Does not record a hit. |
| `GET` `HEAD` `POST` | `/c/:public_id` | **Public.** Records a hit, returns the configured response. |
| `ANY` | `/inbox/<slug>` | **Dev only.** Captures webhook-style requests when `ENABLE_DEV_INBOX=1`. |
| `GET` `DELETE` | `/api/inbox` | **Dev only.** Read or clear the in-memory dev inbox when `ENABLE_DEV_INBOX=1`. Bearer or session auth. |
| `POST` | `/api/wallet/v1/log` | **Public Apple Wallet web service.** Accepts Wallet diagnostics. |
| `GET` | `/api/wallet/v1/passes/:passTypeId/:serial` | **Public Apple Wallet web service.** Authenticates `Authorization: ApplePass <token>`, records `wallet-fetched`, returns the latest `.pkpass`. |
| `GET` | `/api/wallet/v1/devices/:deviceId/registrations/:passTypeId` | **Public Apple Wallet web service.** Wallet update poll; returns 204 and does not record a hit. |
| `POST` `DELETE` | `/api/wallet/v1/devices/:deviceId/registrations/:passTypeId/:serial` | **Public Apple Wallet web service.** Records `wallet-installed` on registration and `wallet-uninstalled` on removal. |

Parsed request bodies are bounded. `POST /api/keys` and
`POST /api/api-keys` reject JSON bodies over 64 KiB with
`413 payload_too_large`. Apple Wallet log bodies are capped at 32 KiB, and the
dev inbox capture path is capped at 1 MiB.

Supported installer `type` values are `shell`, `shell-sudo`,
`macos-login`, `macos-boot`, `macos-wake`, `macos-network`,
`linux-boot`, `linux-wake`, `linux-network`, `windows-logon`,
`windows-wake`, `windows-network`, `css-background`, `js-clone-detector`,
`nfc-ndef`, `homeassistant`, `homeassistant-receiver`, and `scrypted`.

Webhook destinations get an HMAC secret. Outbound raw-webhook deliveries include
`X-Mantis-Timestamp` and `X-Mantis-Signature: sha256=<hex>` over
`<timestamp>.<json body>`. The plaintext secret is only shown on create, replace,
explicit reveal, or rotate responses; normal listing returns a fingerprint.

## Response kinds for the trigger endpoint

| `response_kind` | Payload | Result |
|---|---|---|
| `gif` (default) | — | `200` + 1×1 transparent GIF |
| `empty` | — | `204 No Content` |
| `json` | any | `200` + JSON body |
| `redirect` | `{"url":"..."}` | `302` to that HTTP(S) URL with no-store cache headers |
| `html` | `{"html":"..."}` | `200 text/html` with no-store cache headers and a restrictive CSP sandbox |

## Webhook payload

```json
{
  "type": "mantis.hit",
  "key": { "id": "...", "public_id": "...", "memo": "...", "url": "..." },
  "hit": {
    "id": "...",
    "occurred_at": "2026-05-12T10:00:00.000Z",
    "ip": "203.0.113.5",
    "user_agent": "Mozilla/5.0 ...",
    "referer": null,
    "ua_browser": "Chrome",
    "ua_os": "macOS",
    "ua_device": "desktop",
    "bot_label": null,
    "is_duplicate": false,
    "headers": { "host": "...", "accept": "...", "x-mantis-source": "shell", "x-mantis-user": "alice", "x-mantis-ssh-client": "203.0.113.42 54321 22", "..." : "..." }
  }
}
```

Webhook payloads also include a parsed `host_context` object when the hit came from one of our installer snippets:

```json
"host_context": {
  "source": "shell",
  "user": "alice",
  "host": "alice-mbp",
  "ssh_client": "203.0.113.42 54321 22",
  "ssh_client_ip": "203.0.113.42",
  "ssh_connection": "203.0.113.42 54321 10.0.0.5 22",
  "tty": "/dev/pts/0",
  "sudo_cmd": null,
  "network_interface": null
}
```

For a `shell-sudo` hit you'd see `"source": "shell-sudo", "sudo_cmd": "apt update --quiet"`. For a `linux-network` hit, `"source": "linux-network", "network_interface": "wlan0"`. Fields not relevant to the installer are `null`.

`host_context` is `null` for hits that didn't include `X-Mantis-*` headers (e.g., a file/folder key, or a regular curl to the URL).

Webhooks are sent through a **Postgres-backed retry queue** (no Redis required). On failure the notification is retried with exponential backoff: 1m, 5m, 30m, 2h, 12h (each with ±20% jitter), giving up after 5 attempts. Delivery state is tracked on each hit's `notifications` array.
