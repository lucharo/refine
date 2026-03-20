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
- Check `~/.claude/skills/` for user skills
- Check `~/.claude/plugins/` for plugin skills (these are third-party — you can't edit them in place)

Also scan the session for patterns that no existing skill covers.

## Step 3: Decide

For each skill used:
- Did it work well as-is? → leave it alone
- Did the user correct something, or did the skill miss something? → refine it
- Is it a third-party plugin skill that needs personalising? → create a refined copy

For the session overall:
- Was there a repeated workflow or hard-won knowledge worth codifying? → new skill
- Was there a correction the user made that indicates a preference? → new skill or update existing

If nothing is worth changing, say so and stop. Don't force changes.

## Step 4: Edit

Write all skills to `refined/<skill-name>/SKILL.md` in this repo.

- **Refining a skill you own** (already in `refined/`): edit it in place
- **Personalising a third-party skill**: create a copy in `refined/` with your improvements. Don't modify the original plugin files — they'd get overwritten on update.
- **Creating a new skill**: write it to `refined/<name>/SKILL.md`

Keep changes minimal and focused.

## Step 5: Link and commit

```bash
# Symlink new skills into ~/.claude/skills/
ln -sf "$(pwd)/refined/<name>" ~/.claude/skills/<name>

# Commit in this repo
git add refined/
git commit -m "refine: <what changed and why>"
```

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
- Only edit skills in `refined/`. Never edit files in `~/.claude/skills/` directly or in plugin directories.
- Don't create skills for one-off tasks that won't recur.
- Don't capture things obvious from reading code or CLAUDE.md.

## Output

One-line summary of what you did (or "nothing to refine").
