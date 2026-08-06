# Draw

A freeform sketchpad backed by [Excalidraw](https://excalidraw.com), built as a JustFiles app.

Frames double as slides: `orderFrames` sorts them by reading order rather than z-order, so a
board can be presented without rearranging it.

## Develop

```sh
pnpm install
pnpm dev          # vite dev server with an in-memory kernel host
```

`pnpm dev` boots a local host that emulates the real runtime, so the app runs standalone —
no JustFiles install required.

## Check

```sh
pnpm typecheck
pnpm lint
pnpm test
pnpm format       # writes
```

## Build

```sh
pnpm build
```

Emits the app bundle into `dist/`:

| File | What it is |
| --- | --- |
| `manifest.json` | id, name, version, icon, declared capabilities |
| `app.js` | the headless reducer, run in a Worker |
| `gui.js` | the browser GUI, self-contained (React and CSS inlined) |
| `icon.png` | copied from `public/` |

`manifest.json`'s `version` is read from `package.json`, so the package version is the single
place a release version is set.
