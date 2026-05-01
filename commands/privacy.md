---
name: userplane-privacy
description: Audit this repo's Userplane privacy posture — blur attrs, metadata PII, CSP, inline leaks.
argument-hint: ""
---

Delegate to the `privacy-agent` subagent. Scope is the current repository. Expect a severity-ranked report covering (1) missing `data-userplane-blur` on PII-adjacent elements, (2) raw PII in `setMetadata` / `setUser` payloads, (3) CSP `frame-src` gaps for third-party iframes (Stripe, Auth0, etc.), and (4) inline handlers or template expressions that leak PII into the DOM. Every finding must include file:line and a concrete diff. Read-only.
