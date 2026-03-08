---
name: userplane-sensitive-data
description: Configure Userplane blur to redact sensitive data in screen recordings. Use when user asks how to hide sensitive fields, blur PII or credit card inputs, set up privacy in recordings, or check compatibility with FullStory, Hotjar, or Sentry blur attributes.
license: MIT
metadata:
  author: Userplane
  version: 0.0.1
  category: reference
  tags:
    - userplane
    - screen-recording
---

# Userplane Sensitive Data Redaction

Configure blur to obscure sensitive content in screen recordings. Blur is adaptive — intensity adjusts based on content size. Blurring happens at capture time, so sensitive data is never recorded in a readable state.

## When to use

- Blurring sensitive elements (passwords, credit cards, PII) in recordings
- Setting up centralized blur rules via meta tags
- Checking third-party tool compatibility (FullStory, Hotjar, Sentry, etc.)
- Excluding specific elements from blur
- Enabling automatic sensitive field detection

## Examples

### Example 1: Blur a specific element
User says: "How do I hide a credit card field in recordings?" or "Blur a form section" or "Redact PII in screen recordings"
Add `data-userplane-blur` to the element or its container. See the Blur methods section.

### Example 2: Centralize blur rules
User says: "How do I blur multiple elements without touching each one?" or "Blur a third-party widget I don't control" or "Set blur rules in one place"
Use `<meta name="userplane:blur" content=".selector">` tags in the page `<head>`. See the Meta tags section.

### Example 3: Check third-party compatibility
User says: "I already use FullStory blur attributes, do I need to re-tag for Userplane?" or "Does Userplane respect Hotjar mask?" or "Sentry blur compatibility"
Userplane automatically recognizes attributes from 16 providers. See the Third-party tool compatibility section.

## Blur methods

Three ways to mark elements for blur. All can be mixed and matched.

### Data attribute

Add `data-userplane-blur` to any HTML element:

```html
<div data-userplane-blur>Blurred content</div>
<div data-userplane-blur="true">Also blurred</div>
```

Accepted truthy values: `true`, `1`, `yes`, or bare attribute.

### CSS class

Add the `userplane-mask` class:

```html
<div class="userplane-mask">Blurred content</div>
```

Useful for class-based workflows or toggling blur via JavaScript.

### Meta tags

Define blur targets using CSS selectors in `<meta>` tags:

```html
<head>
  <meta name="userplane:blur" content=".customer-info, .billing-section" />
  <meta name="userplane:blur" content="#credit-card-form" />
</head>
```

Multiple meta tags are supported. All selectors across all tags are collected and applied.

Best for: centralizing rules, blurring third-party content you don't control, CMS-managed targets.

### Choosing a method

| Method | Best for |
|--------|----------|
| `data-userplane-blur` | Specific elements you own and can modify directly |
| `.userplane-mask` class | Class-based workflows, toggling blur with JavaScript |
| `<meta>` tags | Centralizing rules, blurring third-party content, CMS-managed targets |

## Excluding elements from blur

Set the attribute to a falsy value to exclude an element, even if a parent would blur it:

```html
<div data-userplane-blur>
  <p>This is blurred.</p>
  <p data-userplane-blur="false">This is NOT blurred.</p>
</div>
```

Accepted falsy values: `false`, `0`, `no`. Unmask always takes precedence over mask.

## Automatic blur

When the **Hide sensitive fields** domain preference is enabled (in Workspace Settings > Domains), the SDK automatically detects and blurs:

- Password inputs (`type="password"`)
- Credit card fields (`autocomplete` values like `cc-number`, `cc-exp`, `cc-csc`)
- Other browser-identified sensitive input types

No HTML changes required.

## Third-party tool compatibility

The SDK recognizes privacy attributes from 16 providers. If you already tag elements for another tool, they are automatically blurred in Userplane recordings too — no re-tagging needed. Supported providers include FullStory, Hotjar, Sentry, OpenReplay, Heap, Amplitude, rrweb, LogRocket, Microsoft Clarity, and more.

Generic privacy attributes are also recognized: `[data-private]`, `.private`, `[data-sensitive]`, `.sensitive`.

For the complete mask and unmask selector tables across all 16 providers, see `references/REFERENCE.md`.

## Dynamic content

Blur automatically handles content added after initial load — client-side framework renders, lazy-loaded content, dynamically injected HTML. New elements matching any active blur selector are blurred as soon as they appear in the DOM.

## Common patterns

### Blur a form section

```html
<form>
  <div data-userplane-blur>
    <label>Card number</label>
    <input type="text" autocomplete="cc-number" />
    <label>Expiry</label>
    <input type="text" autocomplete="cc-exp" />
  </div>
  <div>
    <label>Shipping address</label>
    <input type="text" />
  </div>
</form>
```

### Blur with exceptions

```html
<div data-userplane-blur>
  <p>Account balance: $4,200.00</p>
  <p>Account type: <span data-userplane-blur="false">Business</span></p>
</div>
```

### Centralized blur via meta tags

```html
<head>
  <meta name="userplane:blur" content=".pii, .financial-data, .health-info" />
</head>
```

### React component

```jsx
function PatientRecord({ patient }) {
  return (
    <div>
      <h2>{patient.name}</h2>
      <div data-userplane-blur>
        <p>DOB: {patient.dateOfBirth}</p>
        <p>SSN: {patient.ssn}</p>
      </div>
      <p>Appointment: {patient.nextAppointment}</p>
    </div>
  );
}
```

## Verification

1. Add `data-userplane-blur` to a test element
2. Create a recording using a recording link on the same domain
3. Play back the recording and verify the element is obscured

Blurred elements appear with a visual blur effect — position and size visible, content unreadable.
