# Wayfinding ops — Forgejo / Codeberg (`fgj`)

*How the map, child tickets, blocking, frontier, claim, and resolve operations physically live on Forgejo/Codeberg. See [WAYFINDING-OPS.md](./WAYFINDING-OPS.md) for tracker detection, and `wayfinder/SKILL.md` for what each operation means.*

Map = a single issue; children = ordinary issues pointing back at it. Everything runs through the `fgj` skill. `fgj` exposes no dependency, assignee, or sub-issue commands and no server-side label filter, so blocking and the map→child link are **body conventions**, claiming is a **label**, every frontier query pulls `--json` and filters client-side — and the frontier isn't rendered in Forgejo's UI; the map body is the source of truth.

**One-time setup.** `fgj` can't create labels and Forgejo rejects `issue create -l <label>` for a nonexistent label, so create these once in the web UI: `wayfinder:map`, `wayfinder:research`, `wayfinder:prototype`, `wayfinder:grilling`, `wayfinder:task`, `wayfinder:claimed`.

- **Map**: `fgj issue create -t "<title>" -b "<body>" -l wayfinder:map`. Edit the body later with `fgj issue edit <map> -b "<body>"`.
- **Child ticket**: `fgj issue create -t "<title>" -b "<body>" -l wayfinder:<type>`. No native sub-issues, so bind to the map with `Part of #<map>` on the body's **first line**; the `## Question` block follows.
- **Blocking**: no native dependencies — add a `Blocked by: #<n>, #<n>` line under the `Part of` line (`none` when nothing gates it), wired in a second pass with `fgj issue edit`. Unblocked when every listed issue is closed.
- **Claim**: `fgj issue edit <n> --add-label wayfinder:claimed`; release with `--remove-label wayfinder:claimed`.
- **Frontier**: `fgj issue list --state open --json`, then filter client-side — keep issues whose body first line is `Part of #<map>`, drop those labelled `wayfinder:claimed`, drop those whose `Blocked by:` still lists an open issue (check with `fgj issue view <n> --json`). Example:

  ```bash
  fgj issue list --state open --json \
    | jq --arg map "#<map>" '[.[] | select(.body | startswith("Part of " + $map))]'
  ```

- **Resolve**: `fgj issue close <n> -c "<answer>"` (comments and closes at once), then append the context pointer to the map's Decisions-so-far with `fgj issue edit <map> -b "<body>"`.
