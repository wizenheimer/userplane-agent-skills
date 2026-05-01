---
name: userplane-integrate
description: Add Userplane screen recording to this repo — detects the framework and writes the install.
argument-hint: ""
---

Delegate to the `integrate-agent` subagent. Ask it to detect the framework in the current repository, load the matching `userplane-{framework}` skill, and apply the install as concrete edits (provider/script, env wiring, CSP). If it detects Userplane is already installed, tell the user to run `/userplane:audit` instead.
