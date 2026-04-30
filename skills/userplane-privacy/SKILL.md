---
name: userplane-privacy
description: Audit a repo's Userplane privacy posture from Codex. Use when the user asks about PII, redaction, blur attributes, metadata leakage, GDPR, or privacy risks in recordings.
license: MIT
metadata:
  author: Userplane
  version: 0.1.0
  category: workflow
  tags:
    - userplane
    - screen-recording
    - codex
    - privacy
---

# Userplane Privacy Workflow

Audit the current repo for privacy and sensitive-data risks in a Userplane recording setup.

## When to use

- The user asks whether sensitive data is protected in Userplane recordings.
- The user asks for a privacy, PII, GDPR, redaction, or blur review.
- The user wants to find metadata leaks or CSP gaps that affect recording privacy.

## Workflow

1. Confirm Userplane appears to be present. Search for `userplane`, `@userplane/sdk`, `USERPLANE_`, and Userplane script URLs.
2. If no Userplane install is found, stop and tell the user to run `userplane-integrate` first.
3. Load `userplane-sensitive-data` as the ground truth.
4. Also consult `userplane-metadata-sdk` and `userplane-cdn`.
5. Scan for missing blur coverage:
   - password and email inputs
   - fields whose name, id, label, or nearby text suggests SSN, DOB, tax ID, passport, license, account, routing, card, CVV, IBAN, address, phone, or other PII
   - PII-adjacent containers without `data-userplane-blur` or a supported equivalent
6. Scan `setUser` and `setMetadata` calls for raw sensitive values:
   - email
   - phone
   - address
   - date of birth
   - government IDs
   - payment data
7. Check CSP config for Userplane domains and third-party iframe hosts used by the app.
8. Check inline handlers and visible template output for values that could leak PII into recordings.
9. Report severity-ranked findings with file:line citations and concrete diffs.

## Report format

```text
Summary: <issue count> issues (<high> high, <medium> medium, <low> low)
Top fix: <one sentence>

## High
- [file:line] <finding> -> <diff>

## Medium
...

## Low / notes
...
```

## Hard rules

- Read-only. Do not edit files.
- Every finding needs file:line evidence and a concrete fix.
- Do not flag already blurred elements. Check ancestors before reporting.
- Stable pseudonymous IDs are acceptable metadata; raw PII is not.
