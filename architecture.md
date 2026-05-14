# Project layout

```
src/                      # Next.js server (API + dashboard + public trigger)
  app/
    layout.tsx            # root layout, imports globals.css
    page.tsx              # / redirects to /keys or /login
    globals.css           # Tailwind 4 entry
    login/                # paste-key form + server action
    logout/               # POST clears session cookie
    (app)/                # route group: auth-required pages share a nav layout
      keys/
        page.tsx          # list + disable/enable
        new/              # create form
        [id]/             # detail + hits + copy/delete
    api/keys/...        # authenticated CRUD
    api/api-keys/...      # authenticated key mgmt
    api/inbox/...         # dev inbox JSON
    c/[publicId]/         # the public trigger
    inbox/                # dev webhook capture + viewer
  db/                     # Drizzle schema + migrations
  lib/                    # auth, env, keys, notify, response, session, etc.
  instrumentation.ts      # boot hook: migrations + bootstrap
cli/                      # @mantis/cli — terminal client
  src/
    commands/             # one file per command
    lib/                  # api client, config (keychain), output
docker/
docker-compose.yml
drizzle.config.ts
postcss.config.mjs
```
