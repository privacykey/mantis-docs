# HTTP API

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

All `/api/*` endpoints require `Authorization: Bearer mantis_live_...`.

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/api/keys` | Create a key. Body: `{ memo, response_kind?, response_payload?, destinations?, dedupe_window_seconds?, expires_at? }` |
| `GET` | `/api/keys` | List keys (paginated, `?limit=&cursor=`) |
| `GET` | `/api/keys/:id` | Get one |
| `PATCH` | `/api/keys/:id` | Update memo/notify/response/`disabled` |
| `DELETE` | `/api/keys/:id` | Hard-delete key + cascade hits |
| `GET` | `/api/keys/:id/hits` | Paginated hit log |
| `GET` | `/api/hits/recent?since=<iso>&key_id=<id>` | Recent hit feed across accessible keys, used by CLI watch mode |
| `GET` | `/api/keys/:id/download?format=docx\|xlsx\|pptx\|pdf\|folder` | Download a generated mantis file or `folder` honey-directory zip (cookie or Bearer auth) |
| `GET` | `/api/keys/:id/install?type=<type>[&format=json]` | Generated installer snippet for host/web/IoT events (shell/macOS/Linux/Windows/Home Assistant/Scrypted) — cookie or Bearer auth |
| `POST` | `/api/keys/:id/reset` | Reset a key's latched monitor state. Cookie or Bearer auth. |
| `GET` `HEAD` | `/status/:public_id` | **Public.** Uptime-monitor status endpoint. 200 = ok, 503 = tripped, 404 = not monitored / disabled / unknown. Does not record a hit. |
| `GET` | `/api/api-keys` | List keys (hashes never returned) |
| `POST` | `/api/api-keys` | Mint a new key (plaintext returned **once**) |
| `DELETE` | `/api/api-keys/:id` | Revoke (soft) |
| `GET` `HEAD` `POST` | `/c/:public_id` | **Public.** Records a hit, returns the configured response. |

## Response kinds for the trigger endpoint

| `response_kind` | Payload | Result |
|---|---|---|
| `gif` (default) | — | `200` + 1×1 transparent GIF |
| `empty` | — | `204 No Content` |
| `json` | any | `200` + JSON body |
| `redirect` | `{"url":"..."}` | `302` to that URL |
| `html` | `{"html":"..."}` | `200 text/html` |

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
