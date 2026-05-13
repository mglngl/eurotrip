# CLAUDE.md — Eurotrip 2026 Project Memory

> Read this file before touching any code in this repo. It defines the design system, architecture, and conventions that every change must respect.

## Project overview

A single-page travel itinerary for a 16-day Europe trip (Aug 10 – Aug 25, 2026). The page visualizes daily stops, transit between cities, highlights, and trip-level stats. It is a personal planning tool, not a public product — but it is built to magazine-grade visual quality.

- **Repo:** `mglngl/eurotrip` on GitHub
- **Hosting:** Currently the raw repo; may be deployed to GitHub Pages later
- **Audience:** The trip-takers themselves

## Architecture — and why it's intentional

This is a **single HTML file** (`index.html`) with no build step. Everything loads from CDNs.

- **React 18 UMD** + **ReactDOM** from `unpkg.com`
- **Babel Standalone** transpiles the inline `<script type="text/babel" data-presets="env,react">` block in the browser
- **Tailwind CSS** via the `cdn.tailwindcss.com` runtime
- **Google Fonts:** Fraunces, Inter, JetBrains Mono

### Rules for the architecture

1. **Stay single-file.** Do not introduce a bundler, `package.json`, `node_modules`, npm dependencies, or build tooling unless I explicitly ask. The whole point is that the file is portable and openable in a browser with zero setup.
2. **No new external runtime dependencies** without asking first. If a feature needs a library, propose it and wait for approval before adding the `<script>` tag.
3. **No JSX module imports.** Everything lives inside the existing `<script type="text/babel">` block.
4. **No build artifacts in the repo.** Don't commit anything that isn't human-authored.

## Design system — the non-negotiables

The aesthetic is editorial, warm, and analog. Treat the design tokens below as fixed law. If a change requires a new token, propose adding it to `:root` rather than hardcoding a value inline.

### Color palette (CSS custom properties on `:root`)

| Token              | Value     | Used for                                    |
| ------------------ | --------- | ------------------------------------------- |
| `--cream`          | `#f6f1e7` | Page background (warm off-white)            |
| `--cream-2`        | `#efe7d6` | Secondary panels, hover states              |
| `--cream-3`        | `#e6dcc6` | Borders, dotted dividers, scrollbar tracks  |
| `--ink`            | `#0e1f3a` | Primary text, dark fills, navy buttons      |
| `--ink-2`          | `#1b2e4f` | Slightly lighter ink for body copy          |
| `--terracotta`     | `#c2542b` | Accent, numbers, progress bar               |
| `--terracotta-soft`| `#e08a5f` | Shimmer gradient mid-stop                   |
| `--mute`           | `#6b6452` | Muted labels, mono caption text             |

**Never** introduce a new color (especially blue, green, or purple accents) without explicit approval. The palette is the brand.

### Typography

- **`Fraunces`** — display serif. Use for city names, headlines, hero text, day pill names, stat numbers. Class: `font-display`. Optical sizing is on; weights 300–900 are loaded.
- **`Inter`** — default body sans, applied via `body { font-family: ... }`. No class needed; just write text.
- **`JetBrains Mono`** — for tiny uppercase labels with wide letter-spacing (e.g. `"DAY 03"`, `"COUNTRIES"`, `"HIGHLIGHTS"`). Class: `font-mono`. Always pair with `uppercase` and a tight `tracking-[0.2em]`–`tracking-[0.22em]`.

### Iconography

- City scenes are **hand-coded inline SVG** in the `CityIllustration` component — flat, geometric, two-to-three color, no gradients, no photos, no external assets. Each new city must follow the same visual language: silhouettes, simple repeating shapes, palette pulled from the day's `palette` array. If a city is added, add a matching SVG scene to the `scenes` object.
- Emoji are used sparingly as **glyphs** on each day (one per day, in the `glyph` field). They should be evocative, not decorative — a single object that captures the day.

### Motion

Three animation classes are defined and **reused everywhere**. Don't invent new ones unless a new interaction pattern is being introduced:

- `fade-key` — fade + slight rise on key change (used on the hero when switching days)
- `slide-in` — fade + slide from right (used on title, summary, sections)
- `scaleup` — fade + scale 0.96 → 1 (used on the hero card mount)

Easings are always `cubic-bezier(.2,.7,.2,1)`. Durations are 220–480 ms. The `progress-shimmer` animation on the progress bar runs 6s linear infinite.

### Layout patterns

- **Max content width:** `1400px` (overall) / `920px` (main panel reading column).
- **Mobile breakpoint:** `900px`. Helper classes `.desktop-only` and `.mobile-only` already exist — use them, don't reinvent.
- **Sticky header:** `top-0 z-30` on the `TopBar`. The sidebar sticks below it at `top-[73px]`.
- **Paper texture:** the `.paper` class adds a subtle dot-pattern background. Apply it to the page root and the sidebar.
- **Dividers:** the `.dotline` class is the canonical horizontal divider. Don't use `<hr>` or solid borders for section breaks.

## Data model — the single source of truth

The entire UI is driven by the `TRIP` constant: a 12-element array of day objects. **Adding a stop, editing a date, or changing a transit line is always a `TRIP` edit, never a JSX edit.**

Each day object shape:


```js
{
  day: 3,                          // 1-indexed day number
  city: "Bruges + Brussels",       // Display label (combo days use " + ")
  cities: ["Bruges", "Brussels"],  // Array — must match a key in CityIllustration.scenes
  country: "Belgium",              // Display string; combo countries use "CH → IT"
  countryCode: "BE",               // ISO-2; used for the STATS countries count
  dateLabel: "Jun 07 · Sun",       // "MMM DD · DDD" format
  glyph: "🍫",                     // Single emoji
  palette: ["#E8DCC0", "#7B3F2A", "#0E1F3a"],  // [bg, ink, accent] for the SVG
  summary: "...",                  // 2–3 sentence prose blurb
  highlights: ["...", "...", "..."], // Exactly 3 strings, ideally
  transit: {
    mode: "Train",                 // Train | Eurostar | Flight | Day trip | Rest
    line: "Thalys · AMS → BRU",
    duration: "1h 52m",
    overnight: true,               // Optional
  },
  isCombo: true,                   // True when two cities are visited same day
  legKm: 220,                      // Distance of the leg arriving INTO this day
  combo: [                         // Required when isCombo: true
    { name: "Bruges",   glyph: "🏰", note: "Morning · canals, chocolate, Markt" },
    { name: "Brussels", glyph: "🧇", note: "Afternoon · Grand-Place, beer, frites" }
  ]
}
```


The `STATS` IIFE at the bottom of the data block derives totals from `TRIP`. **It updates automatically** when `TRIP` changes — don't hand-edit stat numbers anywhere.

## Conventions

- Indent with **2 spaces**.
- Component names: `PascalCase`. Helpers: `camelCase`.
- Tailwind utility ordering: layout → box-model → typography → color → state. Group related utilities.
- Keep components small and named. If a JSX block grows past ~30 lines, extract a sub-component.
- Inline `<style>` is fine for design tokens and animations — do **not** create a separate CSS file.
- All copy is in English. Voice is wry, observational, slightly literary (see existing `summary` fields for tone).

## What "preserving the design" means in practice

When asked for any change, ask yourself:

1. Does this use existing tokens (`--cream`, `--ink`, `--terracotta`), or am I inventing values?
2. Does this use `font-display` / `font-mono` correctly, or am I styling text inline?
3. Does this respect the dotline divider, paper texture, and sticky header patterns?
4. If a new component, does it match the editorial spacing rhythm (24/28 px gaps, generous whitespace)?
5. If touching `TRIP`, does each new field follow the existing object shape exactly?

If any answer is no, stop and ask before proceeding.

## Working with me

- I'm a non-developer with technical literacy. **Explain trade-offs in plain language** before making structural decisions.
- Prefer **small, reviewable diffs** over big rewrites. Default to Ask mode — propose, wait for accept.
- When you finish a unit of work, **summarize what changed in 3–5 bullets** and suggest a commit message.
- If you're uncertain about my intent, ask one focused question rather than guessing.
- Never invent trip facts (dates, flight numbers, restaurants) — if I haven't given them to you, ask.
