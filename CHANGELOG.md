# Changelog

## 0.5.1

- Add Codex-aware discovery guidance: inspect `~/.agents/skills` and repo `.agents/skills`, not
  just Claude-style skill folders
- Fix stale ownership guidance: `~/.agents/skills` is a discovery surface, not automatically a
  read-only managed install
- User skills now link into both `~/.claude/skills` and `~/.agents/skills` by default

## 0.5.0

- **Refine gate**: Stop hook blocks Claude from ending a session unless `/refine` was run — modeled on the Codex review gate pattern
- `/refine:setup` command to enable/disable the gate per project (`--enable-gate` / `--disable-gate`)
- PostToolUse tracker detects `/refine` invocation and writes a session marker
- Removed old unused `refine-stop` hook (superseded by `stop-gate`)

## 0.4.1

- Add rejection filter: distinguish project facts ("we use X") from behavioral rules ("always do Y"). Facts belong in docs/memory, only rules go in CLAUDE.md. Includes a litmus test: "what to DO" = rule, "what IS" = fact.

## 0.4.0

- Prefer skills over CLAUDE.md for multi-step workflows — CLAUDE.md is for rules, skills are for procedures
- Clear user vs repo CLAUDE.md guidance: user = how I work, repo = how this codebase works
- CLAUDE.md updates allowed (not just appends) when existing entries are stale — still ask first
- Memory files explicitly out of scope — refine handles skills and CLAUDE.md, not memory

## 0.3.1

- Proactive skill creation: if a workflow could bring value, create the skill — don't wait for proof of recurrence
- Proactively read CLAUDE.md as part of standard evaluation, don't wait to be asked

## 0.3.0

- Require genuine evaluation before concluding "nothing to refine" — agents were skipping straight to that conclusion without engaging
- Smart about when to use tools vs context: skills created in the same session don't need re-reading, skills only used do
- Fix broken symlink from npx skills test install

## 0.2.0

- Move refined skills to `~/.refined/` as a separate git repo
- Add CLAUDE.md as a refine target (append-only, ask before editing)
- Support vercel-labs/skills pattern (`.agents/skills/` as read-only)
- Add `AskUserQuestion` to allowed tools
- Fix symlinks: use `ln -sfn` with `$HOME`, check targets before editing
- Add `npx skills add` as primary install method
- Plugin skill personalisation documented as a known limitation
- Distinguish user vs local/repo skill scope (ask on creation)
- Git tracking per skill (default yes for user, no for local)

## 0.1.0

- Initial release
- `/refine` skill: scan session, read skills, create/refine, commit
- Stop hook (disabled by default)
- Plugin structure with marketplace.json for distribution
