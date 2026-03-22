# refine

A Claude Code plugin. Type `/refine` at the end of a session and it reviews what happened — which skills were used, what corrections you made, what patterns emerged. Then it creates or improves skills based on that.

Skills get better as you use them. That's the whole idea.

## What it does

1. Scans the session for skills that were invoked and how they performed
2. Reads those skill files to see what could be improved
3. Creates new skills for repeated workflows, or refines existing ones
4. Can also append to CLAUDE.md when the pattern is a general preference rather than a specific workflow
5. Commits changes with git

## What it won't do

- Touch plugin skills (`plugin:skill` format) — plugin updates would overwrite your changes
- Touch skills installed via `npx skills add` — same reason
- Force changes when nothing is worth changing
- Create skills for one-off tasks

## Where skills live

- `~/.refined/` — git repo for user-level refined skills, symlinked into `~/.claude/skills/`
- `.claude/skills/` — project-level skills, committed in the project repo
- When creating a new skill, it asks which scope

## Install

```bash
# Add as a local marketplace source
# (assumes the repo is cloned/symlinked into ~/.claude/plugins/refine)
claude plugin marketplace update local-plugins
claude plugin install refine@local-plugins
```

Or from GitHub:

```bash
claude plugin marketplace add lucharo/refine
claude plugin install refine@refine
```

## Use

```
/refine
```

That's it. Run it as your last command before you leave a session.

## Compatibility

Refined skills are standard SKILL.md files. They work with:
- Claude Code (via `~/.claude/skills/` symlinks)
- Any agent that reads SKILL.md (Cursor, Cline, etc. via `.agents/skills/`)
- `npx skills add ~/.refined` (vercel-labs/skills CLI)

## Limitations

Plugin skills use a `plugin:skill` namespace. User skills can't use colons. So there's no clean way to create a personalized version of a plugin skill — the names would differ and the agent wouldn't know which to pick. This needs platform-level support for skill overrides.
