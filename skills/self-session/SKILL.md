---
name: self-session
description: >
  Locate and size the CURRENT Claude Code or Codex session transcript so an agent can read
  its own history. Use when you need to find "my session id", "this session's transcript",
  read/scan the current conversation, or decide how many sub-agents to shard a long session
  across. Building block for the retrospect skill and refine's report-only mode.
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
---

# self-session — find and size your own transcript

Claude Code and Codex both write JSONL transcripts. This skill is the reliable way for an agent
to find *its own* current transcript and decide how to read it.

## 1. Find the transcript (reliable)

Use the exact harness ID when available so parallel sessions cannot be confused.

If [`trawl`](https://github.com/lucharo/trawl) is installed, it already does this and is the
better answer — it resolves a *thread*, so a session that was resumed or whose worktree moved
still returns all of its transcript files rather than whichever fragment matched first:

```bash
trawl resolve                 # prints: id, agent, project, path
```

It **exits non-zero rather than guessing**: 3 = no session id in the environment, 4 = id valid
but not yet indexed. That refusal is the point — under parallel sessions a confident wrong
transcript is worse than a failure.

Without `trawl`, resolve by ID directly:

```bash
SID="${CLAUDE_CODE_SESSION_ID:-}"
if [ -n "$SID" ]; then
  TRANSCRIPT="$(find "$HOME/.claude/projects" -name "$SID.jsonl" -print -quit 2>/dev/null)"
elif [ -n "${CODEX_THREAD_ID:-}" ]; then
  TRANSCRIPT="$(find "$HOME/.codex/sessions" -name "*${CODEX_THREAD_ID}.jsonl" -print -quit 2>/dev/null)"
else
  # Fallback only if both IDs are missing: newest transcript by mtime. LESS reliable under
  # parallel sessions — verify the tail matches the current task before trusting it.
  TRANSCRIPT="$(find "$HOME/.claude/projects" "$HOME/.codex/sessions" -name '*.jsonl' \
    -type f -print 2>/dev/null | xargs ls -t 2>/dev/null | head -1)"
fi
echo "$TRANSCRIPT"
```

Claude's project dir is slugified from the session's *original* working directory, not the current
cwd. Codex stores dated rollout files under `~/.codex/sessions/`. Search the full roots; do not guess.

## 2. Size it, and decide sharding

With `trawl`, ask for the ranges instead of computing them — it prints one ready-to-run command
per reader, already capped:

```bash
trawl shard <id>              # contiguous ranges + the digest command for each
```

Each printed command is a `trawl digest … --from-line N --to-line M`, which returns user turns
plus truncated assistant text with harness scaffolding stripped, so a reader gets the
conversation rather than raw JSONL.

Without `trawl`:

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
