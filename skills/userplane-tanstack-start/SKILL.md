---
name: userplane-tanstack-start
description: Integrate Userplane screen recording into a TanStack Start application. Use when user asks to set up screen recording in TanStack Start, add Userplane to a TanStack project, or fix a notFoundError caused by missing Userplane URL search params.
license: MIT
compatibility: TanStack Start 1+, TanStack Router 1+
metadata:
  author: Userplane
  version: 0.0.1
  category: integration
  tags:
    - userplane
    - screen-recording
    - tanstack-start
---

# Userplane TanStack Start Integration

Install Userplane screen recording in a TanStack Start application using a client-side provider component.

## When to use

- Adding Userplane to a TanStack Start project
- Setting up screen recording with TanStack Router search param validation
- Configuring Userplane URL parameters in TanStack Router's Zod schema

## Examples

### Example 1: Basic setup
User says: "How do I add Userplane to my TanStack Start app?" or "Set up screen recording in TanStack Start" or "Integrate Userplane with TanStack Router"
For CDN-only: add the tags to the root route's `head` config. For programmatic control use the provider component pattern — see below.

### Example 2: Router throws notFoundError
User says: "TanStack Router throws notFoundError when I open a recording link" or "userplane-token causes a router error" or "router rejects unknown search params"
Add `userplane-token`, `userplane-action`, and `userplane-workspace` to your root route's Zod search schema. See the URL parameters section.

### Example 3: SSR initialization error
User says: "Userplane crashes on the server" or "window is not defined" or "initialize is being called during SSR"
Ensure `initialize()` is called inside `useEffect`, not in a loader function. See the Troubleshooting section.

## CDN quick start

Add tags to the root route's `head` configuration:

```tsx
// app/routes/__root.tsx
import { createRootRoute, HeadContent, Outlet } from '@tanstack/react-router';

export const Route = createRootRoute({
  head: () => ({
    meta: [{ name: 'userplane:workspace', content: 'YOUR_WORKSPACE_ID' }],
    scripts: [{ type: 'module', src: 'https://cdn.userplane.io/embed/script.js' }],
  }),
  component: RootComponent,
});

function RootComponent() {
  return (
    <>
      <HeadContent />
      <Outlet />
    </>
  );
}
```

## npm SDK integration

### Install

```bash
npm install @userplane/sdk
```

### Provider component

Create a provider and render it inside the root route component:

```tsx
// app/components/UserplaneProvider.tsx
'use client';

import { useEffect } from 'react';

export function UserplaneProvider({ children }: { children: React.ReactNode }) {
  useEffect(() => {
    import('@userplane/sdk').then(({ initialize }) => {
      initialize({
        workspaceId: import.meta.env.VITE_USERPLANE_WORKSPACE_ID,
      });
    });
  }, []);

  return <>{children}</>;
}
```

```tsx
// app/routes/__root.tsx
import { UserplaneProvider } from '../components/UserplaneProvider';

function RootComponent() {
  return (
    <UserplaneProvider>
      <HeadContent />
      <Outlet />
    </UserplaneProvider>
  );
}
```

### URL parameters — IMPORTANT

TanStack Router validates search parameters against a Zod schema. If a route schema does not include `userplane-token`, `userplane-action`, and `userplane-workspace`, the router throws a `notFoundError` when Userplane appends those params.

Add the Userplane params to your root route's search param schema:

```typescript
// app/routes/__root.tsx
import { createRootRoute } from '@tanstack/react-router';
import { z } from 'zod';

const searchSchema = z.object({
  'userplane-token': z.string().optional(),
  'userplane-action': z.string().optional(),
  'userplane-workspace': z.string().optional(),
});

export const Route = createRootRoute({
  validateSearch: searchSchema,
  component: RootComponent,
});
```

Defining this on the root route means all child routes inherit it. You can also use `z.record(z.string())` for a catch-all.

If your app redirects unauthenticated users, preserve the params through the redirect:

```typescript
import { redirect } from '@tanstack/react-router';

export const Route = createFileRoute('/dashboard')({
  beforeLoad: ({ context, search }) => {
    if (!context.auth.isAuthenticated) {
      throw redirect({
        to: '/login',
        search: {
          'userplane-token': search['userplane-token'],
          'userplane-action': search['userplane-action'],
          'userplane-workspace': search['userplane-workspace'],
        },
      });
    }
  },
});
```

## Sensitive data

Add `data-userplane-blur` to any element you want blurred in recordings. See the `userplane-sensitive-data` skill for the full reference.

## Metadata

Call `set()` inside the `useEffect` after `initialize()`:

```tsx
useEffect(() => {
  import('@userplane/sdk').then(({ initialize, set }) => {
    initialize({ workspaceId: import.meta.env.VITE_USERPLANE_WORKSPACE_ID });
    set('environment', import.meta.env.MODE);
  });
}, []);
```

See the `userplane-metadata-sdk` skill for the full API.

## SSR safety

TanStack Start renders on the server before hydrating in the browser. `@userplane/sdk` is SSR-safe to import.

| Concern | Safe? | Notes |
|---------|-------|-------|
| Static `import` at top of file | Yes | Module is SSR-safe |
| Calling `initialize()` in `useEffect` | Yes | Runs browser-only |
| Dynamic `import('@userplane/sdk')` in `useEffect` | Yes | Bundle-size optimization only |
| Calling `initialize()` in a loader | No | Loaders run on the server |
| Missing search params in route schema | No | Router throws `notFoundError` |

## Troubleshooting

### SDK not initializing
Cause: The `VITE_USERPLANE_WORKSPACE_ID` environment variable is missing or not prefixed with `VITE_`.
Solution: Verify the variable is set in `.env` with the `VITE_` prefix and restart the dev server.

### Router throws notFoundError on recording link
Cause: TanStack Router validates search params against a Zod schema; unrecognized `userplane-*` params cause a `notFoundError`.
Solution: Add `userplane-token`, `userplane-action`, and `userplane-workspace` to the root route's `validateSearch` schema. See the URL parameters section.

### SSR initialization error
Cause: `initialize()` is called in a loader function, which runs on the server.
Solution: Move `initialize()` into `useEffect` in the provider component. See the npm SDK integration section.

## Environment variables

| Variable | Description |
|----------|-------------|
| `VITE_USERPLANE_WORKSPACE_ID` | Your Userplane workspace ID |
