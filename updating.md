# Updating

Pick the row that matches how each component was installed. After bumping the server, run `mantis doctor` to confirm CLI/server compatibility.

| Component | Install method | Update |
|---|---|---|
| **Server** | Docker (`docker compose up -d`) | `git pull && docker compose up -d --build` — `AUTO_MIGRATE=1` (set in `docker-compose.yml`) runs new migrations on container start. |
| **Server** | Local dev (`npm run dev`) | `git pull && npm install && npm run db:migrate` — then restart `npm run dev`. |
| **Server** | Railway / Render (Git-tracked) | `git push` to the tracked branch — both auto-redeploy. Migrations run on boot via `AUTO_MIGRATE=1`. |
| **Server** | Fly.io | `git pull && fly deploy`. |
| **CLI** | Homebrew (`brew install privacykey/tap/mantis`) | `brew update && brew upgrade mantis`. |
| **CLI** | Direct tarball from [releases](https://github.com/privacykey/mantis/releases?q=cli-v) | Download the new `mantis-<platform>.tar.gz`, extract, replace the binary on your `$PATH`. |
| **Edge worker** | `mantis-edge/` via wrangler | `git pull && cd mantis-edge && npm install && npx wrangler deploy` — secrets (`MANTIS_EDGE_KEY`, `MANTIS_EDGE_WEBHOOK_ALLOWLIST`) persist across deploys, no re-keying needed. |
| **IoT helper** | `iot-helper/` direct | `git pull && cd iot-helper && npm install` — then restart your `mantis-iot-helper.js` process. |
| **Host installers** | Per-machine snippets from `mantis install …` | Existing snippets keep firing as long as the trigger URL hasn't changed — only re-run `mantis install <key-id> --type <type> --out <path>` when a release note flags a snippet-format change. |

To get notified of new releases, **Watch → Custom → Releases** on the GitHub repo — `cli-v*` tags publish CLI binaries, plain `v*` tags publish server releases.
