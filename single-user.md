# Single-user by default

Mantis is single-user out of the box. The first API key minted on a fresh
deploy is automatically `is_admin = true` (the bootstrap path sets the flag).
That key is the operator: it sees and manages every key on the instance.
Additional API keys created later default to **non-admin** — they can only
see and manage the keys they created themselves.

If you only ever mint one API key for your deploy you'll never notice the
distinction; the schema is shaped that way so it's easy to later promote a
shared instance to multi-user. Until then, treat every operator as admin and
keep your bootstrap key safe. The `audit log` (`mantis audit log`, admin-only)
records create / update / delete / login events across the instance.
