# Design: V2 — Preview Panel

**Date:** 2026-06-05  
**Status:** Approved

## Overview

Replace the read-only output textarea with a rendered HTML preview panel + collapsible raw HTML block. Add two copy buttons: "Copy formatted" (rich text for Telegram Desktop) and "Copy HTML" (raw string for n8n bot API).

## Architecture

Modify `index.html` in place. No new files. The `convert()` function and left panel are untouched. Only the right pane and buttons change.

## Right Panel Layout

```
┌─ div#preview ─────────────────────────┐
│  Rendered HTML — bold, italic, links  │
└───────────────────────────────────────┘
▶ HTML  ← <details> element, collapsed by default
  <textarea id="output" readonly>
  raw HTML string here
  </textarea>
```

- `div#preview` — styled to match the old textarea (same border, padding, height, font). `overflow-y: auto`. Links are rendered and clickable.
- `<details><summary>HTML</summary>` — native browser collapsible. Contains `<textarea id="output" readonly>` for raw HTML.

## Buttons

Three buttons below the editor: `[Convert]` `[Copy formatted]` `[Copy HTML]`

- **Convert** — calls `convert(input.value)`, writes result to `preview.innerHTML` and `output.value`. If input is empty, clears both.
- **Copy formatted** — if preview is empty, return. Selects all content in `div#preview` via `window.getSelection()` + `document.createRange()`, calls `document.execCommand('copy')`, then clears the selection. On success, shows "Copied!" on the button for 2 seconds.
- **Copy HTML** — if `output.value` is empty, return. Copies `output.value` string via `navigator.clipboard.writeText()` with `execCommand` fallback. On success, shows "Copied!" for 2 seconds.

## CSS Changes

- `#preview` — same visual style as old `textarea[readonly]`: `background: #fafafa`, `border: 1px solid #ddd`, `border-radius: 8px`, `padding: 0.75rem`, `height: 320px`, `overflow-y: auto`, `line-height: 1.5`
- `#preview` font: system sans-serif (not monospace — rendered text reads better in a proportional font)
- `details` — `margin-top: 0.5rem`, `font-size: 0.8rem`
- `details textarea` — `width: 100%`, `height: 120px`, `font-family: monospace`, `font-size: 0.8rem`
- Three buttons keep the same style as v1; "Copy formatted" and "Copy HTML" both use the gray style

## What's Unchanged

- Left pane (input textarea)
- `convert()` function and all regex rules
- `runTests()` self-tests
- Responsive breakpoint (@media 768px)
