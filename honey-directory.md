# Honey directory

A `.zip` bundle of pre-baited files, all wired to the same key. Drop the extracted folder on a shared drive; any file inside that gets opened, double-clicked, or `cat`'d fires the mantis.

```bash
mantis new "Q4 Leadership Plans" \
  -w http://localhost:3000/inbox/folder \
  --folder ./honey.zip

# Or from the dashboard: key detail page → "honey directory (zip)" → ↓ folder.zip
# Or API: GET /api/keys/<id>/download?format=folder
```

The unzipped folder contains 9 bait files, each independently triggering the same key:

| File | Trigger when |
|---|---|
| `Q4 Salary Review.xlsx` | Opened in Excel / LibreOffice Calc |
| `Restructuring Memo - Draft.docx` | Opened in Word / LibreOffice Writer |
| `All-Hands Q4 Plans.pptx` | Opened in PowerPoint / LibreOffice Impress |
| `Layoff Schedule 2026.pdf` | Opened in Adobe Reader (`/OpenAction`) or clicked link |
| `passwords.txt` | Cat'd / read — has fake credentials + URL line at the bottom |
| `database-credentials.txt` | Same pattern — fake DB creds + URL |
| `README.txt` | Reading the directory's "what is this" file |
| `Open in Browser.url` | Double-clicked on Windows (Internet Shortcut) |
| `Latest Version.webloc` | Double-clicked on macOS (URL bookmark) |

Note: this does **not** fire automatically when someone browses the folder in Finder/Explorer — that requires DNS infrastructure (a future stage). What it *does* give you is high-surface honeypot detection: a curious user spelunking the directory will open at least one of those files, and any of them triggers the alert.

Fake credentials in `passwords.txt` / `database-credentials.txt` use AWS's and Stripe's documented example keys (`AKIAIOSFODNN7EXAMPLE`, `sk_live_4eC39HqLyjWDarjtT1zdp7dc`) — publicly known fakes, not real credentials anywhere.
