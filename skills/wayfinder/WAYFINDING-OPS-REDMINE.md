# Wayfinding ops: Redmine (`redmine-cli`)

*How the map, child tickets, blocking, frontier, claim, and resolve operations physically live on Redmine. See [WAYFINDING-OPS.md](./WAYFINDING-OPS.md) for tracker detection, and `wayfinder/SKILL.md` for what each operation means.*

Map = a `Wayfinder Map` issue; children = per-type `Wayfinder …` issues, native subtasks of it. Everything runs through the `redmine-cli` skill; subtasks, relations, assignment, and workflow status are all native, so nothing rides in body conventions.

**One-time setup (web UI; the REST API can't create trackers or workflows).** Per instance: create trackers `Wayfinder Map`, `Wayfinder Research`, `Wayfinder Prototype`, `Wayfinder Grilling`, `Wayfinder Task`; copy a workflow onto each permitting `New → In Progress → Closed` for the agent's role; enable all five on the project(s). Record the project identifier in the map's `## Notes`.

**Every `issues list` needs `--project <proj>`.** Omit it and the CLI falls back to its default project, so `--parent <map>` silently returns nothing.

- **Map**: `redmine issues create --project <proj> --tracker "Wayfinder Map" --subject "<title>" --description "<body>"`. Edit the body with `redmine issues update <map> --description "<body>"` (fetch via `redmine issues get <map> -o json`, append, write back). List maps: `redmine issues list --project <proj> --tracker "Wayfinder Map" --status '*'`.
- **Child ticket**: `redmine issues create --project <proj> --tracker "Wayfinder <Type>" --parent <map> --subject "<title>" --description "## Question\n<…>"`. `<Type>` ∈ {Research, Prototype, Grilling, Task}; `--parent` makes it a native subtask.
- **Blocking**: `redmine issues relations add <blocker> --to <blocked> --type blocks`, wired in a second pass (blockers need ids first).
- **Claim**: `redmine issues update <n> --assignee me --status "In Progress"`, which assigns and moves to In Progress in one call (the `assign` subcommand sets only the assignee).
- **Frontier**: `redmine issues list --project <proj> --parent <map> --status open --relations -o json`; drop the assigned (claimed) and any blocked by a still-open issue. Narrow to a type with `--tracker "Wayfinder <Type>"`.
- **Resolve**: `redmine issues update <n> --status "Closed" --note "<answer>"` (not `issues close`, which picks the first closed status). Then append the context pointer to the map's Decisions-so-far via the map-body edit above.

**Silent-drop caveat.** A status change to a tracker with no workflow (or a transition the role lacks) is dropped but still returns `OK: Updated issue`. A ticket stuck at `New` after a claim = the workflow wasn't copied.
