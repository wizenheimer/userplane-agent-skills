---
name: userplane-integrate
description: Add Userplane screen recording to a web app from Codex. Use when the user asks to install, set up, integrate, or add Userplane to the current repository.
license: MIT
metadata:
  author: Userplane
  version: 0.1.0
  category: workflow
  tags:
    - userplane
    - screen-recording
    - codex
    - integration
---

# Userplane Integrate Workflow

Install Userplane screen recording into the current web app as concrete code edits.

## When to use

- The user asks Codex to add or install Userplane.
- The user wants framework-specific setup, including provider or script wiring.
- The user asks for a working Userplane integration instead of general guidance.

## Workflow

1. Detect the framework by reading `package.json` and nearby config files.
   - Next.js -> use `userplane-nextjs`
   - Nuxt -> use `userplane-nuxt`
   - Angular -> use `userplane-angular`
   - Vue (Vite) -> use `userplane-vue`
   - SvelteKit -> use `userplane-sveltekit`
   - Astro -> use `userplane-astro`
   - TanStack Start -> use `userplane-tanstack-start`
   - React (Vite) -> use `userplane-react`
   - Static HTML -> use `userplane-static-html`
2. Load the matching framework skill as the source of truth.
3. Also consult `userplane-web-sdk`, `userplane-metadata-sdk`, `userplane-cdn`, and `userplane-best-practices`.
4. Search for existing Userplane evidence before editing:
   - `@userplane/sdk`
   - `userplane.io`
   - `USERPLANE_` env vars
   - existing Userplane provider, plugin, or script code
5. If Userplane is already installed, stop and recommend `userplane-audit`.
6. Ask for the write key only if it is needed and cannot be represented as an env var placeholder.
7. Apply the framework-specific install:
   - create or update the provider, plugin, script tag, or SDK init point
   - wire the framework-appropriate env var
   - keep browser-only SDK calls off server render paths
   - add CSP guidance or config where the framework supports it
8. Finish with changed files, the integration shape used, and the one next user action.

## Hard rules

- Write real edits when integration is needed.
- Do not invent SDK APIs; use the loaded framework and SDK skills.
- Do not duplicate an existing Userplane provider or script.
- Do not add unrelated analytics, telemetry, packages, or refactors.
- Do not hardcode a production write key.
