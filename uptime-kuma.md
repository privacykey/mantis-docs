# Uptime Kuma integration

Uptime Kuma integration lets you piggyback on [Uptime Kuma](https://github.com/louislam/uptime-kuma)'s 80+ notification integrations when you prefer status-monitor fan-out over mantis's built-in notification destinations.

The mechanic: every key has a per-key *status URL* at `/status/<public_id>` that flips between OK (HTTP 200) and tripped (HTTP 503) when the mantis fires. Point a Uptime Kuma HTTP(s) monitor at that URL — Kuma detects the status code or body change and fires its configured notifications.

## Modes

| `monitor_mode` | When tripped | When does it clear? |
|---|---|---|
| `off` (default) | never (status URL returns 404) | — |
| `latch` | once *any* hit recorded | manual reset via `POST /api/keys/<id>/reset` |
| `window` | any hit within `monitor_window_seconds` (default 300) | automatically when the newest hit ages out |

## Setup walkthrough

```bash
# 1. Pick a key and enable monitoring
mantis monitor <key-id> --mode latch
#   → status URL: http://mantis.example.com/status/<public_id>
#   → current:    ok

# 2. In Uptime Kuma → Add new monitor:
#      Monitor type:        HTTP(s)
#      URL:                 http://mantis.example.com/status/<public_id>
#      Heartbeat Interval:  30s (or longer; Kuma minimum 20s)
#      Status code accepted: 200
#      → Save and attach your preferred notifications

# 3. Test it: trigger the mantis URL
curl http://mantis.example.com/c/<public_id>

# Within 30s, Uptime Kuma sees the status URL flip 200 → 503,
# and fires its configured Discord/Slack/Teams/whatever notifications.

# 4. When you've acknowledged the alert:
mantis reset <key-id>   # in latch mode; window mode auto-resets
```

## Status endpoint behavior

| Key state | `GET /status/<public_id>` |
|---|---|
| Doesn't exist | `404` `{"error":"not_monitored"}` |
| `monitor_mode = off` | `404` `{"error":"not_monitored"}` (same as nonexistent — no info leak) |
| Disabled or expired | `404` `{"error":"not_monitored"}` (Uptime Kuma won't keep alerting on a key you've shut down) |
| Active, no trip | `200` `{"status":"ok","mode":"latch"\|"window"}` |
| Active, tripped | `503` `{"status":"tripped","tripped_at":"<iso>","mode":...}` |

All responses include `Cache-Control: no-store`. The status endpoint **does not record a hit** — Uptime Kuma can poll it forever without filling your hits log.

## Notes

- Same key still records hits and dispatches configured notification destinations on `/c/<public_id>` — `/status/<public_id>` is a separate read-only reflection of state.
- Uptime Kuma is optional. The status URL is plain HTTP(s); any monitor that watches for status-code or body changes (e.g., Pingdom, BetterUptime, healthchecks.io, your own cron) works.
- In `latch` mode, switching to `off` then back to `latch` does not lose trip state — it's derived from `hits` filtered by `monitor_reset_at`.
