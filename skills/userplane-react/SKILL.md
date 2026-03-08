---
name: userplane-react
description: Integrate Userplane screen recording into a React (Vite) application. Use when user asks to set up screen recording in React, add Userplane to a Vite app, or configure a UserplaneProvider component.
license: MIT
compatibility: React 18+, Vite 5+
metadata:
  author: Userplane
  version: 0.0.1
  category: integration
  tags:
    - userplane
    - screen-recording
    - react
---

# Userplane React Integration

Install Userplane screen recording in a React application built with Vite using a provider component and `useEffect`.

## When to use

- Adding Userplane to a React (Vite) project
- Setting up screen recording in a client-side rendered React app
- Configuring the Userplane provider component for React

## Examples

### Example 1: Basic setup
User says: "How do I add Userplane to my React app?" or "Set up screen recording in React" or "Integrate Userplane with Vite React"
For CDN-only: add the two script tags to `index.html`. For programmatic control use the UserplaneProvider pattern — see below.

### Example 2: Recording link session lost on redirect
User says: "The recording link sends users to login and nothing happens" or "userplane-token gets dropped on redirect"
Carry `userplane-` prefixed URL parameters through your auth redirect. See the URL parameters section.

## CDN quick start

Add these two tags to the `<head>` of your `index.html`:

```html
<meta name="userplane:workspace" content="YOUR_WORKSPACE_ID" />
<script type="module" src="https://cdn.userplane.io/embed/script.js"></script>
```

Copy the snippet with your workspace ID pre-filled from **Workspace Settings > Domains** in the Userplane dashboard.

## npm SDK integration

Use the npm SDK when you need programmatic control — triggering recordings from a button, attaching user metadata, or reading recording state.

### Install

```bash
npm install @userplane/sdk
```

### Provider component

Create a `UserplaneProvider` component and wrap your app with it in `main.tsx`:

```tsx
// src/providers/UserplaneProvider.tsx
import { useEffect } from 'react';
import { initialize } from '@userplane/sdk';

export function UserplaneProvider({ children }: { children: React.ReactNode }) {
  useEffect(() => {
    initialize({
      workspaceId: import.meta.env.VITE_USERPLANE_WORKSPACE_ID,
    });
  }, []);

  return <>{children}</>;
}
```

```tsx
// src/main.tsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import { UserplaneProvider } from './providers/UserplaneProvider';
import App from './App';

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <UserplaneProvider>
      <App />
    </UserplaneProvider>
  </StrictMode>
);
```

Vite React apps are pure client-side rendered. A static import at the top of the file works — no dynamic `import()` or SSR guards are needed. `useEffect` ensures `initialize()` runs once after mount.

### URL parameters

When a customer opens a recording link, Userplane appends `userplane-token` and `userplane-action` to the URL. If your app uses a router that strips unknown search parameters (e.g. a protected route redirect), carry the `userplane-` prefixed params through:

```typescript
const url = new URL(window.location.href);
const loginUrl = new URL('/login', window.location.origin);

for (const [key, value] of url.searchParams) {
  if (key.startsWith('userplane-')) {
    loginUrl.searchParams.set(key, value);
  }
}

window.location.href = loginUrl.toString();
```

## Sensitive data

Add `data-userplane-blur` to any element you want blurred in recordings. See the `userplane-sensitive-data` skill for the full reference.

## Metadata

Call `set()` after `initialize()` to attach user context to recordings:

```tsx
import { useEffect } from 'react';
import { initialize, set } from '@userplane/sdk';

export function UserplaneProvider({ children }: { children: React.ReactNode }) {
  useEffect(() => {
    initialize({ workspaceId: import.meta.env.VITE_USERPLANE_WORKSPACE_ID });
    set('environment', 'production');
  }, []);

  return <>{children}</>;
}
```

See the `userplane-metadata-sdk` skill for the full API.

## SSR safety

Not applicable — Vite React apps are client-side only. No SSR guards needed.

## Troubleshooting

### SDK not initializing
Cause: The `VITE_USERPLANE_WORKSPACE_ID` environment variable is missing or not prefixed with `VITE_`.
Solution: Verify the variable is set in `.env` with the `VITE_` prefix and restart the dev server.

### Recording link session lost on redirect
Cause: Your router strips unknown search parameters when redirecting unauthenticated users.
Solution: Carry all `userplane-` prefixed query params through the redirect. See the URL parameters section.

### StrictMode double-invoke
Cause: React StrictMode invokes `useEffect` twice in development to detect side effects.
Solution: No changes needed — `initialize()` ignores subsequent calls and behaves correctly under StrictMode.

## Environment variables

| Variable | Description |
|----------|-------------|
| `VITE_USERPLANE_WORKSPACE_ID` | Your Userplane workspace ID |
