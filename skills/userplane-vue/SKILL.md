---
name: userplane-vue
description: Integrate Userplane screen recording into a Vue 3 (Vite) application. Use when user asks to set up screen recording in Vue, add Userplane to a Vue 3 project, or configure a Vue plugin for Userplane.
license: MIT
compatibility: Vue 3+, Vite 5+
metadata:
  author: Userplane
  version: 0.0.1
  category: integration
  tags:
    - userplane
    - screen-recording
    - vue
---

# Userplane Vue 3 Integration

Install Userplane screen recording in a Vue 3 application built with Vite using a Vue plugin.

## When to use

- Adding Userplane to a Vue 3 (Vite) project
- Setting up screen recording via a Vue plugin
- Configuring Userplane with Vue Router navigation guards

## Examples

### Example 1: Basic setup
User says: "How do I add Userplane to my Vue app?" or "Set up screen recording in Vue 3" or "Integrate Userplane with Vue"
For CDN-only: add the two script tags to `index.html`. For programmatic control use the Vue plugin pattern — see below.

### Example 2: Recording link session lost on redirect
User says: "The recording link sends users to login and nothing happens" or "userplane-token gets dropped on redirect"
Carry `userplane-` prefixed URL parameters through your Vue Router navigation guard. See the URL parameters section.

## CDN quick start

Add these two tags to the `<head>` of your `index.html`:

```html
<meta name="userplane:workspace" content="YOUR_WORKSPACE_ID" />
<script type="module" src="https://cdn.userplane.io/embed/script.js"></script>
```

## npm SDK integration

### Install

```bash
npm install @userplane/sdk
```

### Vue plugin

Create a Vue plugin and register it in `main.ts` via `app.use()`:

```typescript
// src/plugins/userplane.ts
import type { App } from 'vue';
import { initialize } from '@userplane/sdk';

interface UserplanePluginOptions {
  workspaceId: string;
}

export const userplanePlugin = {
  install(_app: App, options: UserplanePluginOptions) {
    initialize({ workspaceId: options.workspaceId });
  },
};
```

```typescript
// src/main.ts
import { createApp } from 'vue';
import App from './App.vue';
import { userplanePlugin } from './plugins/userplane';

const app = createApp(App);

app.use(userplanePlugin, {
  workspaceId: import.meta.env.VITE_USERPLANE_WORKSPACE_ID,
});

app.mount('#app');
```

The `install` function runs after the app is created and before mounting — a browser-only context in a Vite Vue app. No SSR guards are needed.

### URL parameters

If Vue Router redirects users before the SDK reads URL parameters, preserve `userplane-` prefixed params through the redirect:

```typescript
// router/index.ts
router.beforeEach((to, _from, next) => {
  if (requiresAuth(to) && !isAuthenticated()) {
    const loginQuery: Record<string, string> = { redirect: to.fullPath };

    for (const [key, value] of Object.entries(to.query)) {
      if (key.startsWith('userplane-') && typeof value === 'string') {
        loginQuery[key] = value;
      }
    }

    next({ path: '/login', query: loginQuery });
  } else {
    next();
  }
});
```

## Sensitive data

Add `data-userplane-blur` to any element you want blurred in recordings. See the `userplane-sensitive-data` skill for the full reference.

## Metadata

Call `set()` inside the plugin after `initialize()`:

```typescript
import type { App } from 'vue';
import { initialize, set } from '@userplane/sdk';

export const userplanePlugin = {
  install(_app: App, options: { workspaceId: string }) {
    initialize({ workspaceId: options.workspaceId });
    set('environment', 'production');
  },
};
```

See the `userplane-metadata-sdk` skill for the full API.

## SSR safety

Not applicable — Vite Vue apps are client-side only. No SSR guards needed.

## Troubleshooting

### SDK not initializing
Cause: The `VITE_USERPLANE_WORKSPACE_ID` environment variable is missing or not prefixed with `VITE_`.
Solution: Verify the variable is set in `.env` with the `VITE_` prefix and restart the dev server.

### Recording link session lost on redirect
Cause: Vue Router strips unknown query parameters when redirecting unauthenticated users.
Solution: Carry all `userplane-` prefixed query params through the redirect in the navigation guard. See the URL parameters section.

### Plugin not initializing
Cause: `app.use(userplanePlugin)` is called after `app.mount('#app')`.
Solution: Call `app.use()` before `app.mount()` to ensure the plugin's `install` function runs in the browser context.

## Environment variables

| Variable | Description |
|----------|-------------|
| `VITE_USERPLANE_WORKSPACE_ID` | Your Userplane workspace ID |
