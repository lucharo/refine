---
name: refine
depends: [retrospect, self-session]
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
  - Task
---
# Refine
Review this session. Identify patterns worth capturing as new skills, improvements to existing skills, or additions to CLAUDE.md.

## Modes
Two ways to run — same Step 1 scan, different ending:
- **Default (identify + apply):** scan the session yourself, then make the changes (Steps 2-5).
- **Report-only / delegated (identify, don't apply):** when the user says "refine but don't apply",
  "what would you refine", "refine via sub-agent", or on a **long session where you want a thorough
  sweep so no lesson is missed**, delegate the scan instead of doing it yourself. Run the
  **retrospect** skill with the target *"skill / CLAUDE.md refinement opportunities"* — it uses
  **self-session** to locate and shard the transcript across neutral reader sub-agents (one per
  ~1500 lines), and returns refinement candidates. The sub-agents **only report; they do not edit.**
  You then present the candidates to the user, or apply the approved ones via Steps 4-5. This is a
  mode of refine, not a separate skill — Steps 1-5 below still define what a "refinement" is.

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
ls -la ~/.agents/skills/ 2>/dev/null
ls -la .claude/skills/ 2>/dev/null
ls -la .agents/skills/ 2>/dev/null
ls -la ~/.refined/ 2>/dev/null
```
Check the locations your current agent actually discovers. For Codex, user skills live in
`~/.agents/skills` and repo skills in `.agents/skills`. For Claude-style setups, look at
`~/.claude/skills` and `.claude/skills`.
For each skill, whether from context or from reading, evaluate:
- Is the description specific enough to trigger correctly?
- Are the instructions clear and complete based on how the skill was actually used?
- Did the session reveal gaps, edge cases, or missing steps?
- Are references to other files/skills correct?
Check ownership before editing:
- Symlink → `~/.refined/`: yours to refine
- Symlink → a plugin dir or managed install: read-only (updates would overwrite)
- Regular file in `~/.claude/skills/` or `.claude/skills/`: yours to refine
- Regular entry in a repo `.agents/skills/`: not automatically repo-owned — `npx skills`
  installs can land there as regular files too. Treat it as repo-owned (editable in that repo)
  only if the repo's git tracks it; if it's untracked or listed in a skills lock/manifest
  (e.g. `skills-lock.json`), treat it as externally managed and don't edit it.
- Entry in `~/.agents/skills/`: inspect the target first. Treat it as a discovery surface, not
  the source of truth. If it points to `~/.refined/`, edit the target in `~/.refined/`. If it
  points into a plugin cache or managed install, do not edit it.
- If an off-the-shelf or shared skill needs personal refinement, prefer forking it into
  `~/.refined/<name>` and repointing the user discovery link there, even when the current
  discovery-surface copy looks editable. The goal is to keep personal refinements in one place and
  disable the base copy at the user discovery surface rather than carrying divergent edits in a
  bundled install.
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
- If the user corrected agent behavior in this session, do not stop at a vague recommendation. Draft the exact 1-3 line CLAUDE.md addition or update you would make, and present that concrete snippet when asking for confirmation.
- If the correction is about repo-state handling (for example, "pull latest" in a dirty repo), separate the general rule from the one-off repo facts: capture the behavior rule in CLAUDE.md, not the repo's specific state.
- A single correction often contains BOTH a fact and a rule. Extract the behavioral rule for CLAUDE.md yourself; for the fact half, do NOT edit docs or memory files (out of scope — see "Memory files are out of scope") — surface it to the user as a suggested docs/memory note instead. Example: "X and Y are completely different things" is a fact about X/Y plus a rule: "don't conflate unrelated concepts without evidence". Dropping either half is a failure mode.
**What does NOT go in CLAUDE.md or skills:**
- Project-specific facts or implementation decisions ("we use h-dvh not h-screen", "auth uses Better Auth") — these are documentation or memory, not rules.
- Change-log entries ("fixed X by switching to Y") — git history covers this.
- Generic engineering advice ("investigate root causes") — too obvious to be a useful rule.
The test: if it tells you WHAT to do in a situation, it's a rule (CLAUDE.md). If it tells you what IS, it's a fact (docs/memory). A single correction can contain BOTH — extract the rule and flag the fact for docs/memory as a follow-up. Routing only the fact to memory and declaring "nothing to refine" is a common failure when the user corrected visible behavior this session.
Choose the right file:
- **User CLAUDE.md** (`~/.claude/CLAUDE.md`): who the user is and how they work across all projects — tool preferences, communication style, secret handling, memory conventions
- **Repo CLAUDE.md** (`CLAUDE.md` or `.claude/CLAUDE.md`): how this specific codebase works — stack, commands, validation, deploy, commit conventions
If it would be useful in a different repo, it belongs in user CLAUDE.md. If it only makes sense for this project, it belongs in repo CLAUDE.md.
Your default stance should be to create or improve something. Most sessions contain at least one workflow, preference, or piece of knowledge worth capturing. "Nothing to refine" is valid but should be rare — it means you genuinely found no reusable workflow, no skill to improve, and no CLAUDE.md update needed.
## Step 4: Edit
### Skills
**Refining an existing skill**: first `ls -la` to check if it's a symlink.
- Symlink to `~/.refined/` → edit the target file in `~/.refined/`
- Symlink to a plugin dir or managed install → do NOT edit (read-only — managed externally)
- Regular file in `~/.claude/skills/` or `.claude/skills/` → edit in place
- Regular entry in a repo `.agents/skills/` → edit in place only if the repo's git tracks it;
  untracked or lock-file-listed entries are externally managed — don't edit
- Entry in `~/.agents/skills/` → inspect the target first; if it points to `~/.refined/`, edit
  `~/.refined/`, not the discovery link
- If the skill is effectively off-the-shelf but you want a personal variant, copy the whole skill
  into the store using the **same placement rules as a new user skill** below (resolve tier, then
  the chosen root's flat-vs-category marker) — not unconditionally to `~/.refined/<name>/`. Then
  replace the user discovery entry with a symlink to that refined copy. Keep a reversible backup of
  the previous discovery-surface directory when practical.
**Creating a new skill**: ask the user which scope:
- **User skill** — useful across all projects. Write to the user skill store `~/.refined/`.
  Placement resolves in two **orthogonal** steps: pick the ROOT (confidentiality tier), then apply
  that root's own layout marker. A flat store is never silently reorganised into categories.

  **Step A — pick the ROOT (confidentiality tier).** Only applies if `~/.refined/skills/private/`
  exists; otherwise ROOT is `~/.refined/`.
  - **public/shareable** (generic workflows — the default) → ROOT is `~/.refined/`.
  - **confidential** (names internal systems, endpoints, org tooling, or private life / personal
    accounts and infra not for any public repo) → ROOT is
    `~/.refined/skills/private/<profile>/`.

  Discover the available profiles at runtime — `ls -d ~/.refined/skills/private/*/` — and pick the
  one matching the skill's confidentiality domain (a user might have e.g. `work/` and `home/`, or
  an employer-named dir). **Never invent a new profile.** Infer the tier from the skill's content;
  when genuinely torn, ask. Before writing employer-confidential content to any root other than a
  `private/` profile, **stop and ask the user to confirm** (AskUserQuestion) — do not just warn; a
  mis-tiered skill leaks to machines (or repos) that shouldn't surface it.

  **Step B — apply the ROOT's layout marker.** Each root carries its own `CATEGORIES.md`; read the
  marker for the root chosen in Step A, never a different root's. **The marker path differs per
  root** — stat the exact path below, not `<ROOT>/CATEGORIES.md` blindly:
  - public root (`~/.refined`) → marker is `~/.refined/skills/CATEGORIES.md`
  - private profile root (`~/.refined/skills/private/<profile>/`) → marker is
    `<ROOT>/CATEGORIES.md`
  - **Flat layout (the default)** — if that root's marker does not exist, write to
    `<ROOT>/<name>/SKILL.md` (for the public root that is `~/.refined/<name>/SKILL.md`). Plain
    list, no grouping. This is what most users get.
  - **Category layout (opt-in)** — if the marker exists, the root is grouped. Read it for the
    one-line category definitions, **infer** the best-fit category from what the skill does, and
    write to `~/.refined/skills/<category>/<name>/SKILL.md` for the public root, or
    `<ROOT>/<category>/<name>/SKILL.md` for a private profile root. Only when two categories fit
    equally and you genuinely can't decide, ask the user (offer the closest 2-3). If the file
    exists but is empty, ask the user which category to use. Never drop the skill at the root.
  - To opt in, a user creates the marker at that root's path above (`~/.refined/skills/CATEGORIES.md`
    for the public root) listing their categories (one per line, `- name — one-line definition`);
    refine reads it from then on.
- **Local/repo skill** — specific to the current project. Write to the repo's skill location
  (`.claude/skills/<name>/SKILL.md` for Claude-style repos, `.agents/skills/<name>/SKILL.md` for
  Codex/shared repos).
Before committing the name, check for clashes across every location a skill with
that name could already live (or that refine might write to):
```bash
# Ensure no plugin already uses this name
grep -q '"<name>' ~/.claude/plugins/installed_plugins.json 2>/dev/null && echo 'CLASH: plugin exists'
# Ensure no existing skill uses this name in any discovery surface or source dir
for d in ~/.refined ~/.claude/skills ~/.agents/skills .claude/skills .agents/skills; do
  [ -e "$d/<name>" ] && echo "CLASH: $d/<name> exists"
done
# Category store (only when opted in): names must be unique across every category and every
# private profile — they all flatten into one discovery hub, and `ln -sfn` overwrites silently.
if [ -f "$HOME/.refined/skills/CATEGORIES.md" ]; then
  for d in ~/.refined/skills/*/; do [ -e "$d<name>" ] && echo "CLASH: $d<name> exists"; done
fi
for d in ~/.refined/skills/private/*/ ~/.refined/skills/private/*/*/; do
  [ -e "$d<name>" ] && echo "CLASH: $d<name> exists"
done
```
If a clash is found, pick a more specific name (e.g. `personal-<name>`).
### CLAUDE.md
**Always ask before editing CLAUDE.md.** Show the proposed change and ask the user to confirm.
**First check the file isn't generated.** Many people keep dotfiles under a manager (chezmoi, yadm, stow). Editing a rendered `CLAUDE.md` that a manager owns is undone at the next apply, and where two files render from one shared source, changing one breaks them apart. Check with `chezmoi source-path ~/.claude/CLAUDE.md` — if it resolves, edit **that source**, not the rendered file, and say which file you actually changed. `chezmoi re-add` skips `.tmpl` and `modify_` sources, so an edit to the rendered file cannot be recaptured afterwards; it is simply lost. The source may also carry `{{ if ... }}` profile blocks that a rendered-file edit cannot express. Check for a twin (`~/.codex/AGENTS.md` is a common one) rendering from the same source.
**Then read the file's own rules before deciding placement.** Read its first ~30 lines including HTML comments — those are stripped at load but visible to the Read tool, and are where maintainer instructions live. A file may declare a routing header saying which kinds of rule go where, a size budget, or a section reserved for one-line gotchas. Follow what the file says, and state which route you took when you ask.
**Placement, in order of preference:**
- Into the **section that already covers the topic**. If a related entry exists, rewrite it in place rather than adding a second that half-agrees — two rules on one topic is how a file starts contradicting itself. If they genuinely conflict, surface the conflict and let the user choose; don't silently keep both.
- If the file has a **list for that kind of rule** (a "Footguns" or "Gotchas" list), add **one bullet to that list**, not a new `##` section.
- **Never append at end-of-file.** A file may close with an index, a skills table, or a deliberate final instruction placed there so it lands last; appending past it displaces the thing it is positioned to be. End-of-file is the default only when the file has no discernible structure.
**Respect a size budget.** If the file states one (an HTML comment, a stated line or character target) or is already large — Claude Code warns above 40k characters and the docs target under 200 lines — count before and after. If the addition pushes it over, propose a trim in the *same* pass: name the entry now covered by a skill, or the two that should merge. Never add to a file that is over budget without saying so.
- Keep additions concise (1-3 lines per entry)
- File choice is covered in Step 3 above (user vs repo CLAUDE.md)
- When asking, include the exact text to add, where it will go, and a one-sentence reason it belongs in CLAUDE.md rather than a skill.
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
**Git-track user skills by default — don't ask.** User skills (written to `~/.refined/`) are git-tracked as a firm convention: stage and commit them without surfacing it as a choice. Repo/local skills default to NOT tracked separately (they live in the project's own git). Only ask about tracking when the user has signalled a reason to deviate. When you do need a scope question (Step 4, user vs repo), don't bundle a redundant "git or not" option alongside it — the git default follows from the scope.
**User skills** (written to `~/.refined/`). `<store-path>` is the skill dir you wrote — it has one
shape per ROOT/layout combination from Step 4:
- public root, flat → `~/.refined/<name>`
- public root, category → `~/.refined/skills/<category>/<name>`
- private profile root, flat → `~/.refined/skills/private/<profile>/<name>`
- private profile root, category → `~/.refined/skills/private/<profile>/<category>/<name>`

**If the store ships a relink tool, use it** — it wires every discovery surface from the store in
one step (categories included), so you don't hand-roll symlinks:
```bash
[ -x "$HOME/.scripts/skills-sync" ] && "$HOME/.scripts/skills-sync" relink
```
Otherwise link by hand into the surfaces the user actually uses (skip any that don't apply —
e.g. skip `~/.claude/skills` for a Codex-only setup), `mkdir -p` each parent first:
Replacing an off-the-shelf user skill? **Move the existing discovery directory aside BEFORE
linking** — `ln -sfn` will not cleanly replace a real directory:
```bash
# Per surface, only for the surfaces the user actually uses — every surface you relink,
# not just the Claude one:
for s in "$HOME/.claude/skills" "$HOME/.agents/skills"; do
  [ -d "$s/<name>" ] && [ ! -L "$s/<name>" ] \
    && mv "$s/<name>" "$s/<name>.base-YYYY-MM-DD"
done
```
Then link, one command per surface — run only the ones that apply (skip `~/.claude/skills` for a
Codex-only setup, skip `~/.agents/skills` for a Claude-only one):
```bash
# Claude-style discovery
mkdir -p "$HOME/.claude/skills" && ln -sfn "<store-path>" "$HOME/.claude/skills/<name>"
# Codex/shared user-level discovery
mkdir -p "$HOME/.agents/skills" && ln -sfn "<store-path>" "$HOME/.agents/skills/<name>"
```
Then track it (user skills are git-tracked by default). **Commit in the repo that OWNS the path** —
a private profile dir is typically its own repo (or gitignored/nested inside the store), so never
`git add` a `private/` path from the outer `~/.refined` repo:
```bash
# Public root — <store-relpath> is skills/<category>/<name> for a category store, else <name>
git -C "$HOME/.refined" add "<store-relpath>/" && git -C "$HOME/.refined" commit -m "refine: <what changed and why>"
# Private profile root — commit inside the profile repo; <profile-relpath> is
# <category>/<name> if that profile has a CATEGORIES.md, else <name>
git -C "$HOME/.refined/skills/private/<profile>" add "<profile-relpath>/" \
  && git -C "$HOME/.refined/skills/private/<profile>" commit -m "refine: <what changed and why>"
```
If the private profile dir is not itself a git repo, stop and ask the user how it should be tracked
rather than falling back to the outer store repo.
**Local/repo skills** (written to the repo's skill directory):
```bash
# Commit in the project repo if the user wants
git -C <project-root> add <repo-skill-dir>/<name>/ && git -C <project-root> commit -m "refine: <what changed and why>"
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
If something looks like a fact to remember rather than a rule to follow or a workflow to codify, it's not refine's job to write it — surface it as a suggested note for the user's docs/memory system instead of editing those files.
## Constraints
- Scale the number of skills touched or created to the session's evidence — a rich session can justify several, a thin one none.
- A skill should be under 200 lines. Longer means it's doing too much.
- Prefer refining an existing skill over creating a new one.
- Never edit externally managed skills (plugins, `npx skills` installs) — they get overwritten on update.
- Always check symlink targets before editing — don't follow symlinks into plugin dirs or `.agents/skills/`.
- Don't create skills for truly one-off tasks (e.g. "fix this specific bug"). But if a workflow could plausibly be useful again, create the skill — don't wait for proof of recurrence.
- Don't capture things obvious from reading code or CLAUDE.md.
- Always ask before editing CLAUDE.md. Place each entry in the section that already covers its topic, never at end-of-file, and rewrite a related entry rather than stacking a second beside it.
- Check whether CLAUDE.md is generated by a dotfile manager before editing it; if so, edit the source, not the rendered file.
- Honour a size budget the file declares, and trim in the same pass when an addition would break it.
## Output
One line, counted: `added N, merged N, removed N` across skills and CLAUDE.md (or "nothing to refine"). Report merges and removals explicitly — a run that only ever adds should be visible as one, because that is how these files silently grow past the point where they're followed.
