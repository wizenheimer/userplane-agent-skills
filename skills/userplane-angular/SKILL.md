---
name: userplane-angular
description: Integrate Userplane screen recording into an Angular application. Use when user asks to set up screen recording in Angular, add Userplane to an Angular project, or initialize Userplane via APP_INITIALIZER.
license: MIT
compatibility: Angular 17+
metadata:
  author: Userplane
  version: 0.0.1
  category: integration
  tags:
    - userplane
    - screen-recording
    - angular
---

# Userplane Angular Integration

Install Userplane screen recording in an Angular application using `APP_INITIALIZER`.

## When to use

- Adding Userplane to an Angular project
- Setting up screen recording via Angular's `APP_INITIALIZER`
- Configuring Userplane with Angular route guards for URL parameter preservation

## Examples

### Example 1: Basic setup
User says: "How do I add Userplane to my Angular app?" or "Set up screen recording in Angular" or "Integrate Userplane with Angular"
For CDN-only: add the two tags to `src/index.html`. For programmatic control use `APP_INITIALIZER` — see below.

### Example 2: Recording link session lost on redirect
User says: "The recording link sends users to login and nothing happens" or "userplane-token gets dropped on redirect"
Carry `userplane-` prefixed URL parameters through your Angular route guard redirect. See the URL parameters section.

### Example 3: SSR initialization error
User says: "Userplane crashes on the server" or "window is not defined" or "APP_INITIALIZER runs on Angular Universal"
Wrap `initialize()` with `isPlatformBrowser` when using Angular Universal. See the Troubleshooting section.

## CDN quick start

Add these two tags to the `<head>` of `src/index.html`:

```html
<meta name="userplane:workspace" content="YOUR_WORKSPACE_ID" />
<script type="module" src="https://cdn.userplane.io/embed/script.js"></script>
```

## npm SDK integration

### Install

```bash
npm install @userplane/sdk
```

### APP_INITIALIZER

Create an initializer factory and register it with `APP_INITIALIZER` in your app config. Angular calls `APP_INITIALIZER` providers during bootstrap, before the first component renders.

```typescript
// src/app/initializers/userplane.initializer.ts
import { initialize } from '@userplane/sdk';
import { environment } from '../../environments/environment';

export function provideUserplane() {
  return () => {
    initialize({ workspaceId: environment.userplaneWorkspaceId });
  };
}
```

```typescript
// src/app/app.config.ts
import { ApplicationConfig, APP_INITIALIZER } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideUserplane } from './initializers/userplane.initializer';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    {
      provide: APP_INITIALIZER,
      useFactory: provideUserplane,
      multi: true,
    },
  ],
};
```

Store the workspace ID in your environment file:

```typescript
// src/environments/environment.ts
export const environment = {
  production: false,
  userplaneWorkspaceId: 'ws_abc123',
};
```

```typescript
// src/environments/environment.prod.ts
export const environment = {
  production: true,
  userplaneWorkspaceId: 'ws_abc123',
};
```

Angular CLI replaces `environment.ts` with `environment.prod.ts` for production builds automatically.

### URL parameters

If your app uses route guards that redirect unauthenticated users, preserve `userplane-` prefixed parameters through the redirect:

```typescript
// src/app/guards/auth.guard.ts
import { inject } from '@angular/core';
import { CanActivateFn, Router, ActivatedRouteSnapshot } from '@angular/router';

export const authGuard: CanActivateFn = (route: ActivatedRouteSnapshot) => {
  const router = inject(Router);
  const authService = inject(AuthService);

  if (!authService.isAuthenticated()) {
    const queryParams: Record<string, string> = {};

    for (const [key, value] of Object.entries(route.queryParams)) {
      if (key.startsWith('userplane-') && typeof value === 'string') {
        queryParams[key] = value;
      }
    }

    router.navigate(['/login'], { queryParams });
    return false;
  }

  return true;
};
```

## Sensitive data

Add `data-userplane-blur` to any element you want blurred in recordings. See the `userplane-sensitive-data` skill for the full reference.

## Metadata

Call `set()` inside the initializer factory after `initialize()`:

```typescript
import { initialize, set } from '@userplane/sdk';
import { environment } from '../../environments/environment';

export function provideUserplane() {
  return () => {
    initialize({ workspaceId: environment.userplaneWorkspaceId });
    set('environment', environment.production ? 'production' : 'development');
  };
}
```

See the `userplane-metadata-sdk` skill for the full API.

## SSR safety

Not applicable for standard Angular CLI apps. If using Angular Universal (SSR), ensure `initialize()` is only called in browser context (check `isPlatformBrowser`).

## Troubleshooting

### SDK not initializing
Cause: The workspace ID is not set in `environment.ts` or the wrong environment file is used for the build.
Solution: Verify `environment.userplaneWorkspaceId` is set in both `environment.ts` and `environment.prod.ts`.

### Recording link session lost on redirect
Cause: Angular route guards redirect unauthenticated users without preserving `userplane-` query params.
Solution: Carry all `userplane-` prefixed params through the router navigation in the auth guard. See the URL parameters section.

### APP_INITIALIZER runs on Angular Universal server
Cause: `initialize()` is called unconditionally inside `APP_INITIALIZER`, but Angular Universal also runs initializers on the server where `window` is unavailable.
Solution: Inject `PLATFORM_ID` and wrap `initialize()` with `isPlatformBrowser(platformId)`.

## Environment variables

| Variable | Description |
|----------|-------------|
| `USERPLANE_WORKSPACE_ID` | Your Userplane workspace ID (set in `environment.ts`) |
