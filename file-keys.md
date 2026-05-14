# File keys

Ten file formats supported, each embedding the mantis URL so the file's "trigger" fires when the file is opened in the appropriate app:

| Format | Mechanism | Best in | Notes |
|---|---|---|---|
| `.docx` | External-image relationship in OOXML | Word, LibreOffice Writer | Most reliable — fires on render, also from email attachments after "Enable Editing" |
| `.xlsx` | Same external-image trick, attached to a worksheet drawing | Excel, LibreOffice Calc | Identical reliability to DOCX |
| `.pptx` | Same external-image trick on slide 1 | PowerPoint, Keynote (some), LibreOffice Impress | Same as above |
| `.pdf` | **Combo**: `/OpenAction → /URI` + clickable `/Link` annotation | Adobe Reader, Foxit, most enterprise PDF readers | Less reliable — Chrome's PDFium viewer doesn't fire OpenAction; macOS Preview doesn't either. The clickable link covers the "user reads + clicks" case in those viewers. |
| `.svg` | `<image href>` referencing the trigger URL | Browsers, some image viewers, photo libraries | Apps that raster-thumbnail may not fire on preview — opening the original always does |
| `.html` | `<img src>` in a standalone page | Any browser, web-clip notes | Fires on first render |
| `.md` | `![](URL)` image syntax | Joplin / Trilium / Logseq / Gitea README | Fires when the renderer loads images |
| `.eml` | RFC 5322 message with HTML body `<img src>` | Thunderbird, Apple Mail, mail-archive previews | Fires on message render |
| `.ics` | iCalendar event with `ATTACH;FMTTYPE=image/png` + `URL` | Apple Calendar, some CalDAV clients | Behavior varies by client |
| `.vcf` | vCard 4.0 with `PHOTO;VALUE=URI` | macOS / iOS Contacts, CardDAV clients | Fires when the contact's avatar renders |

See **[self-hosted-apps.md](./self-hosted-apps.md)** for per-app recipes (Immich, Paperless, Joplin, Vaultwarden, dashboards, code hosts, etc.).

```bash
# Create key + generate any/all formats in one step
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
  --vcf  ./forecast.vcf

# Or download a file for an existing key
mantis download <key-id> --docx ./out.docx
mantis download <key-id> --pdf ./out.pdf

# Dashboard: key detail page → "file keys" card has download links for all formats
# API: GET /api/keys/<id>/download?format=docx|xlsx|pptx|pdf|svg|html|md|eml|ics|vcf  (Bearer or session)
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
