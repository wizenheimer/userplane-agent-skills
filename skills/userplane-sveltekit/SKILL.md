---
name: userplane-sveltekit
description: Integrate Userplane screen recording into a SvelteKit application. Use when user asks to set up screen recording in SvelteKit, add Userplane to a Svelte app, or initialize Userplane safely with onMount.
license: MIT
compatibility: SvelteKit 2+, Svelte 4+
metadata:
  author: Userplane
  version: 0.0.1
  category: integration
  tags:
    - userplane
    - screen-recording
    - sveltekit
---

# Userplane SvelteKit Integration

Install Userplane screen recording in a SvelteKit application using `onMount` in the root layout.

## When to use

- Adding Userplane to a SvelteKit project
- Setting up screen recording via `onMount` in `+layout.svelte`
- Configuring Userplane with SvelteKit server hooks for URL parameter preservation

## Examples

### Example 1: Basic setup
User says: "How do I add Userplane to my SvelteKit app?" or "Set up screen recording in SvelteKit" or "Integrate Userplane with Svelte"
For CDN-only: add the two script tags to `src/app.html`. For programmatic control use `onMount` in `+layout.svelte` — see below.

### Example 2: Recording link session lost on redirect
User says: "The recording link sends users to login and nothing happens" or "userplane-token gets dropped on redirect"
Carry `userplane-` prefixed URL parameters through your SvelteKit server hook redirect. See the URL parameters section.

### Example 3: SSR initialization error
User says: "Userplane crashes on the server" or "window is not defined" or "initialize is being called during SSR"
Ensure `initialize()` is called inside `onMount`, not in a `load` function. See the Troubleshooting section.

## CDN quick start

Add these two tags to the `<head>` in `src/app.html`:

```html
<meta name="userplane:workspace" content="YOUR_WORKSPACE_ID" />
<script type="module" src="https://cdn.userplane.io/embed/script.js"></script>
```

## npm SDK integration

### Install

```bash
npm install @userplane/sdk
```

### Root layout initialization

Initialize the SDK in `src/routes/+layout.svelte` using `onMount`. `onMount` only runs in the browser.

```svelte
<!-- src/routes/+layout.svelte -->
<script>
  import { onMount } from 'svelte';
  import { PUBLIC_USERPLANE_WORKSPACE_ID } from '$env/static/public';

  onMount(async () => {
    const { initialize } = await import('@userplane/sdk');
    initialize({ workspaceId: PUBLIC_USERPLANE_WORKSPACE_ID });
  });
</script>

<slot />
```

Set the variable in your `.env` file:

```
PUBLIC_USERPLANE_WORKSPACE_ID=ws_abc123
```

SvelteKit's `$env/static/public` module only exposes variables prefixed with `PUBLIC_`, inlined at build time.

### URL parameters

If your app uses a SvelteKit server hook that redirects unauthenticated users, preserve `userplane-` prefixed parameters:

```typescript
// src/hooks.server.ts
import { redirect } from '@sveltejs/kit';
import type { Handle } from '@sveltejs/kit';

export const handle: Handle = async ({ event, resolve }) => {
  if (requiresAuth(event) && !isAuthenticated(event)) {
    const url = event.url;
    const loginUrl = new URL('/login', url.origin);

    for (const [key, value] of url.searchParams) {
      if (key.startsWith('userplane-')) {
        loginUrl.searchParams.set(key, value);
      }
    }

    redirect(302, loginUrl.toString());
  }

  return resolve(event);
};
```

## Sensitive data

Add `data-userplane-blur` to any element you want blurred in recordings. See the `userplane-sensitive-data` skill for the full reference.

## Metadata

Call `set()` inside `onMount` after `initialize()`:

```svelte
<script>
  import { onMount } from 'svelte';
  import { PUBLIC_USERPLANE_WORKSPACE_ID } from '$env/static/public';

  onMount(async () => {
    const { initialize, set } = await import('@userplane/sdk');
    initialize({ workspaceId: PUBLIC_USERPLANE_WORKSPACE_ID });
    set('environment', 'production');
  });
</script>

<slot />
```

See the `userplane-metadata-sdk` skill for the full API.

## SSR safety

`@userplane/sdk` is SSR-safe to import. The `initialize()` call must run client-side, which `onMount` guarantees.

| Concern | Safe? | Notes |
|---------|-------|-------|
| Static `import` at top of `<script>` | Yes | Module is SSR-safe |
| Calling `initialize()` inside `onMount` | Yes | Runs browser-only |
| Dynamic `import('@userplane/sdk')` in `onMount` | Yes | Bundle-size optimization only |
| Calling `initialize()` in a `load` function | No | `load` runs on the server |
| `export const ssr = false` | Not needed | `onMount` is sufficient |

## Troubleshooting

### SDK not initializing
Cause: The `PUBLIC_USERPLANE_WORKSPACE_ID` environment variable is missing or imported from the wrong path.
Solution: Import from `$env/static/public` (not `$env/static/private`) and ensure the variable has the `PUBLIC_` prefix in `.env`.

### Recording link session lost on redirect
Cause: SvelteKit server hooks redirect users without preserving `userplane-` query params.
Solution: Carry all `userplane-` prefixed params through the redirect in `src/hooks.server.ts`. See the URL parameters section.

### SSR initialization error
Cause: `initialize()` is called in a `load` function, which runs on the server for SSR pages.
Solution: Move `initialize()` into `onMount` in `+layout.svelte`. `onMount` only runs in the browser.

## Environment variables

| Variable | Description |
|----------|-------------|
| `PUBLIC_USERPLANE_WORKSPACE_ID` | Your Userplane workspace ID |
