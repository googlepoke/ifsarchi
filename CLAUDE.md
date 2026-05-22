# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A single self-contained HTML slide deck — **not a typical codebase**. The deliverable is `IFS-Cloud-Technical-Architecture.html` (~2.6k lines, inline `<style>` + `<script>`, no build step, no package manager, no tests). It is a 40-slide technical briefing on IFS Cloud architecture, designed both as a ~90-minute presenter deck and a leave-behind reference for prospect architects and security teams.

The folder `IFS-Cloud-Technical-Architecture_files/` holds local copies of every external asset (Google Fonts CSS, D3, IFS logos) so the deck works offline from `file://`.

## Running it

Open the `.html` file in any modern browser. That is the entire workflow.

Presenter controls (handler at ~line 1788):

- `←` `→` / `Space` / `PageUp` `PageDown` — navigate
- `Home` `End` — first / last slide
- `N` — toggle speaker notes panel
- `?` — help overlay
- `F` — fullscreen
- `O` — overview grid trigger exists (`toggleOverview()` ~line 1808) but is a stub; the grid isn't implemented
- Click zones: left 20 % = prev, right 38 % = next. Elements matching `.n-node, .tg, button, a, .no-nav, #notes, #help` are excluded from click-nav.

## Deck structure

Slides are `<section class="slide" id="sNN" data-label="…">` between lines ~288 and ~1708. 40 slides, three topics, each ending in a recap:

1. **Architecture & integration** (s03–s14) — principles, layered tiers, tech stack, container topology, Aurena, REST/OData, IFS Connect, integration patterns
2. **Platform operations, security & compliance** (s15–s28) — Kubernetes runtime, scalability, HA, defence in depth, identity, data protection, certifications, GDPR, secure dev, extensibility
3. **Deployment & hosting** (s29–s38) — deployment models, shared responsibility, Australian hosting topology, Sydney production, DR, residency, environments, ops

Plus title + agenda (s01–s02), three topic dividers, three recaps, summary (s39), next steps + Q&A (s40). The `<!-- SLIDES_INSERT -->` marker at line 1709 is the canonical insertion point for new slides.

## Authoring conventions

When adding or editing slides:

- **Three-way numbering** must stay in sync: `id="sNN"`, `data-label="…"` (shown in the notes-panel header and overview), and the per-slide footer `<b>NN</b> / 40`. The total `/ 40` in every slide footer must be updated if the slide count changes.
- **Speaker notes** live inline as `<div class="notes-data">…</div>` per slide. Hidden on the stage (`display:none`) and rendered into the bottom panel at runtime by `renderNotes()`. Use `<h4>` for sub-headings inside notes — they get mono-uppercase styling.
- **Exactly one slide should boot active.** Whichever slide carries `class="slide active"` is the first one shown (currently s02, the agenda).
- **D3 diagrams are lazy-registered**, not inlined into the slide markup — see below.

## D3 diagram framework

Diagrams register via `dgReg(slideId, hostElementId, viewBoxW, viewBoxH, fn)` (defined ~line 1838). Each is initialized on first activation of its slide and cached in `initOnce`. The callback receives `(svg, tip, px, vbW, vbH, el)`:

- `svg` — d3 selection of a fresh `<svg>` with the configured viewBox
- `tip` — tooltip controller (`show(html, x, y)` / `hide()`)
- `px(event)` — DOM event → viewBox coords, accounting for the stage scale
- `vbW` / `vbH` / `el` — viewBox dimensions and the host DOM element

Use the shared helpers — **don't hardcode**:

- `R(parent, x, y, w, h, opts)` — rounded rect, `{r, fill, stroke, sw}`
- `T(parent, x, y, str, opts)` — text, `{fs, fw, fill, mono, anchor, ls}`
- `C` — color palette object (see Nexus Black below)
- `SANS` / `MONO` — font-family constants
- `class="n-node"` on hit-target groups enables hover styling and excludes them from global click-nav

Existing diagrams start at line ~1863 (`05 · LAYERED ARCHITECTURE`) and are separated by `/* === NN · TITLE === */` banner comments.

## Nexus Black design system

Same dark design language as the `nexus-black-pptx` skill — keep edits consistent with it.

- **Palette** — defined in CSS `:root` (~line 15) and JS `C` (~line 1831). **Both copies must stay in sync.**
  - Surfaces: `--page #090909`, `--s1 #101011`, `--s2 #161618`, `--s3 #1d1d20`
  - Lines: white at 8 / 14 / 22 % opacity
  - Text: `--tx #F2F2F3`, `--tx-dim #9A9AA2`, `--tx-faint #65656d`
  - Brand violet ramp: `--v1 #48157C` → `--v2 #8427E2` → `--v3 #A855F7`
  - Accents: `--teal #2DD4BF`, `--blue #0365D8`
  - Status: `--hi #EF4444` / `--md #F59E0B` / `--lo #27B663` / `--ok #0365D8`
- **Type**: Inter Tight (sans) + JetBrains Mono (mono), both loaded from `_files/css2`. Roles: `.kicker` (mono 11 px violet uppercase), `h1.title` 43 px, `h2.stitle` 33 px, `.lead` 17 px dim.
- **Type-pairing fallback** when porting to PowerPoint via the `nexus-black-pptx` skill: Suisse Intl + PP Supply Mono are the original brand fonts; the deck uses Inter Tight + JetBrains Mono as open substitutes.

## Asset folder quirks

- `_files/d3.min.js.download` — the `.download` suffix is a Chrome "Save Page As" artifact. The `<script src>` at line 10 points at this exact filename; renaming the file requires updating the tag.
- `_files/css2` is a saved Google Fonts CSS response (extensionless). Referenced verbatim by the `<link>` at line 9.
- `Logo.svg` and `Logo(1).svg` are byte-identical save-time duplicates. Neither is currently referenced — the wordmark is rendered as styled text in `.logo`.

## What not to do

- **Don't introduce a build step, bundler, or modular split** unless explicitly asked. Single-file portability is the point — the deck has to work as a USB-stick leave-behind from `file://`.
- **Don't break the 1280 × 720 stage.** Every slide and diagram assumes that fixed canvas; `fit()` (~line 1754) scales it to the window. Authoring at any other dimension misaligns the whole deck.
- **Don't strip `.notes-data` blocks** — they are content, not debug scaffolding.
- **Don't hardcode colors or fonts** in new diagrams. Pull from `C` / `SANS` / `MONO` (JS) or the `--` CSS variables — palette changes should land in one place.
- **Don't update an individual slide's page-count footer** without updating all 40.
