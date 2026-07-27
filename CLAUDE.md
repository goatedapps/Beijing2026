# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repository contains a single self-contained static HTML file, `index.html`, which renders an interactive day-by-day itinerary for a 6-day-5-night personal trip to Beijing (Oct 17–22, 2026). There is no build system, package manager, or test suite — the entire app is one file with inline `<style>` and `<script>` blocks.

## Running / Previewing

There is no build step. Open `index.html` directly in a browser, or serve it locally, e.g.:

```
python -m http.server 8000
```

then visit `http://localhost:8000/index.html`.

## Architecture

The page is driven by a single in-script data array, `DAYS` (in the `<script>` block near the bottom of `index.html`), where each entry represents one day of the trip:

- `stops`: an ordered array mixing two item shapes:
  - Location stops: `{ time, title, coord: [lat, lng], text, photo? }` — rendered as numbered timeline entries with an optional Wikimedia photo.
  - Transit segments: `{ transit: { type: "car"|"bus"|"walk", label } }` — rendered as pill-shaped connectors between stops using icons from the `ICONS` map.

All DOM rendering is done by imperatively building elements in JS (no framework): the `DAYS.forEach` loop in the render section constructs a `<details class="day">` accordion per day, containing a text timeline (left) and a Leaflet map (right).

Key structural points to preserve when editing:

- **Adding/editing a day or stop**: just edit the `DAYS` array — rendering, numbering, map markers, and route lines are all derived automatically from it. Stop numbering (`stopIndex`) skips `transit` entries and increments only for real stops.
- **Maps are lazy-initialized**: each day's Leaflet map (`initDayMap`) is only created the first time that day's `<details>` is expanded (tracked via `mapInited`), for performance. Day 1 is force-opened and its map initialized on page load.
- **Map styling**: the sepia/vintage look is a CSS `filter` on `.leaflet-map`, not a custom tile layer — the tile source is standard OpenStreetMap.
- **Photo fallback**: `<img onerror="imgFallback(this)">` swaps to a `.photo-fallback` placeholder if a Wikimedia image fails to load; this is a global function attached to `window`.
- **External dependencies** are all via CDN `<link>`/`<script>` tags (Google Fonts, Leaflet CSS/JS from unpkg) — no local assets or node_modules.

## Content notes

- Coordinates, transit durations, and flight details are trip-planning estimates for personal reference, not live data (see footer disclaimer).
- Landmark photos are hotlinked from Wikimedia Commons and credited beneath each image.
