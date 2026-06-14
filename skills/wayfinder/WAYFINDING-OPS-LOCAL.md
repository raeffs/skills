# Wayfinding ops — Local markdown files

*How the map, child tickets, blocking, frontier, claim, and resolve operations physically live when there's no issue tracker. See [WAYFINDING-OPS.md](./WAYFINDING-OPS.md) for tracker detection, and `wayfinder/SKILL.md` for what each operation means.*

Map = a file; children = one file per ticket. No tracker, no API — just files under `.scratch/`.

- **Map**: `.scratch/<effort>/map.md` — the Destination / Notes / Decisions-so-far / Not-yet-specified / Out-of-scope body.
- **Child ticket**: `.scratch/<effort>/issues/NN-<slug>.md`, numbered from `01`, question in the body. A `Type:` line records the type (`research`/`prototype`/`grilling`/`task`); a `Status:` line records `claimed`/`resolved` (absent = open, unclaimed).
- **Blocking**: a `Blocked by: NN, NN` line near the top; unblocked when every listed file is `resolved`.
- **Frontier**: scan `.scratch/<effort>/issues/` for open, unblocked, unclaimed files; lowest number wins.
- **Claim**: set `Status: claimed` and save before any work.
- **Resolve**: append the answer under an `## Answer` heading, set `Status: resolved`, then append the context pointer to the map's Decisions-so-far in `map.md`.
