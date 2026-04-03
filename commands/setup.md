---
description: Check refine gate status and optionally enable or disable it for this project
argument-hint: '[--enable-gate|--disable-gate]'
allowed-tools: Bash(bash:*)
---

Run:

```bash
bash "${CLAUDE_PLUGIN_ROOT}/hooks/setup" --json $ARGUMENTS
```

Output rules:
- Present the setup output to the user.
- If `refineGateEnabled` is true, note that Claude will be blocked from stopping until `/refine` is run each session.
- If `refineGateEnabled` is false, note that the gate is inactive and sessions can end without running `/refine`.
