---
name: userplane-audit
description: Verify an existing Userplane install is correct — read-only PASS/FAIL checklist with diffs.
argument-hint: ""
---

Delegate to the `audit-agent` subagent. Scope is the current repository. Expect a PASS/FAIL checklist (provider wiring, script placement, SSR hazards, setUser/setMetadata calls, CSP, env consistency) with file:line citations and a concrete diff for every FAIL. The agent must not apply edits — it is read-only.
