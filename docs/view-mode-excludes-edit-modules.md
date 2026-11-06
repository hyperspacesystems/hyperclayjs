# view-mode-excludes-edit-modules

A feature flag that skips edit-only modules when the page is in view mode, reducing bundle size for viewers.

## Usage

Add `view-mode-excludes-edit-modules` to your features list:

```html
<script type="module">
  await import('https://cdn.jsdelivr.net/npm/hyperclayjs@1/src/hyperclay.js?preset=standard&features=view-mode-excludes-edit-modules');
</script>
```

Or with custom features:

```html
<script type="module">
  await import('https://cdn.jsdelivr.net/npm/hyperclayjs@1/src/hyperclay.js?features=toast,dialogs,view-mode-excludes-edit-modules');
</script>
```

## How It Works

1. **Detects edit mode** using the same logic as hyperclay:
   - URL parameter `?editmode=true` (highest priority)
   - Cookie `isAdminOfCurrentResource` (fallback)

2. **In edit mode**: All requested modules load normally

3. **In view mode**: Modules marked as `isEditModeOnly` are automatically excluded

## Edit-Only Modules

These modules are skipped in view mode when this feature is enabled:

| Module | Description |
|--------|-------------|
| `save-core` | Basic save function |
| `save-system` | CMD+S, [trigger-save] button |
| `autosave` | Auto-save on DOM changes |
| `unsaved-warning` | Warn before leaving with unsaved changes |
| `save-toast` | Toast notifications for save events |
| `edit-mode-helpers` | [viewmode:disabled], [editmode:resource], [editmode:onclick] |
| `persist` | Persist input/select/textarea values to DOM |
| `snapshot` | DOM snapshots for save and sync |
| `sortable` | Drag-drop sorting |
| `onaftersave` | Run JS when save status changes |
| `cache-bust` | Cache-bust href/src attributes |
| `file-upload` | File upload with progress |
| `live-sync` | Real-time DOM sync across browsers |

## Size Savings

When using the `standard` preset:

| Mode | Modules Loaded | Approximate Size |
|------|----------------|------------------|
| Edit mode | All standard modules | ~50KB |
| View mode | Only view-compatible modules | ~15KB |

## When to Use

- **Production sites** where most visitors are viewers, not editors
- **Performance-critical pages** where every KB matters
- **Sites with heavy edit features** (live-sync, sortable, file-upload)

## When NOT to Use

- **Development** - you want full functionality regardless of mode
- **Admin-only pages** - all users are editors anyway
- **Simple sites** - if you're already using minimal preset, savings are small

## Platform Readiness Contract for Vendored Plugins

- **Ask the platform lifecycle, never namespace truthiness or capability presence:** vendored plugins must wait on `hyperclay.ready` / `hyperclay:ready` (defined in `src/core/exportToWindow.js` and `build/hyperclay.template.js`, which generates `src/hyperclay.js`) or `clay.ready` / `clay:ready`, never `mutation-ready` or a one-shot check at import time. In `hyperclayjs`, `loadModules` imports the rest wave concurrently via `Promise.all`, so flat vendor bundles finish before core modules like `edit-mode` or `beforeSave`; read a capability and, if missing, retry on the `ready` event and promise (and verify `hyper-undo` `on()` returns an unsubscribe function rather than `undefined`).
- **Preset protection and rollout order:** `view-mode-excludes-edit-modules` is what keeps `richclay` (`EDIT_MODE_ONLY`) out of view mode in standard presets, so hand-rolled `features=` lists without this sentinel bypass that guard, while the `cms` preset intentionally omits `edit-mode`. Always publish the `hyperclayjs` loader before or alongside re-vendoring the `richclay` bundle, because both clients execute vendored bundles.
