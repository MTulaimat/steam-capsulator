# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A **single-file, fully client-side** web tool that generates every Steam store/library capsule image from one background + optional logo. Everything lives in `index.html` — markup, CSS, and a single inline `<script>`. There is **no build step, no package.json, no dependencies to install** (JSZip is loaded from a CDN). The exact capsule dimensions come from Steam's asset guidelines (the current "doubled" sizes, e.g. Header 920×430, Main 1232×706, Library Hero 3840×1240).

## Running / building / checking

- **Run**: open `index.html` in a browser. No server needed. (Don't auto-launch a browser unless asked.)
- **No tests / no linter.** To sanity-check the inline JS after edits, extract the last `<script>` block and run `node --check`:
  ```bash
  node -e "const fs=require('fs');const h=fs.readFileSync('index.html','utf8');const m=[...h.matchAll(/<script>([\s\S]*?)<\/script>/g)];fs.writeFileSync('.__c.js',m[m.length-1][1]);require('child_process').execSync('node --check .__c.js',{stdio:'inherit'});fs.unlinkSync('.__c.js');console.log('OK')"
  ```
- **Deploy**: GitHub Pages serves `main` branch root at https://mtulaimat.github.io/steam-capsulator/. Any `git push` to `main` auto-redeploys (~1 min).

## Architecture (the inline `<script>`)

Plain vanilla JS, organized into labeled `/* ---------- ... ---------- */` sections. The important cross-cutting pieces:

- **`CAPSULES`** — array of the 9 asset definitions (`id, name, w, h, group, file, logo, ...`). This is the single source of truth for sizes/filenames. Per-capsule behavior is driven by flags:
  - `logo: 'on' | 'off' | 'forbidden'` — `'off'` defaults the logo off (Page Background); `'forbidden'` bans it entirely (Library Hero — Steam disallows logos there).
  - `transparent: true` + `logoOnly: true` — Library Logo: ignores the background, composites the logo on a transparent canvas, and is **always exported as PNG** regardless of the PNG/JPG toggle.
  - `safe: {w,h}` — draws a "safe area" guide overlay (Library Hero's 860×380).

- **`ST`** — per-capsule runtime state keyed by capsule id: `bg`/`logo` transforms (`{scale, x, y, init}`), `includeLogo`, background `fx`, `logoFx`, and per-capsule image overrides `bgImg`/`logoImg`. **All editing state is per-capsule** — capsules are framed/styled independently.

- **`IMG`** = the shared `{bg, logo}` uploads. **Never read `IMG.bg`/`IMG.logo` directly inside per-capsule logic.** Always resolve through `bgOf(c)` / `logoOf(c)`, which return the per-capsule override if set, else the shared image. `anyImages()` is the "is there anything to show/export" check.

- **`render(c, g, sf, forExport)` is the single rendering source of truth.** It is called by three consumers: the live editor (`drawStage`, sf = display scale), per-capsule export (`renderToCanvas`, sf = 1), and the preview-grid thumbnails (small sf). Any visual change must therefore be expressed in terms of `sf` and respect `forExport`:
  - Coordinates are in **capsule pixel space**; multiply by `sf` to draw. Transforms store `x,y` as the image's top-left in capsule px and `scale` as an absolute multiplier on the image's natural size.
  - Editor-only chrome (selection outlines, handles, safe-area guides) must be gated behind `!forExport` so it never appears in exports. Selection outlines are drawn in `drawStage`/`drawSelection`, not in `render`.
  - Background effects use `ctx.filter` (blur scales by `sf`, with an edge "bleed" so blur doesn't fringe the frame) plus overlay passes (tint/darken/bottom-shade/vignette).
  - `drawLogoWithFx` composites the logo with shadow/outline/glow. Shadow/glow use the off-canvas `shadowOffset` trick (park the source far off-screen via `OFF` so only its shadow lands on the canvas); outline is built from a cached colored `silhouette()` ring.

- **Backgrounds are clamped to always cover the frame** (`clampBg`) — panning/zooming can't expose empty edges (Steam capsule art must be full-bleed). Call `clampBg(c)` after any background move/scale.

- **Interaction**: pointer events with multi-touch. `ptrs` Map tracks pointers; one pointer = click-to-select (hit-test the logo via `pointHitsLogo`, else background) + drag; two pointers = pinch-to-zoom about the midpoint. Wheel zoom on desktop.

- **UI flow**: `refreshAll()` is the central re-render — rebuilds the sidebar (`buildSidebar`) and the **context-sensitive** right panel (`buildControls`, which shows background settings when the bg layer is active and logo settings when the logo layer is active), then `drawStage()`. The controls panel is rebuilt from a template string on every change; event handlers are re-attached each rebuild.

- **Responsive**: a `@media (max-width:820px)` block restacks the layout (sidebar becomes a horizontal strip, controls move below the editor, page scrolls).

## Conventions specific to this repo

- **ALWAYS bump the version on every change.** Increment the `APP_VERSION` constant in `index.html` (shown as a subtle `vX.Y` label in the header) for *every* requested change before committing — no exceptions, including docs/config-only changes. Use `0.1` minor steps (e.g. `0.7` → `0.8`).
- **Commits/pushes only when asked.** Commit messages end with the `Co-Authored-By: Claude Opus 4.8 (1M context)` trailer.
- **PowerShell commit gotcha**: when committing multi-line messages, use a **single-quoted** here-string (`@'` … `'@`). Double quotes *inside* the message break the here-string and split the command — keep the body free of `"`.
