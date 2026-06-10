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
- The two `axis` values drive color coding throughout (role = red, location = orange) via `.axis-role` / `.axis-location` and `mk-*` / `n-mark` classes.

## Conventions when editing

- **Keep it one file, zero dependencies** — no bundler, no npm, no framework. Match the existing vanilla-JS / hand-written-CSS style.
- To change the org chart, edit the `TREE` data object (and `branch`/`booth` builders), not the generated DOM.
- When adding or reordering slides, update both the `.pres-slide` markup and `slideTitles[]`.
- Screenshots referenced by the slides live in `images/` (`1.png`–`5.png`).
