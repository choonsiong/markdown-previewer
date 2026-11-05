# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page markdown previewer: the whole app is `index.html` (CSS in one `<style>`, JS in one `'use strict'` IIFE), plus `favicon.svg`. There is no build step, package manager, linter or test suite. Libraries come from cdnjs `<script>` tags, so the page needs network access to work.

## Running and verifying

- Run: open `index.html` in a browser. Nothing to install. For automated browser checks, serve the folder over HTTP (`python3 -m http.server`), because Playwright blocks `file:` URLs.
- There are no automated tests. Check a change in a browser: type or paste markdown, load the sample, upload or drag in a `.md` file, export a PDF, and look at it in both light and dark system themes and at a phone width (820px or less).
- Deployment is GitHub Pages serving `index.html` from the repository root of the default branch (see README), so `index.html` must stay at the root, with `favicon.svg` beside it (it is linked by a relative path).
- Releases: after a pull request merges, tag the default branch's head `vX.Y.Z` and push the tag. `.github/workflows/release.yml` then publishes the "vX.Y.Z release" GitHub release with generated notes, and fails if the tag is not on the default branch. A release builds and deploys nothing: GitHub Pages serves the default branch as soon as a pull request merges.

## How the page works

**Render pipeline.** Each `input` on the textarea waits 120 ms (`scheduleRender`), then runs `render()`: `marked.parse` with a custom renderer, then `DOMPurify.sanitize`, then every link gets `target="_blank" rel="noopener"`. The same debounce saves a draft to `localStorage` under `mdp:draft` and `mdp:name`. On load, a saved draft wins over the built-in `SAMPLE`.

**marked is pinned to 12.x on purpose.** The custom renderer (`code`, `table`, `checkbox`, `listitem`) uses marked 12's positional-argument signatures. marked 13+ passes token objects instead, so upgrading marked means rewriting those four functions. Code blocks are highlighted inside `renderer.code` with highlight.js (auto-detected when the language is missing or unknown), and tables are wrapped in `.table-wrap` so they scroll sideways.

**Theming lives in CSS custom properties** on `:root`. The dark palette appears twice and the two copies must stay identical: once under `@media (prefers-color-scheme: dark)` as `:root:not([data-theme="light"])`, and once as `:root[data-theme="dark"]`. Nothing sets `data-theme` yet; the second block is a hook for a manual toggle. highlight.js colours come from the `--hl-*` variables, not from a highlight.js stylesheet.

**PDF export.** `buildPdfBlob()` copies the preview's HTML into a detached `.sheet` (always light, with its own hard-coded colours) and renders it to an A4 blob with html2pdf.js. html2pdf clones that element into its own fixed, A4-wide container and has html2canvas capture the container, which makes the export fragile in two ways:
- `.sheet` must stay in normal flow. Absolute positioning (an old off-screen trick) or a fixed width collapses or clips the capture, and the PDF comes out blank.
- html2canvas crops at the position it measures in the real window, so it must lay the copy out exactly where it sits there. Keep `windowWidth` unset and `scrollX`/`scrollY` at 0, or the capture shifts sideways.

**PDF page breaks are planned in script.** html2pdf captures one tall canvas and cuts it every page height, which slices lines of text in half. Instead, `buildPdfBlob()` breaks html2pdf's chain into steps:
1. After `toContainer()`, `measurePdfUnits()` records where every text line and every block that shouldn't split sits. Blocks that shouldn't split are headings, code, tables, quotes and images; one taller than a page is split by rows or lines instead.
2. After `toCanvas()`, `planPdfCuts()` cuts each page at the lowest gap between them, never straight after a heading.
3. `paginateCanvas()` re-lays the canvas at one page per page height, so `toPdf()`'s fixed cuts land on the planned ones.

html2pdf's own `pagebreak` is off (`mode: []`), because its spacers would fight the planned cuts. The `.then()` callbacks get the worker as `this`, which is how they reach `this.prop.container`, `this.prop.canvas` and `this.prop.pageSize`. To check a change, export a long document and look at every page boundary for cut lines, headings left at the foot of a page, and blank pages.

The blob is then saved one of two ways, and both must keep working:
- inside a Claude artifact, through `window.claude.use('downloads')` and `downloadsNs.save(...)`, which can fail with the codes `declined`, `rate_limited` and `too_large`;
- anywhere else (local file, GitHub Pages), through an object-URL `<a download>` click.

The `@media print` block is a separate fallback for the browser's own Print. A colour change to the theme may need matching edits in `.sheet` and `@media print`, since both hard-code light values.

**Layout.** Above 820px the editor and preview sit side by side; the divider sets `--editor-w`, clamped to 25–75% (drag or arrow keys). At 820px or less the divider is hidden and the Write/Preview switch toggles `body[data-view="write"|"read"]` to show one pane at a time. Editor-to-preview scroll sync is proportional and one-way.

## Keeping docs in step

The README lists each library and its version. When a `<script>` tag's version changes, change the README to match.
