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

Identify what happened:
1. Which skills were invoked (look for `<command-name>` tags) or created?
2. What corrections did the user make? These are the most valuable signals.
3. What workflows were repeated?
4. What preferences or conventions emerged?

## Step 2: Evaluate the skills

If skills were **created or edited this session**, they're already in your context — no need to re-read them. Evaluate what you already have.

If skills were **used but not edited**, read the actual SKILL.md files to see their current state. Check these locations:
```bash
ls -la ~/.claude/skills/ 2>/dev/null
ls -la .claude/skills/ 2>/dev/null
ls -la ~/.refined/ 2>/dev/null
```

For each skill, whether from context or from reading, evaluate:
- Is the description specific enough to trigger correctly?
- Are the instructions clear and complete based on how the skill was actually used?
- Did the session reveal gaps, edge cases, or missing steps?
- Are references to other files/skills correct?

Check ownership before editing:
- Symlink → `~/.refined/`: yours to refine
- Symlink → a plugin dir: read-only (updates would overwrite)
- Regular file in `~/.claude/skills/` or `.claude/skills/`: yours to refine
- Anything in `.agents/skills/`: read-only (managed by `npx skills`)

**Also read the project's CLAUDE.md** (and `~/.claude/CLAUDE.md` if relevant). Check whether what happened in this session — tool preferences, conventions, corrections — is reflected there or should be.

## Step 3: Decide

After evaluating skills and CLAUDE.md, ask three questions:

**Existing skills** — for each one you read:
- Does the description match how it was actually used? If not, fix it.
- Did the session expose missing instructions, wrong assumptions, or incomplete steps?
- Did the user correct something the skill should have known?

**New skills** — be proactive. If you identify a workflow worth capturing, create it. Don't just mention it as a possibility and move on.
- Was there a workflow that could bring value in future sessions? Create the skill. You don't need to see it repeated twice — once is enough if it's clearly reusable.
- Was there hard-won knowledge (debugging, research) worth preserving?
- Was there a multi-step process the agent performed that could be codified?

**CLAUDE.md** — for behavioral instructions, not workflows:
- Did the user correct a general behavior (not tied to a specific workflow)?
- Is there a tool preference, communication style, or project convention worth persisting?
- Examples: "always use uv", "don't ask before committing", "use rip instead of rm"
- **Prefer skills over CLAUDE.md** for anything that's a multi-step workflow. CLAUDE.md is for preferences and rules; skills are for procedures.

Choose the right file:
- **User CLAUDE.md** (`~/.claude/CLAUDE.md`): who the user is and how they work across all projects — tool preferences, communication style, secret handling, memory conventions
- **Repo CLAUDE.md** (`CLAUDE.md` or `.claude/CLAUDE.md`): how this specific codebase works — stack, commands, validation, deploy, commit conventions

If it would be useful in a different repo, it belongs in user CLAUDE.md. If it only makes sense for this project, it belongs in repo CLAUDE.md.

Your default stance should be to create or improve something. Most sessions contain at least one workflow, preference, or piece of knowledge worth capturing. "Nothing to refine" is valid but should be rare — it means you genuinely found no reusable workflow, no skill to improve, and no CLAUDE.md update needed.

**Constraints**: max 2 skills touched per refine, max 1 new skill per refine.

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

**Always ask before editing CLAUDE.md.** Show the proposed change and ask the user to confirm.

- Append by default. Updating existing entries is acceptable when they're stale or wrong — but always ask first.
- Keep additions concise (1-3 lines per entry)
- File choice is covered in Step 3 above (user vs repo CLAUDE.md)

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

## Memory files are out of scope

Some users have auto-memory systems (persistent file-based memory). Refine does NOT touch memory files. The persistence hierarchy is:
- **Skills** → reusable workflows and procedures
- **CLAUDE.md** → behavioral rules and project conventions
- **Memory files** → contextual facts, user profile, project state (managed by the memory system, not refine)

If something looks like a fact to remember rather than a rule to follow or a workflow to codify, it's not refine's job.

## Constraints

- Max 2 skills touched per refine. Max 1 new skill per refine.
- A skill should be under 200 lines. Longer means it's doing too much.
- Prefer refining an existing skill over creating a new one.
- Never edit externally managed skills (plugins, `npx skills` installs) — they get overwritten on update.
- Always check symlink targets before editing — don't follow symlinks into plugin dirs or `.agents/skills/`.
- Don't create skills for truly one-off tasks (e.g. "fix this specific bug"). But if a workflow could plausibly be useful again, create the skill — don't wait for proof of recurrence.
- Don't capture things obvious from reading code or CLAUDE.md.
- Always ask before editing CLAUDE.md. Only append, never modify existing entries.

## Output

One-line summary of what you did (or "nothing to refine").
