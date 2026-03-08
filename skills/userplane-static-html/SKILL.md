---
name: userplane-static-html
description: Integrate Userplane screen recording on a static HTML site with no build step. Use when user asks to add screen recording to a plain HTML page, embed Userplane via CDN on a static site, or set up session recording without npm or a framework.
license: MIT
compatibility: Any modern browser
metadata:
  author: Userplane
  version: 0.0.1
  category: integration
  tags:
    - userplane
    - screen-recording
    - static-html
---

# Userplane Static HTML Integration

Install Userplane screen recording on a static HTML site with no build step using the CDN embed script.

## When to use

- Adding Userplane to a static HTML site with no build system
- Setting up screen recording via CDN script tags only
- Simple integration that requires no npm packages

## Examples

### Example 1: Basic setup
User says: "How do I add Userplane to my HTML page?" or "Set up screen recording on a static site" or "Add Userplane without npm"
Add the two script tags to the `<head>` of every page. See the Installation section below.

### Example 2: Recording link session lost on redirect
User says: "The recording link sends users to login and nothing happens" or "userplane-token gets dropped on redirect"
Carry `userplane-` prefixed URL parameters through your auth redirect. See the URL parameters section.

## Installation

Add these two tags to the `<head>` of every page where you want recordings to work:

```html
<head>
  <meta name="userplane:workspace" content="YOUR_WORKSPACE_ID" />
  <script type="module" src="https://cdn.userplane.io/embed/script.js"></script>
</head>
```

Replace `YOUR_WORKSPACE_ID` with your workspace ID from **Workspace Settings > Domains**.

Place the tags as early as possible in `<head>`. The script can only capture console logs and network requests that occur after it initializes.

## Sensitive data

Add `data-userplane-blur` to any element you want blurred:

```html
<div data-userplane-blur>
  <input type="text" placeholder="Credit card number" />
</div>
```

To blur all inputs across the page, use a meta tag:

```html
<meta name="userplane:blur" content="inputs" />
```

Supported `content` values:

| Value | What is blurred |
|-------|----------------|
| `inputs` | All `<input>`, `<textarea>`, and `<select>` elements |
| `images` | All `<img>` elements |
| `all` | The entire page |

To exclude a specific element from blur:

```html
<div data-userplane-blur>
  <p>This is blurred.</p>
  <p data-userplane-blur="false">This is not blurred.</p>
</div>
```

See the `userplane-sensitive-data` skill for the full reference including CSS class blurring and third-party compatibility.

## URL parameters

When a customer opens a recording link, Userplane appends `userplane-token`, `userplane-action`, and `userplane-workspace` to the URL. For static HTML sites, these are read automatically by the embed script — no additional configuration needed.

If your site redirects users before they reach the recorded page, carry `userplane-` prefixed parameters through the redirect. See the `userplane-cdn` skill for a code example.

## Troubleshooting

### Script not initializing
Cause: The `<script>` tag uses `async` or `defer`, deferring execution past the point where early page events are captured.
Solution: Remove `async`/`defer` from the script tag. Use `<script type="module" src="...">` without those attributes.

### Recording link session lost on redirect
Cause: A server-side or JavaScript redirect strips `userplane-` URL parameters before the page loads.
Solution: Carry all `userplane-` prefixed params through the redirect. See `userplane-cdn` for a code example.

### CSP blocking recorder iframe
Cause: A Content-Security-Policy header is blocking the Userplane recorder iframe.
Solution: Add `*.userplane.io` to both `frame-src` and `script-src` in your CSP. See the `userplane-cdn` skill for exact syntax.
