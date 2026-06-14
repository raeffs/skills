# Skills

## Development Workflow

- Sourcecode lives on [Forgejo][forgejo]
- Sourcecode is mirrored to [GitHub][github]

[forgejo]: https://code.raeffs.dev/raeffs-dot-dev/skills
[github]: https://github.com/raeffs/skills

## Skill Adoption

If a skill is adopted, it contains a sidecar `skills/<name>/UPSTREAM.yaml`:

```yaml
repo: <the clone URL>
path: <upstream dir, as it was at ref>
ref: <full 40-char SHA, never a tag or branch>
adopted: <date content last came in from upstream>
license: <license of the upstream skill>
deviations: <optional description of what changed>
```
