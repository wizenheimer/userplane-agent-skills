---
name: userplane-nuxt
description: Integrate Userplane screen recording into a Nuxt 3 application. Use when user asks to set up screen recording in Nuxt, add Userplane to a Nuxt 3 project, or create a browser-only Userplane plugin.
license: MIT
compatibility: Nuxt 3+
metadata:
  author: Userplane
  version: 0.0.1
  category: integration
  tags:
    - userplane
    - screen-recording
    - nuxt
---

# Userplane Nuxt 3 Integration

Install Userplane screen recording in a Nuxt 3 application using a browser-only client plugin.

## When to use

- Adding Userplane to a Nuxt 3 project
- Setting up screen recording with a `.client.ts` Nuxt plugin
- Configuring Userplane with Nuxt middleware for URL parameter preservation

## Examples

### Example 1: Basic setup
User says: "How do I add Userplane to my Nuxt app?" or "Set up screen recording in Nuxt 3" or "Integrate Userplane with Nuxt"
For CDN-only: configure the script tags in `nuxt.config.ts`. For programmatic control use the `.client.ts` plugin pattern — see below.

### Example 2: Recording link session lost on redirect
User says: "The recording link sends users to login and nothing happens" or "userplane-token gets dropped on redirect"
Carry `userplane-` prefixed URL parameters through your Nuxt middleware redirect. See the URL parameters section.

### Example 3: SSR initialization error
User says: "Userplane crashes on the server" or "window is not defined" or "initialize is being called during SSR"
Ensure your plugin file has the `.client.ts` suffix so Nuxt excludes it from the server bundle. See the Troubleshooting section.

## CDN quick start

Add the tags in `nuxt.config.ts`:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  app: {
    head: {
      meta: [{ name: 'userplane:workspace', content: 'YOUR_WORKSPACE_ID' }],
      script: [{ type: 'module', src: 'https://cdn.userplane.io/embed/script.js' }],
    },
  },
});
```

## npm SDK integration

### Install

```bash
npm install @userplane/sdk
```

### Client plugin

Create a plugin file with the `.client.ts` suffix. Nuxt only runs `.client.ts` plugins in the browser — no additional SSR guard is needed.

```typescript
// plugins/userplane.client.ts
export default defineNuxtPlugin(async () => {
  const { initialize } = await import('@userplane/sdk');
  const config = useRuntimeConfig();

  initialize({
    workspaceId: config.public.userplaneWorkspaceId,
  });
});
```

Expose the workspace ID through Nuxt's runtime config:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  runtimeConfig: {
    public: {
      userplaneWorkspaceId: '',
    },
  },
});
```

Set the value in your `.env` file:

```
NUXT_PUBLIC_USERPLANE_WORKSPACE_ID=ws_abc123
```

Nuxt automatically maps `NUXT_PUBLIC_*` environment variables to `runtimeConfig.public.*`.

### URL parameters

If your app uses Nuxt middleware that redirects unauthenticated users, preserve `userplane-` prefixed parameters:

```typescript
// middleware/auth.ts
export default defineNuxtRouteMiddleware((to) => {
  if (!isAuthenticated()) {
    const query: Record<string, string> = {};

    for (const [key, value] of Object.entries(to.query)) {
      if (key.startsWith('userplane-') && typeof value === 'string') {
        query[key] = value;
      }
    }

    return navigateTo({ path: '/login', query });
  }
});
```

## Sensitive data

Add `data-userplane-blur` to any element you want blurred in recordings. See the `userplane-sensitive-data` skill for the full reference.

## Metadata

Call `set()` inside the plugin after `initialize()`:

```typescript
// plugins/userplane.client.ts
export default defineNuxtPlugin(async () => {
  const { initialize, set } = await import('@userplane/sdk');
  const config = useRuntimeConfig();

  initialize({ workspaceId: config.public.userplaneWorkspaceId });
  set('environment', 'production');
});
```

See the `userplane-metadata-sdk` skill for the full API.

## SSR safety

The `.client.ts` suffix tells Nuxt to exclude this plugin from the server bundle entirely. No `process.client` guards needed.

| Concern | Safe? | Notes |
|---------|-------|-------|
| `.client.ts` plugin suffix | Yes | Nuxt excludes file from SSR bundle |
| Static `import` inside `.client.ts` | Yes | File never runs on server |
| Dynamic `import('@userplane/sdk')` | Yes | Bundle-size optimization only |
| Calling `initialize()` in a server plugin | No | `window` is not available |

## Troubleshooting

### SDK not initializing
Cause: The plugin file is missing the `.client.ts` suffix, causing it to run on the server where `window` is unavailable.
Solution: Rename the plugin file to `plugins/userplane.client.ts`.

### Recording link session lost on redirect
Cause: Nuxt middleware redirects users without preserving `userplane-` query params.
Solution: Carry all `userplane-` prefixed params through `navigateTo()`. See the URL parameters section.

### SSR initialization error
Cause: `initialize()` is called in a server plugin (without `.client.ts` suffix).
Solution: Rename the plugin to `userplane.client.ts` — Nuxt automatically excludes `.client.ts` files from the server bundle.

### runtimeConfig.public undefined
Cause: `userplaneWorkspaceId` is not declared under `runtimeConfig.public` in `nuxt.config.ts`.
Solution: Add the key to `runtimeConfig.public` in `nuxt.config.ts` and set `NUXT_PUBLIC_USERPLANE_WORKSPACE_ID` in `.env`.

## Environment variables

| Variable | Description |
|----------|-------------|
| `NUXT_PUBLIC_USERPLANE_WORKSPACE_ID` | Your Userplane workspace ID |
