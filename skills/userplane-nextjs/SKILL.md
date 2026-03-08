---
name: userplane-nextjs
description: Integrate Userplane screen recording into a Next.js App Router application. Use when user asks to set up screen recording in Next.js, add Userplane to an App Router project, or configure SSR-safe initialization with a client provider.
license: MIT
compatibility: Next.js 13+ (App Router)
metadata:
  author: Userplane
  version: 0.0.1
  category: integration
  tags:
    - userplane
    - screen-recording
    - nextjs
---

# Userplane Next.js Integration

Install Userplane screen recording in a Next.js App Router application using a client provider component.

## When to use

- Adding Userplane to a Next.js (App Router) project
- Setting up screen recording with SSR-safe initialization in Next.js
- Configuring Userplane with Next.js middleware for URL parameter preservation

## Examples

### Example 1: Basic setup
User says: "How do I add Userplane to my Next.js app?" or "Set up screen recording in Next.js" or "Integrate Userplane with App Router"
For CDN-only: add the two tags to the root layout `<head>`. For programmatic control use the client provider pattern — see below.

### Example 2: Recording link session lost on redirect
User says: "The recording link sends users to login and nothing happens" or "userplane-token gets dropped on redirect"
Carry `userplane-` prefixed URL parameters through your Next.js middleware redirect. See the URL parameters section.

### Example 3: SSR initialization error
User says: "Userplane crashes on the server" or "window is not defined" or "initialize is being called during SSR"
Ensure `initialize()` is called inside `useEffect` within a `'use client'` component, not in a Server Component. See the Troubleshooting section.

## CDN quick start

Add these two tags to the `<head>` of your root layout:

```html
<meta name="userplane:workspace" content="YOUR_WORKSPACE_ID" />
<script type="module" src="https://cdn.userplane.io/embed/script.js"></script>
```

## npm SDK integration

### Install

```bash
npm install @userplane/sdk
```

### Client provider

Create a client component that initializes the SDK and mount it in your root layout:

```tsx
// app/providers/userplane-provider.tsx
'use client';

import { useEffect } from 'react';

export function UserplaneProvider({ children }: { children: React.ReactNode }) {
  useEffect(() => {
    import('@userplane/sdk').then(({ initialize }) => {
      initialize({
        workspaceId: process.env.NEXT_PUBLIC_USERPLANE_WORKSPACE_ID!,
      });
    });
  }, []);

  return <>{children}</>;
}
```

```tsx
// app/layout.tsx
import { UserplaneProvider } from './providers/userplane-provider';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <UserplaneProvider>{children}</UserplaneProvider>
      </body>
    </html>
  );
}
```

The `'use client'` boundary is required because `useEffect` only runs in the browser. The dynamic `import()` is a bundle-size optimization — a static `import` at the top also works since the SDK is SSR-safe.

### URL parameters

Next.js middleware and route guards may redirect users before the SDK reads URL parameters. Preserve `userplane-` prefixed parameters through redirects:

```typescript
// middleware.ts
import { NextRequest, NextResponse } from 'next/server';

export function middleware(request: NextRequest) {
  const { searchParams } = request.nextUrl;
  const isAuthenticated = checkAuth(request);

  if (!isAuthenticated) {
    const loginUrl = new URL('/login', request.url);

    for (const [key, value] of searchParams) {
      if (key.startsWith('userplane-')) {
        loginUrl.searchParams.set(key, value);
      }
    }

    return NextResponse.redirect(loginUrl);
  }

  return NextResponse.next();
}
```

## Sensitive data

Add `data-userplane-blur` to any element you want blurred in recordings. See the `userplane-sensitive-data` skill for the full reference.

## Metadata

Call `set()` after `initialize()` to attach user context:

```tsx
'use client';

import { useEffect } from 'react';

export function UserplaneProvider({ children }: { children: React.ReactNode }) {
  useEffect(() => {
    import('@userplane/sdk').then(({ initialize, set }) => {
      initialize({ workspaceId: process.env.NEXT_PUBLIC_USERPLANE_WORKSPACE_ID! });
      set('environment', 'production');
    });
  }, []);

  return <>{children}</>;
}
```

See the `userplane-metadata-sdk` skill for the full API.

## SSR safety

`@userplane/sdk` is SSR-safe to import — it does not reference `window` or `document` at module evaluation time.

| Concern | Safe? | Notes |
|---------|-------|-------|
| Static `import` at top of `'use client'` file | Yes | Module is SSR-safe |
| Calling `initialize()` in `useEffect` | Yes | Runs browser-only |
| Calling `initialize()` in a Server Component | No | `window` is not available |
| Dynamic `import('@userplane/sdk')` | Yes | Bundle-size optimization only |

## Troubleshooting

### SDK not initializing
Cause: The `NEXT_PUBLIC_USERPLANE_WORKSPACE_ID` environment variable is missing or not prefixed with `NEXT_PUBLIC_`.
Solution: Verify the variable is set in `.env.local` with the `NEXT_PUBLIC_` prefix and restart the dev server.

### Recording link session lost on redirect
Cause: Next.js middleware redirects users before the SDK reads URL parameters.
Solution: Carry all `userplane-` prefixed query params through the redirect in `middleware.ts`. See the URL parameters section.

### SSR initialization error
Cause: `initialize()` is called in a Server Component or outside a `useEffect`, where `window` is unavailable.
Solution: Ensure the provider file has `'use client'` at the top and `initialize()` is called inside `useEffect`.

### Hydration mismatch
Cause: The provider component is missing the `'use client'` directive, causing a server/client render mismatch.
Solution: Add `'use client'` as the first line of `app/providers/userplane-provider.tsx`.

### StrictMode double-invoke
Cause: React StrictMode invokes `useEffect` twice in development.
Solution: No changes needed — `initialize()` ignores subsequent calls and behaves correctly under StrictMode.

## Environment variables

| Variable | Description |
|----------|-------------|
| `NEXT_PUBLIC_USERPLANE_WORKSPACE_ID` | Your Userplane workspace ID |
