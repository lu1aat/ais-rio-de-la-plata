# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Static HTML site published on GitHub Pages at `lu1aat.github.io/ais-rio-de-la-plata`. No build system, no dependencies to install — all pages are self-contained HTML files.

**Subject matter:** AIS (Automatic Identification System) maritime data analysis in the Río de la Plata / South Atlantic. Content language is Spanish.

## Local preview

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

Serving via HTTP (not `file://`) is required because posts use `fetch()` to load local CSV and GeoJSON files.

## Site structure

```
index.html                        # Homepage — list of all posts
posts/
  NN-slug/
    index.html                    # Post content (self-contained)
    *.csv / *.geojson / *.png     # Data and media for that post
```

Each post directory is numbered sequentially (`01-`, `02-`, …). The homepage `index.html` must be updated manually when adding a new post.

## Design system

All styles live in `<style>` blocks inside each HTML file — there is no shared stylesheet. Both the homepage and post pages share the same conventions:

- **Background:** `#0a0f1e` (dark navy)
- **Accent / links:** `#7dd3fc` (sky blue)
- **Body text:** `#b8cfe8` / `#c8d8f0`
- **Muted / labels:** `#4a7a9b`
- **Borders:** `#1e3a5f`
- **Fonts:** `Inter` (body), `JetBrains Mono` (code, labels, mono elements) — loaded from Google Fonts

Recurring CSS components used in posts: `.toc`, `.callout`, `.debate-box`, `.gallery`, `.img-block`, `.post-footer`, `.pill` (homepage).

Float layout pattern for in-text images: `.img-block.half-right` (floats image to the right at 50% width) paired with `.clearfix` on the containing element.

Table styles (`table`, `th`, `td`) are defined inline per post. First column is typically a monospace identifier in accent color. Status indicator classes used in data tables: `.imo-bad` (red `#e74c3c`), `.imo-good` (green `#2ecc71`), `.imo-note` (muted, small, block-displayed sub-line).

## Data-table posts (no map)

Posts like `02-ais-anomalias-imo` present findings as HTML tables with embedded tweet screenshots, no JavaScript. Data is offered as a downloadable CSV via a `.callout` link — the CSV is **not** fetched by JS; it's purely a static download.

Downloadable CSV schema for anomaly data: `nombre,imo_en_ais,imo_real,bandera,categoria,observacion`. The `categoria` field uses snake_case values (`imo_corto`, `imo_default`, `imo_adulterado`, `imo_cero_extra`). `imo_real` is empty when the correct IMO could not be determined.

## Interactive maps (posts with Leaflet)

Post `01-sw-empress` uses [Leaflet 1.9.4](https://leafletjs.com/) via CDN. The map renders a ship track from a local `positions.csv` and overlays GeoJSON boundary layers. The CSV format expected by the track parser:

```
timestamp,speed,course,lat,lng
```

Rows are newest-first in the file; the script reverses them before rendering. Speed is used to color the track:
- `< 1 kn` → `#555` (anchored)
- `1–4 kn` → `#e74c3c` (slow / working)
- `4–8 kn` → `#f39c12`
- `> 8 kn` → `#2ecc71` (transit)

## Deployment

Push to `main` — GitHub Pages deploys automatically. No CI pipeline.
