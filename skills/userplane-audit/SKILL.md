---
name: userplane-audit
description: Audit an existing Userplane screen recording integration from Codex. Use when the user asks to verify, review, check, or audit their Userplane setup.
license: MIT
metadata:
  author: Userplane
  version: 0.1.0
  category: workflow
  tags:
    - userplane
    - screen-recording
    - codex
    - audit
---

# Userplane Audit Workflow

Verify an existing Userplane integration without making edits.

## When to use

- The user asks whether Userplane is installed correctly.
- The user wants a PASS/FAIL checklist for their setup.
- The user asks for install verification, CSP checks, metadata checks, or SSR safety checks.

## Workflow

1. Confirm Userplane appears to be present. Search for `userplane`, `@userplane/sdk`, `USERPLANE_`, and Userplane script URLs.
2. If no Userplane install is found, stop and tell the user to run `userplane-integrate` first.
3. Detect the framework and load the matching framework skill:
   - `userplane-nextjs`
   - `userplane-nuxt`
   - `userplane-angular`
   - `userplane-vue`
   - `userplane-sveltekit`
   - `userplane-astro`
   - `userplane-tanstack-start`
   - `userplane-react`
   - `userplane-static-html`
4. Also consult `userplane-best-practices`, `userplane-cdn`, `userplane-web-sdk`, and `userplane-metadata-sdk`.
5. Check the repo against the expected framework pattern:
   - provider, plugin, init call, or script tag is present at the right location
   - script placement and CSP match Userplane guidance
   - SDK calls are SSR-safe
   - `setUser` and `setMetadata` usage avoids raw sensitive data
   - env vars are consistent across development and production config
6. Report PASS or FAIL for each check with file:line citations.
7. For every FAIL, include a concrete diff or focused code snippet the user can apply.

## Report format

```text
Summary: <pass count> PASS, <fail count> FAIL
Top fix: <single most important fix>

## Checks
- PASS/FAIL [file:line] <check result>

## Suggested diffs
...
```

## Hard rules

- Read-only. Do not edit files.
- Cite file:line for every claim.
- Do not audit against a close-but-wrong framework skill.
- Do not flag style preferences; report correctness issues only.
