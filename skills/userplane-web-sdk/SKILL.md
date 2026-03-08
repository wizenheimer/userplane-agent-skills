---
name: userplane-web-sdk
description: Userplane Web SDK API reference for initialization, recorder control, and recording state. Use when user asks about SDK methods, initialize options, how to open the recorder programmatically, how to get a session ID, or how to query recording state.
license: MIT
metadata:
  author: Userplane
  version: 0.0.1
  category: reference
  tags:
    - userplane
    - screen-recording
---

# Userplane Web SDK

The `@userplane/sdk` package provides a JavaScript API for integrating Userplane into your web application. Use it to initialize the SDK, trigger recording flows, query recording state, and attach custom metadata.

## When to use

- Looking up Userplane SDK API methods
- Configuring initialization options (`workspaceId`, `openImmediately`, `captureEnabled`)
- Programmatically controlling the recorder (`open`, `unmount`)
- Querying recording state or session IDs
- Debugging bridge connection issues

## Examples

### Example 1: Look up an SDK method
User says: "What does getSessionId() return?" or "How do I check if the SDK is initialized?" or "What are the initialize options?"
Browse the sections below — Initialization, Recorder control, Recording state, and Tab management cover every exported function.

### Example 2: Open the recorder programmatically
User says: "How do I trigger a recording from a button?" or "Open the recorder without a recording link" or "Use openImmediately: false"
Initialize with `openImmediately: false`, then call `open()` from a click handler. See the Recorder control section.

### Example 3: Correlate Userplane sessions with analytics
User says: "How do I link a Userplane session to a support ticket?" or "Get the recording session ID" or "Pass session ID to Intercom"
Use `getSessionId()` and `getLinkId()` after the SDK initializes. See the Recording state section.

## Installation

```bash
npm install @userplane/sdk
```

## Initialization

### `initialize(options?)`

Initializes the Userplane SDK. Call once when your application loads. Subsequent calls are ignored.

```javascript
import { initialize } from '@userplane/sdk';

initialize({ workspaceId: 'ws_abc123' });
```

For the full `InitializeOptions` API reference including all properties and types, see `references/REFERENCE.md`.

#### Examples

```javascript
// Disable auto-open (manual control)
initialize({ workspaceId: 'ws_abc123', openImmediately: false });

// Open with a specific token
initialize({ workspaceId: 'ws_abc123', openImmediately: 'rec_token_xyz' });

// Multiple workspaces
initialize({ workspaceId: ['ws_abc123', 'ws_def456'] });
```

## Recorder control

### `open(token?)`

Opens the recording flow. Returns `true` if opened, `false` otherwise.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `token` | `string` | From URL params | Recording token. If omitted, uses token from current URL. |

```javascript
import { open } from '@userplane/sdk';
open();              // Use token from URL
open('rec_token');   // Use specific token
```

### `unmount()`

Closes and unmounts the recorder. If capture is enabled, the capture iframe is re-mounted after removal.

```javascript
import { unmount } from '@userplane/sdk';
unmount();
```

### `isInitialized()`

Returns `true` if the SDK has been initialized.

### `isRecorderMounted()`

Returns `true` if the recorder is currently mounted and visible.

## Recording state

### `getRecordingState()`

Returns: `RecordingState` — one of `'inactive'`, `'active'`, or `'retained'`

| State | Description |
|-------|-------------|
| `inactive` | No recording in progress |
| `active` | Recording currently in progress |
| `retained` | Recording session exists but not actively recording |

### `getRecordingContext()`

Returns: `RecordingContext | null` — contains `state`, `sessionInfo`, and `tabUnloadEnabled`. For the complete TypeScript interfaces, see `references/REFERENCE.md`.

### `getSessionId()`

Returns the current recording session ID, or `null` if no session is active.

### `getLinkId()`

Returns the recording link ID for the current session, or `null`.

## Connection state

### `isBridgeConnected()`

Returns `true` if the bridge connection between the SDK and the recorder iframe is established. Useful for debugging.

## Common patterns

- **Support button trigger** — Initialize with `openImmediately: false`, call `open()` from a button click handler using `isRecorderMounted()` to prevent double-open
- **Recording state indicator** — Poll `getRecordingState()` to show/hide a "recording in progress" badge in your UI
- **Analytics correlation** — Use `getSessionId()` and `getLinkId()` to send Userplane identifiers to your analytics tool alongside other events

For the complete API reference including `InitializeOptions`, TypeScript interfaces, URL parameters, HTML attributes, and tab management functions, see `references/REFERENCE.md`.
