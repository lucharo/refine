# refine — repo conventions

## Releasing a user-facing change
Skill / hook / command / README / CHANGELOG changes ship as a version bump, in one commit:
- Bump `version` in `.claude-plugin/plugin.json`
- Add a `## X.Y.Z` entry to `CHANGELOG.md`
- Commit as `refine X.Y.Z: <summary>` (repo style — not generic `feat:` / `docs:` prefixes)
- Patch = docs/guidance tweak; minor = new behavior

Repo-internal changes (this file, dev tooling) don't bump the version.
