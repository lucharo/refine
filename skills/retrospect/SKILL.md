---
name: retrospect
depends: [self-session]
description: >
  Read your OWN current session transcript and look for things — dropped or unfinished asks,
  decisions made, lessons, untracked follow-ups, contradictions. Use when the user says
  "retrospect", "reflect on this session", "self-reflect", "what did I miss / drop / forget",
  "audit my own session", "did I capture everything". Takes an optional target ("retrospect on
  the decisions we made"); with no target it runs a general sweep. Reports findings — it does not
  act on them unless asked.
---

# retrospect — go through your own session and look for things

A fresh-eyes pass over the current session's transcript. You are context-poisoned — you know your
intent and have gone blind to your own gaps — so the reading is delegated to sub-agents with no
prior context, and their findings come back to you.

## 1. Frame the target and the gist

- **Target**: take it from the user's phrasing — "retrospect on dropped items", "on the decisions",
  "on lessons worth saving". **No target → a general sweep**: dropped/unfinished asks, decisions
  made, lessons/patterns, follow-ups that were mentioned but never tracked, contradictions, and
  commitments the agent made ("I'll…") that weren't fulfilled.
- **Gist**: write a 1-paragraph summary of what this session has been about (the idea of the
  session). Readers need it as context — hand it to each one.

## 2. Locate and shard the session

Use the **self-session** skill to find the transcript path and decide the reader count (one reader
per ~1500 lines, capped ~6; a short session → one reader). Assign each reader a contiguous line
range so they collectively cover the whole transcript with no gaps.

## 3. Fan out neutral readers

Spawn the readers (in parallel where the harness allows). Give EACH the same brief and its own
range — do **not** tell them your conclusions; neutrality is the point:

> You are auditing a session transcript slice with NO prior context. Read lines START–END of
> `<transcript>` (use `sed -n 'START,ENDp'`). The session is about: `<gist>`. Find everything
> matching: `<target, or the general-sweep list>`. For each finding: what it is, roughly where
> (line / phase), and one line of evidence. Return only findings, most important first. If your
> slice shows nothing relevant, say so.

If the whole session fits one reader, still delegate it to one neutral sub-agent rather than
reading it yourself — your own read is the biased one.

## 4. Merge and report

Collect the readers' findings, dedupe across overlapping slices, rank by importance, and report to
the user as a tight list: finding · where · why it matters. If the target was "dropped items",
exclude things the user *deliberately* changed course on (note them as deliberate, not gaps).

**Report-only by default.** Retrospect surfaces; it doesn't fix. Act on a finding only if the user
asks — and if something should be tracked, create a task/issue rather than leaving it a loose
thread.
