# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A single-file GitHub Pages site (`index.html`) — a whimsical art app for an 8-year-old girl named Isla. No build step, no dependencies, no package manager. Open `index.html` directly in a browser to develop.

## Deployment

Deployed via GitHub Pages (static hosting). Push `index.html`, `photos/`, and `.gitignore` to the `main` branch. No CI, no build process.

## Architecture

Everything lives in `index.html` — inline CSS (`<style>`), HTML, and JavaScript (`<script>`). There are no external JS dependencies; only Google Fonts is loaded externally.

### Canvas system
- Drawing surface is an **SVG element** (`id="drawing-svg"`, `viewBox="0 0 1400 900"`)
- All drawn strokes and stamps are `<g>` or `<path>` elements inside `<g id="strokes">`
- The selection UI (`<g id="sel-ui" data-no-export="1">`) lives outside `#strokes` and is stripped from exported SVGs
- `history[]` array tracks all canvas elements for undo — both strokes and stamps
- Auto-save to `localStorage` key `'isla-art'` (saves `STROKES.innerHTML`)

### Tools
- `tool` variable: `'pen' | 'eraser' | 'emoji' | 'shape'`
- All pointer events go through a single `onDown/onMove/onUp` handler on the SVG element
- Stamp mode (emoji/shape): `pointerdown` checks `e.target` — resize handle → resize, existing stamp → select, empty → place new stamp
- Selection state: `selectedStamp = { el, cx, cy, size }` + `resizingHandle` string

### Stamp system
- Each stamp is a `<g data-stamp="1" data-type="emoji|shape" data-cx data-cy data-size>`
- Emoji stamps: contain a `<text>` element; size = font-size directly
- Shape stamps: contain inline SVG paths scaled via `translate(cx,cy) scale(size/100)`; shapes defined in `SHAPE_DEFS` object as template functions `(color) => svgString`
- Resize: drag any corner handle → `new size = 2 * max(|dx|, |dy|)` from center

### AI generation
- Uses **Pollinations.ai** (free, no API key): `GET https://image.pollinations.ai/prompt/{encoded-prompt}?model=flux&width=1024&height=768&nologo=true&enhance=true&seed={n}`
- SVG canvas is converted to a PNG preview via an offscreen `<canvas>` element (display only — not sent to the API)
- The sketch is shown as visual context; the text prompt + style modifier drives generation

### Photos
- `photos/isla1.png` – `isla4.png` — shown on hover over the page title (`<h1>`)
- Hover logic uses `mouseenter/mousemove/mouseleave` on the `<h1>`, renders into `#isla-popup`

### Export
- `saveDrawing()` clones the SVG, removes `[data-no-export]` elements, serialises with `XMLSerializer`, downloads as `.svg`
- SVG paths are proper vectors — suitable for STL conversion via external tools
