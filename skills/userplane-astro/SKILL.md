---
name: userplane-astro
description: Integrate Userplane screen recording into an Astro application. Use when user asks to set up screen recording in Astro, add Userplane to an Astro site, or initialize Userplane in an Astro script block.
license: MIT
compatibility: Astro 4+
metadata:
  author: Userplane
  version: 0.0.1
  category: integration
  tags:
    - userplane
    - screen-recording
    - astro
---

# Userplane Astro Integration

Install Userplane screen recording in an Astro application using client-side script blocks.

## When to use

- Adding Userplane to an Astro project
- Setting up screen recording via Astro `<script>` blocks
- Configuring Userplane with Astro middleware for SSR redirect handling

## Examples

### Example 1: Basic setup
User says: "How do I add Userplane to my Astro site?" or "Set up screen recording in Astro" or "Integrate Userplane with Astro"
For CDN-only: add the two tags to the `<head>` of your base layout. For programmatic control use a `<script>` block — see below.

### Example 2: Recording link session lost on redirect
User says: "The recording link sends users to login and nothing happens" or "userplane-token gets dropped on redirect"
Carry `userplane-` prefixed URL parameters through your Astro middleware redirect. See the URL parameters section.

### Example 3: SSR initialization error
User says: "Userplane crashes on the server" or "window is not defined" or "initialize is being called during SSR"
Ensure `initialize()` is called inside a `<script>` block, not in the frontmatter (`---`). See the Troubleshooting section.

## CDN quick start

Add these two tags to the `<head>` of your base layout:

```astro
---
// src/layouts/Layout.astro
---

<html lang="en">
  <head>
    <meta name="userplane:workspace" content="YOUR_WORKSPACE_ID" />
    <script type="module" src="https://cdn.userplane.io/embed/script.js"></script>
  </head>
  <body>
    <slot />
  </body>
</html>
```

## npm SDK integration

### Install

```bash
npm install @userplane/sdk
```

### Script block initialization

Add a `<script>` block to your base layout. Astro `<script>` blocks are bundled and run in the browser — no `client:` directive or SSR guard needed.

```astro
---
// src/layouts/Layout.astro
---

<html lang="en">
  <head>
    <meta charset="utf-8" />
    <slot name="head" />
  </head>
  <body>
    <slot />

    <script>
      import { initialize } from '@userplane/sdk';

      initialize({
        workspaceId: import.meta.env.PUBLIC_USERPLANE_WORKSPACE_ID,
      });
    </script>
  </body>
</html>
```

Set the variable in your `.env` file:

```
PUBLIC_USERPLANE_WORKSPACE_ID=ws_abc123
```

`import.meta.env.PUBLIC_*` variables are inlined at build time by Vite. Access them inside `<script>` blocks (client-side), not in the frontmatter (server-side).

### URL parameters

For static deployments, Userplane reads URL parameters automatically. For SSR deployments with redirects, preserve `userplane-` prefixed parameters in middleware:

```typescript
// src/middleware.ts
import { defineMiddleware } from 'astro:middleware';

export const onRequest = defineMiddleware((context, next) => {
  if (!isAuthenticated(context) && requiresAuth(context.url.pathname)) {
    const loginUrl = new URL('/login', context.url.origin);

    for (const [key, value] of context.url.searchParams) {
      if (key.startsWith('userplane-')) {
        loginUrl.searchParams.set(key, value);
      }
    }

    return context.redirect(loginUrl.toString());
  }

  return next();
});
```

## Sensitive data

Add `data-userplane-blur` to any element you want blurred in recordings. See the `userplane-sensitive-data` skill for the full reference.

## Metadata

Call `set()` inside the `<script>` block after `initialize()`:

```astro
<script>
  import { initialize, set } from '@userplane/sdk';

  initialize({ workspaceId: import.meta.env.PUBLIC_USERPLANE_WORKSPACE_ID });
  set('environment', import.meta.env.MODE);
</script>
```

See the `userplane-metadata-sdk` skill for the full API.

## SSR safety

Astro `<script>` blocks are always client-only. The frontmatter (`---`) runs on the server or at build time.

| Context | Runs where | Can call `initialize()`? |
|---------|-----------|--------------------------|
| Frontmatter (`---`) | Server / build time | No |
| `<script>` block | Browser | Yes |
| Framework component with `client:load` | Browser | Yes |
| Framework component without `client:*` | Server only | No |

## Troubleshooting

### SDK not initializing
Cause: The `PUBLIC_USERPLANE_WORKSPACE_ID` environment variable is missing or not prefixed with `PUBLIC_`.
Solution: Verify the variable is set in `.env` with the `PUBLIC_` prefix and restart the dev server.

### Recording link session lost on redirect
Cause: Astro middleware redirects users without preserving `userplane-` query params.
Solution: Carry all `userplane-` prefixed params through the redirect in `src/middleware.ts`. See the URL parameters section.

### SSR initialization error
Cause: `initialize()` is called in the frontmatter (`---`) block, which runs on the server or at build time.
Solution: Move `initialize()` into a `<script>` block, which always runs client-side.

## Environment variables

| Variable | Description |
|----------|-------------|
| `PUBLIC_USERPLANE_WORKSPACE_ID` | Your Userplane workspace ID |
