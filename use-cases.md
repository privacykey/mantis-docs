# What can you actually use this for?

A mantis URL fires whenever someone fetches it. That's a deliberately simple primitive — the interesting bit is *where* you embed it. A few categories that work well today:

## Defensive — "did someone touch the thing they shouldn't?"

- **Honey files on shared drives.** `mantis new "AWS-creds.txt" --pdf decoy.pdf` (or `--docx`, `--xlsx`, `--folder` for a 9-file bundle). Drop them in Finder/Explorer folders alongside real work. If a colleague's curiosity gets the better of them, or if an attacker mounts the drive after stealing creds, you find out the moment the file opens.
- **Fake credentials in `.env` / `.zshrc` / browser bookmarks.** A URL in a "production database URL" comment is irresistible to whoever has shell access.
- **SSH-login alarms on jump boxes.** `mantis new "ssh on prod-bastion" --install shell --ssh-only --out ~/.zshrc.d/mantis.sh`. Discord/Slack pings on every SSH login, with `$USER`, hostname, and the client IP captured. Quiet on local tmux panes thanks to `--ssh-only`.
- **Sudo-watch.** `--install shell-sudo` wraps `sudo` so every privileged invocation in your shell sends a notification with the full command.

## Detective — "where did my content end up?"

- **Track visits to a specific URL on your site.** Drop a 1×1 GIF mantis (`--svg` or just the bare trigger URL) in an obscure page, an email signature, or a forum signature. Get notified when traffic crosses a threshold — useful for honeypages, archive-bot detection, or just knowing when someone visits a private link you shared.
- **See if someone reads your email.** `mantis new "email beacon" --eml beacon.eml` produces a `.eml` file whose embedded reference fires when the mail client renders the message. Attach it (or open one yourself to test) and you'll know when it's been opened, by which IP / user-agent, and when. Same idea with `--ics` for "did they actually open the calendar invite" or `--vcf` for a contact card.
- **Detect site clones.** `mantis new "brand homepage clone-detector" --install js-clone-detector --hostname app.example.com` — the script fires only when it runs on a hostname that isn't yours, capturing `location.href` and `document.referrer` so you can see *where* your site was copied to (phishing kits, scrapers).
- **Find which document an insider exfiltrated.** Generate per-employee `.docx` / `.xlsx` versions of the same file via `mantis bulk-create --csv staff.csv --memo-template "{{name}} - Q4 strategy"`. If one of them leaks, the memo tells you who.

## Operational — "tell me when X happens in my house / lab / fleet"

- **Home Assistant + Scrypted hooks.** `mantis install <id> --type homeassistant` drops a `rest_command` block plus example automations into your HA config. Wire it to "front door opened after 11 PM", "garage camera came online", "unfamiliar device joined the LAN" — anything HA already knows about. Get the alert in the same Discord/Slack channel as your security stuff. The IoT helper at [`iot-helper/`](../iot-helper/README.md) adds an opt-in LAN watcher for hosts that aren't on HA.
- **Unattended host wake-up / network changes.** `--install macos-wake` / `--install linux-wake` / `--install windows-wake` fires when the machine resumes from sleep. `*-network` types fire on Wi-Fi join / Ethernet plug — useful for "did my Raspberry Pi just appear on a strange network?"
- **NFC tags at physical chokepoints.** `--install nfc-ndef` produces a URL you write to a blank NTAG sticker. Tape one to the back of a network closet door, your laptop lid, or a server rack. Anyone who taps with a phone fires it.
- **CI / monitoring backstops.** Use the URL as a heartbeat the opposite way: configure your CI / cron to NOT fire it on success, and have Uptime Kuma watch the matching status endpoint. Silence = healthy. The `mantis monitor <id> --mode latch` flag turns this on.

## Adversarial-research — "spread the URL on purpose"

- **Public paste/scraper bait.** Push tokens shaped like AWS keys, GitHub PATs, or `.env` files into public Pastebins / GitHub gists with the URL embedded as a "real" endpoint. Useful for measuring how fast credential-scraping bots in your industry actually act. (For the stateless variant where you really don't want a server-side hit log, `mantis edge mint` is appropriate.)
- **Tarpit your phishing kit.** When you find a copy of your site running on `evil-bank-login.tk`, hand them a mantis-watermarked logo so every visitor fires a hit and you can correlate to the kit's victim list.

## Stateless variant via `mantis-edge`

When you want sub-50ms Cloudflare-edge response, no DB to host, and "anyone with the AES key can mint" semantics (CI service accounts, hand-off URLs that should not appear in a central audit log), use `mantis edge mint --install <type>` instead of `mantis new --install <type>`. The host snippets are identical; only the URL shape and where the audit lives differ. See [`mantis-edge/README.md`](../mantis-edge/README.md) for the full edge-vs-stateful trade-off.
