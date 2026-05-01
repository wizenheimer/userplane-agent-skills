---
name: userplane-debug
description: Diagnose a Userplane recording — root-cause a failing session via the workspace MCP.
argument-hint: "<recording-id | description>"
---

Delegate to the `debug-agent` subagent. Input: `$ARGUMENTS`.

- If `$ARGUMENTS` is empty, ask the user for either a recording ID or a short description of the session.
- If it looks like a recording ID, the subagent resolves the workspace and fetches directly.
- If it's a description ("Sarah's checkout bug yesterday"), the subagent uses `list_recordings` filters (`created_by`, `project_id`, `link_id`) to narrow candidates, then confirms before fetching.

Expect a timeline correlating console + network + actions, a stated root cause, and a concrete fix.
