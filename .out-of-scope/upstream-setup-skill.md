# The upstream setup skill

`setup-matt-pocock-skills` is not adopted. It configures a repo for Matt's engineering skills: the issue tracker, the triage label vocabulary, and the domain doc layout. Requests to adopt it, or to add a setup step that the adopted skills depend on, are out of scope.

## Why this is out of scope

The adopted skills that upstream makes depend on this setup do not depend on it here. `to-spec`, `to-tickets` and `wayfinder` detect the tracker from the git remote, or from an explicit instruction in `CLAUDE.md`, and pick the matching CLI. Each UPSTREAM.yaml records that deviation. With the tracker detected inline, the setup skill has no job left.

The rest of what it configures, the triage labels and the domain doc layout, serves skills that are not adopted. If one of them is adopted later, its configuration goes inline in the same way.

## Upstream skills covered

- https://github.com/mattpocock/skills at `skills/engineering/setup-matt-pocock-skills`
