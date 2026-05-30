# Agent guide

Guidance for AI agents working in this repo. Keep this file updated as the
project evolves.

## What this project is

A small client-side web app that visualizes Jasper's and Enzo's weight over
time, using CSV exports from a "smart" litter box's proprietary app. The
intended audience is a single user (the owner) viewing locally — there is no
backend, no auth, no telemetry. Keep it simple.

## The cats

Two cats share the litter box:

- **Jasper** — the heavier of the two (~5.6 kg in the initial sample).
- **Enzo** — the lighter (~4.5 kg).

Their typical weights are far enough apart that a simple weight threshold
(`WEIGHT_THRESHOLD_KG` in `src/classify.ts`) reliably attributes a reading to
one cat or the other. If their weights drift close together, revisit the
threshold or introduce a per-reading override UI.

## Repo layout

- `data/` — raw CSV exports, one file per month. Filenames follow
  `poobox_activity_<YYYY-MM-DD>.csv`. **Treat these as read-only inputs.** Do
  not rewrite, normalize, or rename them; parse them at runtime instead.
- `src/`
  - `main.ts` — entrypoint; wires DOM, chart, and upload together.
  - `types.ts` — shared types and the cat registry (`CATS`, `CAT_IDS`).
  - `parse.ts` — pure CSV parser. Takes raw text + an inferred export date,
    returns `RawWeightReading[]`. Handles the `a.m.`/`p.m.` and unit quirks.
  - `classify.ts` — assigns each reading to Jasper or Enzo via threshold.
  - `aggregate.ts` — daily-median aggregation per cat.
  - `data-loader.ts` — Vite glob import of `data/*.csv` plus the runtime
    `File` upload reader. Also owns the dedupe pass.
  - `upload.ts` — file-input + page-wide drag-and-drop UI glue.
  - `chart.ts` — Chart.js setup and view switching (daily median ↔ raw).
  - `stats.ts` — per-cat summary numbers shown beside the chart.
  - `style.css` — all styles. Plain CSS, no preprocessor.
- `index.html` — single page, references `/src/main.ts` as a module.
- `package.json` — npm metadata. Vite is the dev server / bundler; Chart.js is
  the visualization library; `chartjs-adapter-date-fns` provides the time-axis
  date adapter. Prefer adding new deps only when clearly needed.
- `tsconfig.json` — strict TypeScript, `noEmit` (Vite handles transpilation;
  `npm run typecheck` checks types).
- `README.md` — human-facing overview.
- `AGENTS.md` — this file.

## CSV format gotchas

The CSV is messier than it looks. Before changing the parser, re-read a sample
file in `data/`. Known quirks:

- Header row: `Activity,Timestamp,Value`.
- `Activity` may be values other than `Weight recorded` (e.g. usage events).
  The parser only emits weight rows.
- `Timestamp` is `MM-DD H:MM a.m./p.m.` with **no year**. The year is inferred
  from the export filename's date suffix; rows whose `MM-DD` is later than the
  export's `MM-DD` are assumed to be from the previous year (December rows in
  a January export, etc.).
- `a.m.` / `p.m.` use lowercase letters with periods — not `AM`/`PM`. Don't
  feed the string straight to `Date.parse`.
- `Value` is a unit-suffixed string like `"5.5 kg"`. The parser strips the
  unit and warns + skips on anything other than `kg`, since a unit mix-up
  would silently corrupt the chart.
- Rows are roughly newest-first, but downstream code does not rely on
  ordering — sort explicitly when needed.

## Conventions

- TypeScript with `strict` on. New modules use `.ts` and explicit `.ts`
  imports (the bundler resolves them; this keeps the source consistent with
  Node's ESM resolution rules).
- ES modules (`"type": "module"` in `package.json`).
- No global state libraries until there's a real need. `src/main.ts` keeps
  the small amount of UI state it needs in module-local variables.
- Keep CSV parsing pure: a function that takes text + the inferred export date
  and returns a typed array. No DOM, no fetch, no globals — easy to unit-test
  later.
- Time math runs in the user's local timezone (matching the CSV's local-time
  semantics). `aggregate.ts#localIsoDate` is the canonical day-key formatter.

## Things to avoid

- Don't commit large binary assets or anything in `node_modules/` /
  `dist/` — they're already gitignored.
- Don't introduce a backend, database, or build-time data pipeline. The CSVs
  should be loaded and parsed in the browser.
- Don't auto-rename or "clean up" files in `data/`. The raw export filenames
  carry the year, which the parser depends on.
- Don't add analytics, error reporting, or external network calls.
- Don't switch frameworks (React/Vue/Svelte) without buy-in — vanilla DOM +
  Chart.js is the current default and the codebase is small enough not to need
  more.

## When in doubt

Ask the user before:

- Adding a new top-level dependency.
- Introducing a framework — vanilla TS + Chart.js is the current default.
- Changing the CSV input format or adding a server-side processing step.
- Materially changing the cat-assignment heuristic (it's currently a fixed
  weight threshold; smarter approaches like k-means or per-reading overrides
  are valid but should be discussed first).
