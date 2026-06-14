# Wayfinding operations

How the map, its child tickets, blocking, frontier, claim, and resolve operations physically live, per tracker. `wayfinder/SKILL.md` describes *what* each operation means; the per-tracker files below are *how* to perform it on the tracker this repo uses.

Detect the tracker in precedence order — an explicit project-config/Notes instruction always wins over anything inferred from the git remote — then open that tracker's file and follow it:

- **Explicit tracker instruction** — a project-config line (e.g. in `CLAUDE.md`) or the map's `## Notes` naming where issues live, such as `Issues live on Redmine: https://<host>/projects/<identifier>`. Use the backend it names; today that's Redmine → **[WAYFINDING-OPS-REDMINE.md](./WAYFINDING-OPS-REDMINE.md)** (`redmine-cli`, scoped to the `<identifier>` in the URL).
- **Forgejo / Codeberg git remote** (e.g. `code.raeffs.dev`, `codeberg.org`) → **[WAYFINDING-OPS-FORGEJO.md](./WAYFINDING-OPS-FORGEJO.md)** (`fgj`).
- **No remote issue tracker** (or you can't determine one) → **[WAYFINDING-OPS-LOCAL.md](./WAYFINDING-OPS-LOCAL.md)** (local markdown files under `.scratch/`).
