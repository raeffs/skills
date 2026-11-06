---
name: fgj
description: Use fgj to work with Forgejo and Codeberg repositories from the terminal - listing, viewing, creating, and merging pull requests, reading and commenting on issues, and inspecting releases or Actions runs. Trigger whenever the user is working in a git repository whose remote points at a Forgejo instance (e.g. code.raeffs.dev, codeberg.org) and asks about pull requests, issues, reviews, merges, releases, or CI runs - even if they don't name fgj explicitly. Prefer fgj over gh for these hosts; gh only understands GitHub.
---

# fgj — Forgejo/Codeberg CLI

`fgj` is "`gh` for Forgejo." Use it inside a cloned Forgejo repo to inspect and manage PRs, issues, releases, and Actions without leaving the shell.

## When to use this skill

When the user is working in a git repo whose `origin` (or other remote) points at a Forgejo instance and asks to:

- review, open, merge, or comment on a pull request
- read, triage, comment on, or close an issue
- inspect releases or CI runs
- script any of the above via JSON output

If the remote is `github.com`, use `gh` instead. `fgj` does not understand GitHub.

**Quick check for the remote host:**

```bash
git remote get-url origin
```

If the host is `code.raeffs.dev`, or any other Forgejo/Gitea instance, `fgj` is the right tool.

## Repository auto-detection

Inside a cloned Forgejo repo, `fgj` reads the remote and infers both the hostname and the `owner/repo` slug. **Do not pass `-R` when the working directory already resolves to the right repo** — it is noise. Only pass `-R owner/repo` when:

- operating on a different repo than the current working directory, or
- running outside any git repo

## Authentication

The user authenticates once per host with `fgj auth login` (prompts for hostname and a personal access token). Tokens live in `~/.config/fgj/config.yaml`; the env vars `FGJ_HOST` and `FGJ_TOKEN` override at runtime.

**Do not run `fgj auth status` preemptively.** It is noise when things are already working. Only reach for it after a real `fgj` command fails with an auth-looking error ("401", "403", "not authenticated", "no token for host"). When it does fail, surface that to the user and stop — do not try to paper over it.

## Pull requests

```bash
fgj pr list                                  # open PRs in current repo
fgj pr list --state closed                   # or: open, closed, all
fgj pr list --json                           # machine-readable
fgj pr view 123                              # details of PR 123
fgj pr view 123 --json                       # same, as JSON
fgj pr create -t "Title" -b "Body" -H feature-branch -B main
fgj pr merge 123 --merge-method rebase       # rebase, then fast-forward (preferred)
fgj pr merge 123 --merge-method merge        # merge commit
fgj pr merge 123 --merge-method squash       # squash commits
```

When the user asks "review PR 42" or similar, start with `fgj pr view 42 --json` so the data is structured, then summarize for them.

### Merge methods

`fgj pr merge` only plumbs through three of Forgejo's merge styles: `merge`, `rebase`, and `squash`. Default to **`rebase`** — it rebases the branch onto the base and then fast-forwards, which is this user's preferred style (no spurious merge commits). Use `merge` or `squash` only when explicitly requested.

**Not currently supported by `fgj`:**

- **Pure fast-forward-only** (`fast-forward-only` in Forgejo). If the user asks specifically for a fast-forward-only merge (no rebase), tell them `fgj` can't do it today — they can either use `--merge-method rebase` (same end state when the branch is already on top of base) or use the web UI.
- **Auto-merge** (merge-when-checks-pass). `fgj` has no `--auto` flag. If the user asks for auto-merge, say so and point them at the web UI. Do not attempt to approximate it by watching CI and merging manually unless they explicitly ask you to.

## Issues

```bash
fgj issue list                               # open issues
fgj issue list --state all
fgj issue view 456
fgj issue view 456 --json
fgj issue create -t "Title" -b "Body"
fgj issue create -t "Title" -b "Body" -l bug -l enhancement
fgj issue comment 456 -b "My comment"
fgj issue close 456
fgj issue close 456 -c "Fixed in v2.0"       # close with a comment
fgj issue edit 456 -t "New Title"
fgj issue edit 456 --add-label priority --remove-label bug
```

## Releases

```bash
fgj release list
fgj release view v1.2.3
fgj release create v1.2.3 -t "v1.2.3" -n "Notes" ./dist/app.tar.gz
fgj release upload v1.2.3 ./dist/app.tar.gz --clobber
fgj release delete v1.2.3                    # tag is preserved
```

## Actions (Forgejo CI)

```bash
fgj actions workflow list
fgj actions workflow view ci.yml
fgj actions workflow run deploy.yml -f environment=production -r main
fgj actions run list
fgj actions run view 123 --log               # full logs
fgj actions run view 123 --job 456 --log     # single-job logs
fgj actions run watch 123                    # follow to completion
fgj actions run rerun 123
fgj actions run cancel 123
```

Secrets and variables are managed under `fgj actions secret ...` and `fgj actions variable ...` — run `fgj actions --help` for the exact subcommands when needed.

## Repositories

```bash
fgj repo view owner/repo
fgj repo list                                # user's own repos
fgj repo create my-repo -d "desc" --private --add-readme -g Go -l MIT
fgj repo clone owner/repo -p ssh             # or: https
fgj repo fork owner/repo
```

## Global flags and JSON

- `-R owner/repo` — override the auto-detected repo (see above: usually omit)
- `--hostname <host>` — target a specific Forgejo instance explicitly
- `--config <path>` — alternative config file
- `--json` — supported on most `list` and `view` commands. Prefer it whenever you need to parse output; avoid piping human-readable tables through `grep`/`awk`.

### Filtering: server-side is limited, filter client-side

`fgj` exposes **very few filter flags**. The only server-side filter on `list` commands is usually `--state` (`open` / `closed` / `all`). There is **no** `--label`, `--author`, `--assignee`, or similar filter on `pr list` or `issue list` — do not invent one. For anything beyond state, pull the full `--json` output and filter client-side.

**Example — "open issues tagged bug":**

```bash
fgj issue list --json | jq '[.[] | select(.labels[]?.name == "bug")]'
```

**Example — "open PRs by me":**

```bash
fgj pr list --json | jq '[.[] | select(.user.login == "raf")]'
```

If the user asks for a filter you are unsure is supported, either reach for `jq` over `--json` (safe) or confirm with `fgj <group> <subcommand> --help` (also safe) — never guess a flag name.

## Working style

1. **Verify the host first** (`git remote get-url origin`) before running `fgj` commands — if the remote is not Forgejo, stop and suggest `gh` or another tool.
2. **Let `fgj` auto-detect the repo.** No `-R` inside a clone.
3. **Reach for `--json` whenever you need to reason about the output**, then summarize for the user. Human tables are for humans.
4. **Do not invent subcommands or flags.** If something isn't documented here, run `fgj <group> <subcommand> --help` to confirm before executing. Hallucinating a plausible-looking flag (e.g. `--label`, `--author`) is a common failure mode — when in doubt, pull `--json` and filter client-side.
5. **Surface auth failures honestly.** If a command fails with an auth error, tell the user to run `fgj auth login` for the relevant host — do not try to work around it.
