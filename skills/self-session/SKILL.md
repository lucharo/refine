---
name: self-session
description: >
  Locate and size the CURRENT Claude Code session's own transcript so an agent can read
  its own history. Use when you need to find "my session id", "this session's transcript",
  read/scan the current conversation, or decide how many sub-agents to shard a long session
  across. Building block for the retrospect skill and refine's report-only mode.
---

# self-session — find and size your own transcript

Claude Code writes every session to a JSONL transcript named `<session-id>.jsonl`. This skill is
the reliable way for an agent to find *its own* current transcript and decide how to read it.

## 1. Find the transcript (reliable)

The harness sets `CLAUDE_CODE_SESSION_ID` in the environment, and the transcript filename is that
id. Search all project dirs for the matching file — the name match is exact, so parallel sessions
don't confuse it:

```bash
SID="${CLAUDE_CODE_SESSION_ID:-}"
if [ -n "$SID" ]; then
  TRANSCRIPT="$(find "$HOME/.claude/projects" -name "$SID.jsonl" -print -quit 2>/dev/null)"
else
  # Fallback only if the env var is missing: newest transcript by mtime. LESS reliable under
  # parallel sessions — verify the tail matches the current task before trusting it.
  TRANSCRIPT="$(ls -t "$HOME"/.claude/projects/*/*.jsonl 2>/dev/null | head -1)"
fi
echo "$TRANSCRIPT"
```

The project dir is slugified from the session's *original* working directory (where it started),
not the current cwd — so search all of `~/.claude/projects/`, don't guess the slug.

## 2. Size it, and decide sharding

```bash
lines=$(wc -l < "$TRANSCRIPT"); bytes=$(wc -c < "$TRANSCRIPT")
echo "lines=$lines bytes=$bytes"
```

Rule of thumb for how many reader sub-agents to fan out:
- **≤ ~1500 lines** (roughly a short–medium session): **one** reader.
- Otherwise: **one reader per ~1500 lines**, capped at ~6 (diminishing returns beyond that).
  Assign each reader a contiguous line range with `sed -n 'START,ENDp' "$TRANSCRIPT"`.

Readers should slice by range, never each read the whole file. Very large tool-result lines
(embedded file dumps, base64) inflate byte size — line count is the better sharding signal.

## 3. Reading notes (JSONL shape)

- One JSON event per line. The **first user message** holds the original ask/brief.
- `system-reminder` blocks are harness-injected context, **not** the user talking.
- Tool results can be huge; when mining for intent/decisions, skim past them and focus on user
  turns and the agent's own text between tool calls.
- To pull just the human-readable spine cheaply: filter to `type == "user"`/`"assistant"` text
  with `jq` before handing slices to readers, if the raw file is unwieldy.
