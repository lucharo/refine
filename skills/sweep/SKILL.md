---
name: sweep
description: >
  One-shot deep refine: sweep the WHOLE current session across neutral reader sub-agents and
  surface every refinement (new skills, skill improvements, CLAUDE.md additions), then apply the
  ones you approve. Use when the user says "sweep the session", "deep refine", "retrospect and
  refine", "thorough refine", or wants a long session refined so no lesson is missed. This is the
  retrospect+refine combo in a single trigger.
disable-model-invocation: true
allowed-tools:
  - Bash
  - Read
  - Edit
  - Write
  - Grep
  - Glob
  - AskUserQuestion
---

# sweep — retrospect + refine, in one trigger

The delegated, report-first path of `refine`, packaged as a single user-invoked command. You are
context-poisoned about your own session, so the reading is done by neutral sub-agents that run the
`refine` criteria over the transcript; you only merge, present, and apply what's approved.

It composes the existing pieces — nothing here is new logic:

1. **Locate + size** — use the **self-session** skill to find this session's transcript (via
   `CLAUDE_CODE_SESSION_ID`) and decide the reader count (one per ~1500 lines, capped ~6; a short
   session → one reader). Assign each reader a contiguous line range covering the whole file.

2. **Gist** — write a one-paragraph summary of what the session was about. Readers need it as
   context; hand it to each one.

3. **Fan out neutral readers** — spawn one sub-agent per shard (in parallel where possible). Each
   reader **runs the `refine` skill's Step-1 scan in report-only mode** over its range — skills run
   in sub-agents by default, so the reader loads `refine` itself and applies its definition of a
   refinement; you don't restate the criteria. Give each reader the gist + its line range, and
   **never your own conclusions** (neutrality is the point). Each returns ranked candidates tagged
   `new-skill` / `skill-improvement` / `claude-md`, with evidence and a concrete suggested change.

4. **Merge** — dedupe across overlapping shards, rank by value, and drop anything already captured
   (check the live skills / CLAUDE.md before proposing).

5. **Present + apply** — show the merged candidates to the user. Apply only the ones they approve,
   via `refine` Steps 4-5 (write/edit the skill or CLAUDE.md, commit per refine's conventions).
   Report-only until they choose.

## Notes

- `sweep` does not replace `/refine` (inline, identify-and-apply-as-you-go). Reach for `sweep` when
  the session is long or you want the thorough, no-lesson-missed pass.
- It's `disable-model-invocation: true` — a deliberate, sub-agent-spawning action the user triggers,
  not something Claude fires on its own (it costs several sub-agents).
- If the session is tiny, one neutral reader is still better than reading it yourself.
