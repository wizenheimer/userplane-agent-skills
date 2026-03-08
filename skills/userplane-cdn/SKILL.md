---
name: userplane-cdn
description: Install and configure the Userplane CDN embed script. Use when user asks how to install Userplane without npm, add the embed script to a site, configure CSP for Userplane, handle redirects that strip URL parameters, or check browser support.
license: MIT
metadata:
  author: Userplane
  version: 0.0.1
  category: reference
  tags:
    - userplane
    - screen-recording
---

# Userplane CDN Installation

Install and configure the Userplane embed script on your site. The script enables screen recording and captures browser events (console logs, network requests) during recording sessions.

## When to use

- Installing Userplane via CDN script tags
- Configuring Content-Security-Policy for Userplane
- Handling redirects that strip `userplane-` URL parameters
- Checking browser support
- Setting up multiple workspaces on one domain

## Examples

### Example 1: Basic installation
User says: "How do I add the Userplane script to my site?" or "Install Userplane without npm" or "Add the CDN embed script"
Add the `<meta>` and `<script>` tags to the `<head>` of every page. See the Embed script section.

### Example 2: Configure CSP for Userplane
User says: "The recorder iframe is being blocked" or "CSP is blocking Userplane" or "Content-Security-Policy for Userplane"
Add `*.userplane.io` to both `frame-src` and `script-src` in your CSP. See the Content-Security-Policy section.

### Example 3: Recording link session lost on redirect
User says: "Recording link params disappear after login redirect" or "userplane-token gets dropped" or "preserve URL params through redirects"
Manually carry `userplane-` prefixed parameters through your redirect logic. See the Handling redirects section.

## Embed script

Add to the `<head>` of every page where you want recordings:

```html
<meta name="userplane:workspace" content="YOUR_WORKSPACE_ID" />
<script type="module" src="https://cdn.userplane.io/embed/script.js"></script>
```

Replace `YOUR_WORKSPACE_ID` with your workspace ID from **Workspace Settings > Domains**.

## Script placement

Place the `<script>` tag as early as possible in `<head>`. The script can only capture console logs and network requests that occur after it initializes — placing it early gives the most complete context.

**Do not** use `async` or `defer` attributes, or load it via a lazy `import()`. These defer execution and may cause the script to miss early events during page load.

Userplane aggressively caches script assets. Only critical code runs upfront; the rest is deferred internally.

## Domains and subdomains

Install the script on each domain you want to record. A script on the root domain generally captures events from subdomains too (e.g., `example.com` sees `app.example.com`).

**Safari caveat:** Events are only captured if the recording page and script are on the same subdomain. Install the script on each subdomain separately if Safari support is important.

## Handling redirects

When a customer opens a recording link, Userplane appends query parameters:

- `userplane-token` — recording session token (required)
- `userplane-action` — action type, e.g. `recording` (required)
- `userplane-workspace` — workspace ID

If your app redirects users (e.g., to a login page), these params must survive the redirect:

```javascript
const url = new URL(window.location.href);
const userplaneParams = new URLSearchParams();

for (const [key, value] of url.searchParams) {
  if (key.startsWith('userplane-')) {
    userplaneParams.set(key, value);
  }
}

const loginUrl = new URL('/login', window.location.origin);
loginUrl.searchParams.set('redirect', `${url.pathname}?${userplaneParams.toString()}`);

window.location.href = loginUrl.toString();
```

## Content-Security-Policy

If your site sets CSP directives, allow `*.userplane.io` as both a script source and frame source:

```html
<meta
  http-equiv="Content-Security-Policy"
  content="frame-src 'self' *.userplane.io; script-src 'self' *.userplane.io;"
/>
```

Or via HTTP header:

```
Content-Security-Policy: frame-src 'self' *.userplane.io; script-src 'self' *.userplane.io;
```

Without this, the recorder frame and capture script will be blocked.

## Iframe limitations

If the script is installed inside an iframe, it only captures events from within that iframe. Top-level page events are not captured. For full coverage, install the script on the top-level page.

## Browser support

| Browser | Support |
|---------|---------|
| Chrome | Fully supported (including Incognito) |
| Firefox | Fully supported (including Private Browsing) |
| Safari | Supported in standard windows. Private Browsing not currently supported. |
| Edge | Fully supported (including InPrivate) |

## Multiple workspaces

Associate a single domain with multiple workspaces using multiple meta tags:

```html
<meta name="userplane:workspace" content="ws_abc123" />
<meta name="userplane:workspace" content="ws_def456" />
```

Or a comma-separated list:

```html
<meta name="userplane:workspace" content="ws_abc123,ws_def456" />
```

## Troubleshooting

### Script not initializing
Cause: The `<script>` tag uses `async` or `defer`, deferring execution past early page events.
Solution: Remove `async`/`defer`. Use `<script type="module" src="...">` without async/defer attributes.

### Recorder iframe blocked
Cause: A Content-Security-Policy header is missing the `frame-src *.userplane.io` directive.
Solution: Add `frame-src 'self' *.userplane.io` and `script-src 'self' *.userplane.io` to your CSP.

### Recording link session lost on redirect
Cause: An auth redirect strips `userplane-` URL parameters before they can be read by the SDK.
Solution: Preserve all `userplane-` prefixed params through the redirect. See the Handling redirects section.

### Safari cross-subdomain limitation
Cause: On Safari, the SDK on `example.com` does not capture events from `app.example.com`.
Solution: Install the embed script separately on each subdomain if Safari support is required.

## Verification

1. Open your site in a browser
2. Open the developer console — look for Userplane initialization messages
3. Create a test recording link in the Userplane dashboard and open it on your site to confirm the end-to-end flow
