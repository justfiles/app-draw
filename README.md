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

`gui.js` is large (~9.8 MB, ~2.7 MB gzipped) because it carries Excalidraw. That is an inert
static asset, not executable Worker code — `app.js`, the only file that becomes a Worker, is
under 12 KB.

### Known limitation: fonts load from esm.sh

Excalidraw resolves its fonts against `window.EXCALIDRAW_ASSET_PATH` and falls back to
`https://esm.sh/@excalidraw/excalidraw@<version>/dist/prod/` when it is unset. It is unset here,
so text rendering makes a third-party network request at runtime.

Bundling the fonts instead needs the app to know its own asset base URL, and it currently cannot:
the host loads `gui.js` from a Blob URL, so `import.meta.url` is a `blob:` URL and Excalidraw's
relative `./fonts/…` paths resolve against the host origin rather than the app's. Fixing it means
`@justfiles/app` passing a base URL into the GUI's mount context. Tracked upstream.

## Release

Bump `version` in `package.json` and merge to `main`. That is the whole ritual.

`.github/workflows/release.yml` notices a version with no matching tag, runs the checks, builds,
zips the contents of `dist/` into `draw.zip`, attaches a build-provenance attestation, and cuts
the GitHub release at `v<version>`. Every other push to `main` is a no-op, so the workflow is safe
to run on every merge.

The asset name is not a convention — the catalog importer takes the sole `*.zip` on a release, so
it can be whatever describes the app.

Because the tag is derived from `package.json` rather than pushed by hand, the release tag and
`manifest.json`'s version cannot drift apart.

