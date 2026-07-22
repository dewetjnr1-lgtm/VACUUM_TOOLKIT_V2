# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Vacuum & Compressor Toolkit — a mobile-first web app of engineering calculators for
sizing and troubleshooting industrial vacuum pumps, compressors, and pneumatic
conveying systems (liquid-ring pumps, blowers, etc.). It is aimed at field/sales
engineers, not developers.

The entire app is **one static file**: `index.html` (~197 KB, self-contained
HTML + CSS + JS, no external requests, no build step, no dependencies, no
`package.json`). Open it directly in a browser to run it — there is nothing to
install or compile.

## Repository contents

- `index.html` — the whole app (markup, `<style>`, and `<script>` all inline).
- `WORKFLOW.md` — the non-technical owners' (Chris + Uncle) day-to-day process for
  requesting changes via Claude Cowork and shipping them through GitHub Desktop.
  Read it to understand who you're building for: **the people using this repo do
  not read or write code themselves** — every change request arrives as plain
  English, and `index.html` is the only file they expect Claude to touch.
- `FORMULA_VERIFICATION.md` — a dated, one-off audit report that hand-verified every
  calculator's formula against industry references (Busch, Leybold, Giampaolo,
  Crane TP-410, Mills, DIN 1343/ISO 2533, WMO/Sonntag). Treat it as a historical
  record of a verification pass and its fixes, not as current documentation — the
  app has grown new calculators (orifice sizing, heat load, cooling tower make-up,
  pre-separator sizing, conveying step-by-step) since it was written, and it isn't
  updated automatically when formulas change. If you touch a calculator's formula,
  re-derive/check it against the cited reference yourself rather than trusting the
  file to still be accurate for that calculator.

There is no test suite, linter, package manifest, or CI in this repo.

## Development workflow

- **No build/lint/test commands exist.** To "run" the app, just open `index.html`
  in a browser (or serve the directory with any static file server, e.g.
  `python3 -m http.server`).
- After editing a calculator, sanity-check it in a browser: open the tool, click
  **Insert example**, and confirm the results and the "Show the math" panel look
  right. There's no automated test to catch a broken formula.
- Commits are plain, descriptive, present-tense summaries of user-visible changes
  (e.g. "Fix nonsensical negative vacuum readings", "Add orifice sizing, water heat
  load, cooling tower make-up, and workbook conveying calculators"). Match that
  style — no conventional-commit prefixes are used.
- Keep everything in `index.html` unless there's a strong reason to split it out;
  the single-file, dependency-free design is intentional (works offline, can be
  saved to a phone home screen, no build pipeline for non-technical owners to
  maintain).

## Architecture of `index.html`

The file is a client-side single-page app with three parts in document order:

### 1. `<style>` — theme and components
CSS custom properties in `:root` define the palette (`--teal`, `--green`,
`--accent`, `--warn`, etc.) and typography. Reusable component classes:
`.card`, `.fld`/`.ctrl` (input rows), `.res`/`.res.big` (results), `.means`
(plain-language explanation boxes), `details`/`.math` ("Show the math" panels),
`.note.warn`, `.bub`/`.catrow` (nav tiles), `.demo`/`.lawcv` (gas-law canvas
toys). Reuse these classes rather than inventing new ones when adding UI.

### 2. `<main>` — pages
Every screen is a `<section class="page" id="...">`, all present in the DOM at
once; only the one with class `active` is visible (`show(id)` in the script
toggles this — see below). Pages nest into a shallow hierarchy:

- `home` → three category menus: `cat_learn` ("Play and learn"), `cat_size`
  ("Size it"), `cat_site` ("On-site checks").
- `cat_learn` → `learn` (gas-law sub-menu: `learn_intro`, `learn_boyle`,
  `learn_charles`, `learn_gl`, `learn_combined`, `learn_ideal`, `learn_dalton`),
  `cv` (unit converter), `ref` (reference tables).
- `cat_size` → calculator pages `fc` (Normal/Actual flow), `vac` (vacuum pump
  sizing), `cmp` (compressor sizing), `pc` (conveying sizing), `cw` (conveying
  step-by-step), `mp` (which machine), `vap` (vapour & wet gas), `orf` (orifice
  sizing), `hl` (heat load, water), `mw` (cooling tower make-up), `ps`
  (pre-separator sizing).
- `cat_site` → `pd` (pump-down time), `lk` (leak check), `pl` (pipe loss), `pw`
  (power & motor current), `ts` (guided fault-finder / troubleshooter).

Each calculator page follows the same template: an optional collapsible "About"
box, a "Fill these in" card with `Insert example`/`Clear` buttons and one
`.fld` per input, a results card with `.res` rows, a `.means` box that narrates
the answer in plain English, and a `<details>` "Show the math" panel with a
live formula (numbers plugged in) plus a source citation.

### 3. `<script>` — logic
No frameworks; plain DOM APIs. Key pieces, roughly in file order:

- **`G`** — the unit-conversion table: for each quantity (`pressure`, `flow`,
  `flowIn`, `flowN`, `temp`, `length`, `power`, `time`, `leak`, `vol`,
  `velocity`) an array of `[label, factorToBaseUnit]`. Temperature is handled
  specially (non-linear). All calculators store values internally in the base
  unit (kPa, m³/h, °C, etc.) and convert on display.
- **Value helpers** — `gv`/`gs` read a field's raw number/select value;
  `readUnit`/`readMag`/`readPress` convert an input field to base units
  (`readPress` additionally resolves gauge/absolute/vacuum readings against a
  local atmospheric pressure `B`); `showU`/`renderU` and `showPress`/
  `renderPress` store a computed result in base units (`BASE`/`PB` dicts keyed
  by field id) and render it in whatever unit the paired `<select>` currently
  shows, so switching a result's unit dropdown re-renders instantly without
  recomputation.
- **`recalc()`** — the single function that recomputes *every* calculator on
  every input, wrapped as one IIFE block per calculator (`// FLOW CONVERTER`,
  `// VACUUM`, `// COMPRESSOR`, `// VAPOUR`, `// PUMP DOWN`, `// LEAK`, `// PIPE
  LOSS`, `// POWER`, `// CONVEYING`, `// WHICH MACHINE`, etc.). It's wired up
  once via `document.addEventListener("input", recalc)` /
  `("change", recalc)` — there's no per-field wiring. When adding a field to a
  calculator, add its read/compute/display logic inside that calculator's block
  in `recalc()`; it will be picked up automatically by the global listener.
- **`EX`** — example input sets per calculator, applied by `ex(c)` (Insert
  example) and cleared by `clr(c)` (Clear), both of which call `recalc()`
  afterward.
- **Navigation** — `show(id)` sets `CUR`, toggles `.active` on the matching
  `.page`, sets the header title from `TT[id]`, and scrolls to top. `PARENT`
  maps each page id to its parent for the back button (`goBack()`). `TT` holds
  display titles. Add both a `TT` and `PARENT` entry for any new page.
- **Gas-law canvas animations** (`learn_*` pages) — small particle-in-a-box
  physics toys: `mkBalls`/`stepBallsBox`/`drawBalls` are shared primitives;
  `drawBoyle`/`drawCharles`/`drawGL`/`drawCombined`/`drawIdeal`/`drawDalton`
  render each law; `gasLoop()` drives the animation frame loop.
- **`TS`** — the guided troubleshooter's decision tree (`ts` page): each key is
  either a question node `{q, o:[[choiceLabel, nextKey], ...]}` or a result node
  `{r, why, fix:[...]}`. `TSTK` is the navigation stack; `tsGo`/`tsBack`/
  `tsReset` push/pop/reset it and `tsRender()` redraws the current node. To add
  a new fault path, add nodes to `TS` and link them from an existing node's `o`
  list.
- **`MAT`** — material presets (density, particle velocity, dust flag) for the
  conveying calculators, indexed by the material `<select>`.
- **`EX_LAW`/`renderExample`/`shuffle3`/`EX_QUEUE`** — worked-example carousels
  on the gas-law pages; each law has 3 examples shown in shuffled order via
  "Show another example".

## Conventions to follow

- Every numeric input pairs with a unit `<select>`; every result pairs with a
  unit `<select>` (`data-g` attribute names which `G` table it uses) so the
  reader can switch units without retyping. Follow this pattern for new fields.
- Pressure readings are entered as gauge/absolute/vacuum via a `_r` select
  (`REF` array) and resolved through `readPress`/`showPress`/`renderPress`
  against a local atmospheric-pressure field — don't hardcode 101.325 kPa as
  "atmosphere" when a local-altitude field is available on the page.
- Every calculator's `.means` explanation and "Show the math" panel must stay in
  sync with the formula in `recalc()` — when you change a formula, update both
  the plugged-in-numbers string and the plain-English narration, and (per
  `FORMULA_VERIFICATION.md`'s house style) cite the source standard/reference in
  the math panel.
- Copy is deliberately plain-English, non-jargon, and consistent (e.g. "math"
  not "maths" per commit `1eabfb7`) — this app is used by non-specialists.
- Guard against invalid physical states (e.g. target pressure ≥ start pressure,
  absolute pressure ≤ 0, water boiling before saturation) by producing `NaN`
  and showing a specific warning message in `.means`, rather than silently
  displaying a wrong number — every existing calculator does this; match it.
