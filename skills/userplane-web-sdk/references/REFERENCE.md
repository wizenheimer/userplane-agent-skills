# Userplane Web SDK — Full API Reference

## InitializeOptions

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `workspaceId` | `string \| string[]` | From `<meta name="userplane:workspace">` | Workspace ID. Single ID or array for multi-workspace. |
| `openImmediately` | `boolean \| string` | `true` | `true`: auto-open on recording URL params. `false`: don't auto-open. `string`: a recording token to open immediately. |
| `captureEnabled` | `boolean` | `true` | Mount background capture iframe for console logs and network requests. |
| `parseUserplaneData` | `(href: string) => SerializableUserplaneData \| null` | Built-in parser | Custom parser for `userplane-action` and `userplane-token` from URL. Use for non-standard URL routing. |
| `applyUserplaneData` | `(data: SerializableUserplaneData) => string` | Built-in handler | Custom handler for applying Userplane data to URL during cross-tab sync. |

## TypeScript interfaces

### RecordingContext

```typescript
interface RecordingContext {
  state: RecordingState;
  sessionInfo: RecordingSessionInfo | null;
  tabUnloadEnabled: boolean;
}
```

### RecordingSessionInfo

```typescript
interface RecordingSessionInfo {
  workspaceId: string;
  sessionId: string;
  linkId?: string;
}
```

### Serializable

```typescript
type Serializable =
  | string
  | number
  | boolean
  | null
  | Serializable[]
  | { [key: string]: Serializable };
```

## URL parameters

When a customer opens a recording link, these query parameters are appended to the URL:

| Parameter | Description |
|-----------|-------------|
| `userplane-token` | The recording session token |
| `userplane-action` | Action type (`recording`, `verify`, or `capture`) |
| `userplane-workspace` | The workspace ID |

The SDK parses these automatically on initialization. All three parameters must survive any auth redirects — see each framework skill for redirect-preservation examples.

## HTML attributes and meta tags

### Blur attributes

```html
<div data-userplane-blur>Blurred content</div>
<div data-userplane-blur="false">Excluded from blur</div>
```

Also supports `.userplane-mask` CSS class and `<meta name="userplane:blur">` tags. See the `userplane-sensitive-data` skill for the full reference.

### Workspace meta tag

If `workspaceId` is not passed in `initialize()`, the SDK reads it from:

```html
<meta name="userplane:workspace" content="ws_abc123" />
```

## Tab management

### `getTabRef()`

Returns the unique tab reference for this browser tab (persisted in `sessionStorage`). Used internally for cross-tab recording state.

### `clearTabRef()`

Clears the tab reference from `sessionStorage`. A new reference is generated on next initialization.
