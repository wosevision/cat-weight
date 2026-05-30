# catweight

A simple client-side data visualization of Jasper's and Enzo's weight over
time, sourced from CSV exports of a "smart" litter box's proprietary app.

## Getting started

```bash
npm install
npm run dev
```

Then open the URL Vite prints (usually [http://localhost:5173](http://localhost:5173)).

## Features

- **Daily median view** with a translucent min/max range band and a 7-day
  rolling-median trendline overlay per cat — smooths the day-to-day noise
  caused by mid-deposit scale readings.
- **Raw readings view** that scatters every parsed weight, useful for
  inspecting individual events.
- **Date-range presets** (All / 1Y / 90D / 30D), anchored to the most recent
  reading so old datasets still render.
- **Drag-zoom and pan** on the chart (wheel/pinch to zoom, Alt-drag to
  box-zoom, Shift-drag to pan). A "Reset zoom" button appears once you've
  zoomed.
- **Auto dark mode** following the OS `prefers-color-scheme`.
- **CSV upload** via button or page-wide drag-and-drop, deduped against
  bundled data.

## How data flows in

There are two ways to load CSV exports:

1. **Drop them in `data/`.** Anything matching
   `data/poobox_activity_YYYY-MM-DD.csv` is bundled at build time via Vite's
   glob import and shows up automatically on next dev-server start.
2. **Upload at runtime.** Click "Add CSV export(s)" or drag-and-drop one or
   more `.csv` files anywhere on the page. Uploaded readings are merged with
   the bundled ones in-memory; nothing is persisted across reloads.

Each CSV row looks like:

```csv
Activity,Timestamp,Value
Weight recorded,05-30 10:23 a.m.,5.5 kg
```

A few quirks worth knowing about:

- The `Timestamp` column has no year. The year is inferred from the export
  filename (`poobox_activity_2026-05-30.csv` → 2026), with a wrap to the
  previous year for any rows whose `MM-DD` is later than the export's.
- `Value` carries its unit. Anything other than `kg` is skipped with a
  console warning so a unit change can't silently corrupt the chart.
- Multiple cats share the box. Readings are attributed to Jasper or Enzo via a
  fixed weight threshold (see `src/classify.ts`), which works as long as their
  typical weights stay well-separated.

## Scripts

- `npm run dev` — Vite dev server with HMR.
- `npm run build` — type-check and produce a static build in `dist/`.
- `npm run preview` — serve the production build locally.
- `npm run typecheck` — type-check only.

## Stack

- [Vite](https://vitejs.dev/) — dev server and bundler. Requires Node
  20.19+ or 22.12+.
- [TypeScript](https://www.typescriptlang.org/) — strict mode.
- [Chart.js](https://www.chartjs.org/) + `chartjs-adapter-date-fns` — chart
  rendering with a time-aware x-axis.
- [`chartjs-plugin-zoom`](https://www.chartjs.org/chartjs-plugin-zoom/) —
  drag/wheel zoom and pan.

See [`AGENTS.md`](./AGENTS.md) for a tour of the source layout and the small
pile of CSV-format gotchas.
