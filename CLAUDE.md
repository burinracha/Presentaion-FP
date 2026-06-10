# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

An **interactive Thai-language teaching site** that explains a Visitor Management System (VMS) for residential villages/projects — from the top-level Super Admin down to the guard at the booth (the primary end user of the "App ยาม"). Inspired by the click-to-explore style of [9arm lab](https://www.9arm.co/lab/fiberoptics.html), not a static diagram. All copy is in Thai.

## Running

There is no build step, framework, or dependency. The entire site is one static file.

- Open `index.html` directly in a browser, **or**
- Serve locally: `python -m http.server 8080` then open `http://localhost:8080`

Deployment is GitHub Pages (serves `index.html` from the repo root).

## Architecture

Everything lives in **`index.html`** — HTML, CSS (in one `<style>` block), and vanilla JS (one `<script>` at the bottom). No external JS; only Google Fonts (Kanit + IBM Plex Sans Thai) are loaded over the network.

Two coordinated systems share the page:

### 1. Slide deck / scroll navigation
- Slides are `.pres-slide` sections in a scroll container; the sidebar (`.sb-item[data-slide]`) and prev/next buttons call `scrollToSlide(i)`.
- An `IntersectionObserver` ("scrollspy") highlights the active slide and updates the title/counter via `setActive(i)`.
- `slideTitles[]` must stay in sync with the slide order. Keyboard: `←/→` (or `↑/↓`) change slides, `S` toggles the sidebar.

### 2. Interactive org-chart tree (slide 0)
- A single nested data object `TREE` is the source of truth. Each node has: `id`, `name` (Thai), `en` (subtitle), `axis` (`'role'` | `'location'`), and optional `children`, `devices`, `person`, `focus`, `kind`, `defaultCollapsed`.
- Helper builders `branch(...)` and `booth(...)` generate repeated subtrees — edit these to change every branch/booth at once rather than hand-writing each node.
- On load, an `index()` pass populates `byId`, `parentOf`, and `personIndex` (the last groups nodes by the same real person, e.g. `BOOK`, to draw the "คนเดียวกัน" / same-person link).
- `nodeEl(node, depth)` recursively builds the DOM. `select(id)` highlights the node, its ancestor path (`.onpath`), and same-person nodes, then `renderDetail()` fills the side panel.
- The two `axis` values drive color coding throughout (role/คน = red, location/สถานที่ = **purple**) via `.axis-role` / `.axis-location` and `mk-*` / `n-mark` classes.

## Conventions when editing

- **Keep it one file, zero dependencies** — no bundler, no npm, no framework. Match the existing vanilla-JS / hand-written-CSS style.
- To change the org chart, edit the `TREE` data object (and `branch`/`booth` builders), not the generated DOM.
- When adding or reordering slides, update **all four** in lockstep: the `.pres-slide` markup, its `.sb-item[data-slide]` button, `slideTitles[]`, and the `data-index` attribute (cosmetic but keep tidy).
- **Slide-index invariant:** the Nth `.sb-item` button must have `data-slide="N"` and point to the Nth `.pres-slide` in DOM order. Clicks use the `data-slide` attribute; the scroll-spy highlight uses positional `slides.indexOf(...)` — both must agree.
- Screenshots referenced by the slides live in `images/` (`1.png`–`6.png`, plus `3.5.png`).
- Keep emoji minimal — `✓` for feat-list items; the only intentional emoji are the four app-tile icons on the last slide.

## Working notes (read this to go faster)

- **Reply in Thai** — the user works in Thai.
- **Don't use `preview_screenshot`** — it hangs (~30s timeout) in this environment. Verify changes with `preview_eval` instead: read DOM geometry (`getBoundingClientRect`), computed styles (`getComputedStyle`), counts, and class state. That's how every change this far was confirmed.
- **Bias to action.** Edits here are one cheap, reversible file. Make the change and report it, offering to revert — only stop to ask when a choice is genuinely ambiguous *and* costly to undo.

## Quick reference

- **Palette:** role/คน = `--red` · location/สถานที่ = `--orange` (variable name kept, but the value is **purple** `#7a4dd0`) · device = `--teal` · brand = `--accent` (blue) · person/guard = `--gold`.
- **Colors live in `:root`** (top of the `<style>` block); changing a variable recolors everywhere.
- **Example Code/Checksum pairs threaded through the deck:** group `GRP-LH-0001` / `7F3A` (Step 2) · role `GC-FPA000-2375B` / `9C2D` (Step 3 → reused in Step 3.5) · action `ACT-GATE-01` / `B1E0` (Step 4). Keep the role pair identical across Step 3 and 3.5.
- **Reusable slide components:** `.feat-list` (✓ items), `.code-callout` (dark box with `.cc-code` + green `.cc-sum` checksum chip), `.flow-chips`, `.guard-note` / `.sync-note` (tip boxes), `.slide-shot` (screenshot + caption).
