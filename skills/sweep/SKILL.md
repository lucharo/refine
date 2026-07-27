---
name: sweep
depends: [self-session, retrospect, refine]
description: >
  One-shot deep refine: sweep the WHOLE current session across neutral reader sub-agents and
  surface every refinement (new skills, skill improvements, CLAUDE.md additions), then apply the
  ones you approve. Use when the user says "sweep the session", "deep refine", "retrospect and
  refine", "thorough refine", or wants a long session refined so no lesson is missed. This is the
  retrospect+refine combo in a single trigger.
disable-model-invocation: true
allowed-tools:
  - Task
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

1. **Locate + size** — use the **self-session** skill to find this session's transcript from the
   current harness ID and decide the reader count (one per ~1500 lines, capped ~6; a short session
   → one reader). Assign each reader a contiguous line range covering the whole file.

2. **Gist** — write a one-paragraph summary of what the session was about. Readers need it as
   context; hand it to each one.

3. **Fan out neutral readers** — spawn one sub-agent per shard (in parallel where possible). Each
   reader **runs the `refine` skill's Step-1 scan in report-only mode** over its range — skills run
   in sub-agents by default, so the reader loads `refine` itself and applies its definition of a
   refinement; you don't restate the criteria. Give each reader the gist, the **transcript path**
   from self-session, and the exact command to read its slice —
   `sed -n 'START,ENDp' "<transcript-path>"` — and **never your own conclusions** (neutrality is
   the point). Each returns ranked candidates tagged
   `new-skill` / `skill-improvement` / `claude-md`, with evidence and a concrete suggested change.

   Use the Codex App's native subagent tools for this fan-out. Never fall back to
   `agent-sdk-manager`, ad-hoc CLI child sessions, or another external agent harness. If native
   subagent tools are not exposed in the current turn, stop and tell the user that native
   multi-agent support is disabled or unavailable, then retry in a fresh session after it is fixed.

4. **Merge** — dedupe across overlapping shards, rank by value, and drop anything already captured
   (check the live skills / CLAUDE.md before proposing).

5. **Present + apply** — show the merged candidates to the user. Apply only the ones they approve,
   via `refine` Steps 4-5 (write/edit the skill or CLAUDE.md, commit per refine's conventions).
   Report-only until they choose.

6. **Completion gate** — finish only after every shard returned, candidates were deduplicated
   against live files, every approved edit was validated and committed, and the final report lists
   commit IDs plus anything deliberately left unresolved.

## Notes

- `sweep` does not replace `/refine` (inline, identify-and-apply-as-you-go). Reach for `sweep` when
  the session is long or you want the thorough, no-lesson-missed pass.
- It's `disable-model-invocation: true` — a deliberate, sub-agent-spawning action the user triggers,
  not something Claude fires on its own (it costs several sub-agents).
- If the session is tiny, one neutral reader is still better than reading it yourself.
