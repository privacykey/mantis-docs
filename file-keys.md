---
title: "File keys"
description: "Generate bait files that fire a Mantis trigger when opened or rendered."
---

Mantis can generate 13 server-backed artifacts. Most embed the key URL so the
trigger fires when a viewer renders the file; NFC and Wallet artifacts fire
through the platform action they are designed for.

| Format | Mechanism | Best in | Notes |
|---|---|---|---|
| `.docx` | External-image relationship in OOXML | Word, LibreOffice Writer | Most reliable — fires on render, also from email attachments after "Enable Editing" |
| `.xlsx` | Same external-image trick, attached to a worksheet drawing | Excel, LibreOffice Calc | Identical reliability to DOCX |
| `.pptx` | Same external-image trick on slide 1 | PowerPoint, Keynote (some), LibreOffice Impress | Same as above |
| `.pdf` | **Combo**: `/OpenAction → /URI` + clickable `/Link` annotation | Adobe Reader, Foxit, most enterprise PDF readers | Less reliable — Chrome's PDFium viewer doesn't fire OpenAction; macOS Preview doesn't either. The clickable link covers the "user reads + clicks" case in those viewers. |
| `.zip` (`folder`) | Honey-directory bundle of 9 bait files | Shared drives, unpacked project folders | Each file in the extracted folder triggers the same key |
| `.pdf` (`nfc-label`) | Printable QR/NFC sticker label | Physical tags, asset labels | The PDF does not fire by itself; the scan/tap opens the key URL |
| `.pkpass` (`apple-wallet`) | Signed Apple Wallet pass with web-service callbacks | iPhone Wallet | Requires Wallet config; install, uninstall, and fetch callbacks record hits |
| `.svg` | `<image href>` referencing the trigger URL | Browsers, some image viewers, photo libraries | Apps that raster-thumbnail may not fire on preview — opening the original always does |
| `.html` | `<img src>` in a standalone page | Any browser, web-clip notes | Fires on first render |
| `.md` | `![](URL)` image syntax | Joplin / Trilium / Logseq / Gitea README | Fires when the renderer loads images |
| `.eml` | RFC 5322 message with HTML body `<img src>` | Thunderbird, Apple Mail, mail-archive previews | Fires on message render |
| `.ics` | iCalendar event with `ATTACH;FMTTYPE=image/png` + `URL` | Apple Calendar, some CalDAV clients | Behavior varies by client |
| `.vcf` | vCard 4.0 with `PHOTO;VALUE=URI` | macOS / iOS Contacts, CardDAV clients | Fires when the contact's avatar renders |

See **[self-hosted apps](/self-hosted-apps)** for per-app recipes (Immich, Paperless, Joplin, Vaultwarden, dashboards, code hosts, etc.).

```bash
# Create key + generate the formats supported directly by `mantis new`
mantis new "Q4 forecast" \
  -w http://localhost:3000/inbox/q4 \
  --docx ./forecast.docx \
  --xlsx ./forecast.xlsx \
  --pptx ./forecast.pptx \
  --pdf  ./forecast.pdf \
  --svg  ./forecast.svg \
  --html ./forecast.html \
  --md   ./forecast.md \
  --eml  ./forecast.eml \
  --ics  ./forecast.ics \
  --vcf  ./forecast.vcf \
  --folder ./forecast-bundle.zip \
  --qr ./forecast-qr.png

# Download artifacts for an existing key, including Wallet/NFC formats
mantis download <key-id> --docx ./out.docx
mantis download <key-id> --pdf ./out.pdf
mantis download <key-id> --nfc-label ./nfc-label.pdf
mantis download <key-id> --apple-wallet ./mantis.pkpass

# Dashboard: key detail page → "file keys" card has download links for all formats
# API: GET /api/keys/<id>/download?format=docx|xlsx|pptx|pdf|folder|nfc-label|apple-wallet|svg|html|md|eml|ics|vcf  (Bearer or session)
```

**Office reader caveats**:
- ✅ Microsoft Office desktop apps — fetch external image on render (subject to **Protected View** for files marked "from internet"; first "Enable Editing" click triggers).
- ✅ LibreOffice (Writer/Calc/Impress) — fetches external content by default.
- ⚠ Office on the web / Office 365 in browser — depends on tenant policy.
- ❌ macOS Quick Look — does not render external content.

**PDF reader caveats**:
- ✅ Adobe Acrobat Reader — follows `/OpenAction → /URI` (may show a one-time trust prompt for the host).
- ✅ Foxit Reader, PDF-XChange — typically follows OpenAction.
- ⚠ Chrome, Edge, Firefox built-in viewers — OpenAction not honored; mantis fires only if user clicks the visible "View the latest version online" link.
- ❌ macOS Preview — OpenAction not honored; click-the-link fallback works.

All generated files include placeholder body text (`CONFIDENTIAL DRAFT — do not distribute.`). Edit the file after download to make it look authentic.

Apple Wallet artifacts require an Apple Developer Pass Type ID certificate. Set
the `APPLE_PASS_*` env vars or configure `/settings/wallet` as an admin. If
Wallet is not configured, `format=apple-wallet` returns `503 not_configured`.
