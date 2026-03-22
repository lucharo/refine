---
name: refine
description: >
  Reflect on the current session and refine skills or CLAUDE.md based on patterns
  observed. Use when the user says "refine", "what did we learn", or at the end of
  a session to make skills better through use.
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
  - AskUserQuestion
---

# Refine

Review this session. Identify patterns worth capturing as new skills, improvements to existing skills, or additions to CLAUDE.md.

## Step 1: Scan the session

Look through the conversation for `<command-name>` tags — these appear when skills are invoked. Note which skills were used and how they performed.

Also note:
- Corrections the user made (these reveal preferences worth codifying)
- Repeated workflows that could become skills
- Hard-won knowledge (debugging, research) worth preserving
- General behavioral preferences that belong in CLAUDE.md rather than a skill

## Step 2: Read those skills

For each skill used in the session, find and read its SKILL.md. Check these locations:
- `~/.claude/skills/` — user skills
- `.claude/skills/` — project-level skills
- `.agents/skills/` — universal skills (installed via `npx skills add`, cross-agent)
- `~/.claude/plugins/` — plugin skills (read-only)

For each, `ls -la` to check if it's a symlink and where it lives:
- Symlink → `~/.refined/`: yours to refine
- Symlink → a plugin dir: read-only (updates would overwrite)
- Regular file in `~/.claude/skills/` or `.claude/skills/`: yours to refine
- Anything in `.agents/skills/`: read-only (managed by `npx skills`, updates would overwrite)

Also scan the session for patterns that no existing skill covers.

## Step 3: Decide

Ask yourself three questions:

**Skills** — for each user skill used:
- Did it work well as-is? → leave it alone
- Did the user correct something, or did the skill miss something? → refine it

**New skills** — for the session overall:
- Was there a repeated workflow or hard-won knowledge worth codifying? → new skill
- Was there a correction that indicates a process worth capturing? → new skill

**CLAUDE.md** — for general preferences and behavioral instructions:
- Did the user correct a general behavior (not tied to a specific workflow)? → CLAUDE.md
- Is there a tool preference, communication style, or project convention worth persisting? → CLAUDE.md
- Examples: "always use uv", "don't ask before committing", "use rip instead of rm"

If nothing is worth changing, say so and stop. Don't force changes.

**Constraints applied here**: max 2 skills touched per refine, max 1 new skill per refine.

## Step 4: Edit

### Skills

**Refining an existing skill**: first `ls -la` to check if it's a symlink.
- Symlink to `~/.refined/` → edit the target file in `~/.refined/`
- Symlink to a plugin dir or `.agents/skills/` → do NOT edit (read-only — managed externally)
- Regular file in `~/.claude/skills/` or `.claude/skills/` → edit in place

**Creating a new skill**: ask the user which scope:
- **User skill** — useful across all projects. Write to `~/.refined/<name>/SKILL.md` and symlink to `~/.claude/skills/<name>`.
- **Local/repo skill** — specific to the current project. Write to `.claude/skills/<name>/SKILL.md` in the project's repo.

### CLAUDE.md

**Always ask before editing CLAUDE.md.** Show the proposed addition and ask the user to confirm.

- Only append — never modify or remove existing entries
- Keep additions concise (1-3 lines per entry)
- Choose the right file:
  - `~/.claude/CLAUDE.md` — global preferences (affect all projects)
  - `.claude/CLAUDE.md` or `CLAUDE.md` in project root — project-specific

## Why refine doesn't touch externally managed skills

Two categories of skills are read-only:

**Plugin skills** (invoked as `plugin:skill`) — namespaced and versioned by their plugin.
- Editing in place breaks on update.
- Copying creates ambiguity — user skill names can't contain colons, so a copy of `roborev:fix` would need a different name. Two similar skills, agent doesn't know which to pick.

**Skills installed via `npx skills add`** — managed by the [vercel-labs/skills](https://github.com/vercel-labs/skills) CLI, typically symlinked from `.agents/skills/`.
- `npx skills update` would overwrite your changes.
- These are designed to be shared across agents (Claude, Cursor, Cline, etc.).

The ideal future: everyone starts from base skills that get better for their personal use over time. That needs skill override support at the platform level.

For now, if an external skill needs improving: contribute upstream or fork.

Note: `~/.refined/` is itself a valid source for `npx skills add ~/.refined` — so refined skills can be shared with other agents or users.

## Step 5: Link, track, and commit

For each new or modified skill, ask the user whether to git track it (default: yes for user skills, no for local/repo skills).

**User skills** (written to `~/.refined/`):
```bash
# Symlink into ~/.claude/skills/ (-n flag handles directory symlinks correctly)
ln -sfn "$HOME/.refined/<name>" "$HOME/.claude/skills/<name>"

# Optional: also link into .agents/skills/ for cross-agent compatibility (Cursor, Cline, etc.)
# Only if the user uses npx skills / multi-agent setup
# mkdir -p .agents/skills && ln -sfn "$HOME/.refined/<name>" .agents/skills/<name>

# If git-tracked:
git -C "$HOME/.refined" add <name>/ && git -C "$HOME/.refined" commit -m "refine: <what changed and why>"
```

**Local/repo skills** (written to `.claude/skills/` in the project):
```bash
# Commit in the project repo if the user wants
git -C <project-root> add .claude/skills/<name>/ && git -C <project-root> commit -m "refine: <what changed and why>"
```

**Existing skills** (edited in place): commit in whichever repo the file belongs to.

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
- Never edit externally managed skills (plugins, `npx skills` installs) — they get overwritten on update.
- Always check symlink targets before editing — don't follow symlinks into plugin dirs or `.agents/skills/`.
- Don't create skills for one-off tasks that won't recur.
- Don't capture things obvious from reading code or CLAUDE.md.
- Always ask before editing CLAUDE.md. Only append, never modify existing entries.

## Output

One-line summary of what you did (or "nothing to refine").
