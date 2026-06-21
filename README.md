# Steam Capsulator

A single-file, browser-based tool for generating **Steam store & library capsule images** from one background image and an optional logo. It crops/fits the background into every required size, places the logo, lets you refine each one (drag, zoom, numeric position, anchor presets), and exports them individually or as a ZIP with size-matched filenames.

**[▶ Open the app](https://mtulaimat.github.io/steam-capsulator/)**

## Features

- Upload one **background** + optional **logo** — auto-fitted into all capsule sizes at once.
- Per-capsule refinement: drag to move, scroll to zoom, X/Y/scale fields, Fill/Fit/Reset, and 6 logo anchor presets.
- **No-logo mode** for backgrounds that already include the logo (global or per-capsule).
- Special handling from Steam's guidelines: Library Hero blocks logos and shows the 860×380 safe area; Library Logo is logo-only on transparent PNG; Page Background defaults to no logo.
- Export as **PNG or JPG** (transparent Library Logo stays PNG), single download or **download-all ZIP**.
- Filenames match the role + size, e.g. `header_capsule_920x430.png`.

## Asset sizes

| Asset | Size | Group |
|---|---|---|
| Header Capsule | 920×430 | Store |
| Small Capsule | 462×174 | Store |
| Main Capsule | 1232×706 | Store |
| Vertical Capsule | 748×896 | Store |
| Page Background | 1438×810 | Store |
| Library Capsule | 600×900 | Library |
| Library Header | 920×430 | Library |
| Library Hero | 3840×1240 | Library |
| Library Logo | 1280×720 | Library |

## Usage

It's a single static `index.html` — just open it in a browser, or use the hosted GitHub Pages link above. No build step, no dependencies to install (JSZip is loaded from a CDN).
