---
name: userplane-debug
description: Diagnose a Userplane recording from Codex using the Userplane workspace MCP. Use when the user provides a recording ID or describes a failed Userplane session.
license: MIT
metadata:
  author: Userplane
  version: 0.1.0
  category: workflow
  tags:
    - userplane
    - screen-recording
    - codex
    - debug
---

# Userplane Debug Workflow

Resolve and analyze a Userplane recording, then correlate console, network, and user action data into a root-cause timeline.

## When to use

- The user provides a Userplane recording ID.
- The user describes a failed session, such as "Sarah's checkout failure yesterday".
- The user asks Codex to root-cause a bug from Userplane recording evidence.

## Workflow

1. Determine whether the input is a recording ID or a natural-language description.
2. Use the `userplane-workspace` MCP server to resolve workspace context before fetching recording details.
3. If there is exactly one workspace, use it. If there are multiple plausible workspaces, ask the user to choose.
4. For a recording ID:
   - call the recording info tool to confirm it exists
   - stop if it is expired or unavailable
5. For a description:
   - list workspaces and projects as needed
   - search recordings using inferred filters such as user, project, link, and time range
   - narrow to the most likely 1-3 candidates
   - ask the user to confirm the recording before fetching detailed session data
6. Once the recording is confirmed, fetch:
   - recording info
   - console output
   - network requests
   - user actions
7. Correlate events by timestamp and focus on the failure window.
8. If the repository is available, search for code related to the failing route, API, component, or console stack.
9. Report the root cause and a concrete fix. Apply edits only if the user explicitly asked for a code change.

## Report format

```text
Summary: <one sentence>

## Timeline
- <timestamp> <relevant action/error/request>

## Root cause
<evidence-backed conclusion>

## Fix
<concrete code-level recommendation>
```

## Hard rules

- Do not fetch console, network, or action data until the recording is confirmed.
- Filter large MCP responses early to the failure window.
- Quote or cite the exact console line, network row, or action that supports the conclusion.
- Say "unclear from the recording" when the evidence does not support a root cause.
