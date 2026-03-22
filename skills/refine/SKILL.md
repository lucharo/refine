---
name: refine
description: Reflect on the current session and refine skills based on patterns observed. Use at the end of a session to make skills better through use.
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
---

# Refine

Review this session. Identify patterns worth capturing as new skills or improvements to existing skills.

## Step 1: Scan the session

Find which skills were used in this session by checking the conversation for `<command-name>` tags:

```bash
grep -o '<command-name>[^<]*</command-name>' "$TRANSCRIPT_PATH" | sed 's/<[^>]*>//g' | sort -u
```

If `$TRANSCRIPT_PATH` is not available, look at the conversation context directly — you have the full session in your context window already. Note which skills were invoked and how they performed.

## Step 2: Read those skills

For each skill used in the session, read its SKILL.md:
- Check `~/.claude/skills/` for user skills (these are yours to refine)
- Check `~/.claude/plugins/` for plugin skills (read-only — see design note below)

Also scan the session for patterns that no existing skill covers.

## Step 3: Decide

For each user skill used:
- Did it work well as-is? → leave it alone
- Did the user correct something, or did the skill miss something? → refine it

For the session overall:
- Was there a repeated workflow or hard-won knowledge worth codifying? → new skill
- Was there a correction the user made that indicates a preference? → new skill or update existing

If nothing is worth changing, say so and stop. Don't force changes.

## Step 4: Edit

**Refining an existing skill**: edit it where it already lives. Don't move skills between scopes.
- If it's in `~/.refined/` → edit in place
- If it's in `~/.claude/skills/` → edit in place
- If it's in `.claude/skills/` (project-level) → edit in place

**Creating a new skill**: ask the user which scope before writing:
- **User skill** — useful across all projects (e.g. "how I like commit messages", "my PR review style"). Write to `~/.refined/<name>/SKILL.md` in this repo and symlink to `~/.claude/skills/<name>`.
- **Local/repo skill** — specific to the current project (e.g. "how to deploy this app", "this repo's test patterns"). Write directly to `.claude/skills/<name>/SKILL.md` in the project's repo.

Keep changes minimal and focused.

## Why refine doesn't touch plugin skills

Plugin skills (invoked as `plugin:skill`) are namespaced and versioned by their
plugin. Refine doesn't personalise them because:

1. **Editing in place breaks on update** — the next plugin update overwrites your changes.
2. **Copying creates ambiguity** — user skill names can't contain colons, so a copy of
   `roborev:fix` would need a different name (e.g. `roborev-fix`). Now you have two
   similar skills and the agent doesn't know which to pick.

This is a limitation worth solving. The ideal future: everyone starts from base plugin
skills that get better for their personal use over time. That needs Claude Code to
support user-level skill overrides within a plugin namespace — it doesn't today.

For now, if a plugin skill needs improving: contribute upstream or fork the plugin.

## Step 5: Link, track, and commit

For each new or modified skill, ask the user:
- **Git track this skill?** (default: yes for user skills, no for local/repo skills since the repo already has its own git)

**User skills** (written to `~/.refined/`):
```bash
# Symlink into ~/.claude/skills/
ln -sf ~/.refined/<name> ~/.claude/skills/<name>

# If git-tracked: commit in ~/.refined
cd ~/.refined
git add <name>/
git commit -m "refine: <what changed and why>"
```

**Local/repo skills** (written to `.claude/skills/` in the project):
```bash
# Already in the project repo — commit there if the user wants
cd <project-root>
git add .claude/skills/<name>/
git commit -m "refine: <what changed and why>"
```

**Existing skills** (edited in place): if the skill is already git-tracked, commit in whichever repo it belongs to.

## Skill file format

```yaml
---
name: skill-name
description: >
  What this skill does. Be specific — this description decides when
  the skill triggers, so vague descriptions mean it never gets used.
allowed-tools:
  - Read
  - Edit
  # only what the skill actually needs
---
```

The body is the prompt Claude receives when the skill is invoked. Write it as direct instructions.

## Constraints

- Max 2 skills touched per refine. Max 1 new skill per refine.
- A skill should be under 200 lines. Longer means it's doing too much.
- Prefer refining an existing skill over creating a new one.
- Never edit plugin skill files directly (they get overwritten on update).
- Don't create skills for one-off tasks that won't recur.
- Don't capture things obvious from reading code or CLAUDE.md.

## Output

One-line summary of what you did (or "nothing to refine").
