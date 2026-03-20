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

## What to look for

- Approaches that worked well and would be useful again
- Corrections the user made that indicate a preference or practice worth codifying
- Workflows that were repeated and could be turned into a skill
- Hard-won knowledge (debugging, research, workarounds) that should be preserved
- Existing skills that were used but could be improved based on this session

## What NOT to do

- Don't create skills for one-off tasks that won't recur
- Don't capture things obvious from reading code or CLAUDE.md
- Don't create overly broad or vague skills
- Don't touch skills that weren't relevant to this session
- Max 2 skills touched per refine. Max 1 new skill per refine.

## Process

1. **Reflect** — what was the main work? What patterns emerged? What corrections were made?
2. **Read existing skills** — `ls ~/.claude/skills/` and read the ones relevant to this session
3. **Decide** — should an existing skill be refined, or a new one created? Or neither.
4. **Edit** — if yes, make the change. Keep it minimal and focused. Write skills to `managed/` in this repo.
5. **Symlink** — if you created a new skill, symlink it: `ln -sf $(pwd)/managed/<name> ~/.claude/skills/<name>`
6. **Commit** — `git commit` in this repo with a concise message describing what was refined and why.

## Skill file format

Skills live in `managed/<skill-name>/SKILL.md` with YAML frontmatter:

```yaml
---
name: skill-name
description: >
  What this skill does. Be specific — this description is used to decide
  when the skill is relevant, so vague descriptions mean it never triggers.
allowed-tools:
  - Read
  - Edit
  # only what the skill actually needs
---
```

The body is the prompt that Claude receives when the skill is invoked. Write it as direct instructions.

## Constraints

- A skill should be under 200 lines. If it's longer, it's doing too much.
- Prefer updating an existing skill over creating a new one.
- If nothing is worth refining, say so and stop. Don't force changes.
- Never edit skills that aren't in `managed/`. Existing skills in `~/.claude/skills/` that aren't symlinks to `managed/` are maintained elsewhere.

## Output

One-line summary of what you did (or "nothing to refine").
