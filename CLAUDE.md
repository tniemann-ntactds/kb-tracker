# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single-file, standalone HTML workout tracker app designed for iPhone (saved as a home screen PWA). No build system, no dependencies, no npm — everything is in `index.html`.

## Running / Testing

Open the file directly in a browser or use any local HTTP server:
```
npx serve .
# or
python -m http.server 8080
```

There are no tests, no lint steps, and no compilation. Changes are visible immediately on refresh.

## Architecture

The entire app lives in one file with three logical sections:

### Data (top of `<script>`)
- `MDB` — exercise database: movement cues (`q`), muscle groups (`m`), recommended weight key (`w`), YouTube link (`v`)
- `WT` — weight key → display string map (`"16kg"` → `"16 kg / 35 lb"`)
- `FB` — feedback options array (emoji, label, color, response text)
- `DAYS` — 6-day program; each day has `sections[]`, each section has `ex[]` (exercises)
- `MMAP` — muscle group → SVG bounding box, used by `muscleSVG()` to highlight anatomy

### State
Flat module-level variables; no reactive framework. Key ones:
- `screen` — `'select' | 'workout' | 'done'`
- `si`, `ei` — current section index, exercise index
- `curSet`, `resting`, `needsFeedback` — set-tracker state machine
- Four independent interval timers: `restId` (between-set rest), `timerId` (timed exercises), `emomId` (EMOM mode), `amrapId` (AMRAP mode), `tickId` (elapsed clock)

### Render pattern
State changes → call `render()` → full DOM replacement via `app.innerHTML`. No virtual DOM or diffing. `clearTimers()` is called at the top of every `render()` to prevent timer leaks before re-attaching them.

### Exercise flow state machine
```
renderSets → finishSet → [needsFeedback → giveFeedback] → [resting → startRest] → curSet++ → repeat or nextEx()
```
Timed exercises (when `ex.ti` is set) use `startSetTimer()` instead of a manual "Set Complete" tap.

## Code Style Conventions

- Extremely terse variable names (`s`, `d`, `h`, `c`, `ex`, `m`) — match existing patterns when adding code
- Inline HTML string concatenation (`h += ...`) for all rendering — no template literals with complex nesting
- CSS in `<style>` block using utility classes (`btn`, `card`, `sect-label`, etc.) and CSS custom properties from `:root`
- Colors always reference CSS vars (`var(--ac)`, `var(--gn)`) or per-day `color` hex strings passed as `c`
- YouTube links use search URLs (`https://www.youtube.com/results?search_query=...`), not direct video links

## Adding Exercises

Add to `MDB` with keys: `m` (muscle array), `q` (movement cue string), `w` (weight key from `WT` or `"none"`/`"band"`), `v` (YouTube search URL or `null`).

## Adding Workout Days

Add to `DAYS` array. Section types:
- Normal: `{name, note, ex[]}` — each exercise: `{k (MDB key), d (display), s (sets), rp (reps), rs (rest seconds), ti (timed bool), t (seconds), fb (feedback bool)}`
- EMOM: add `emom:true, emomDur` (seconds) to section
- AMRAP: add `amrap:true, amrapDur` (seconds) to section

## Agent skills

### Issue tracker

Issues live as local markdown files under `.scratch/`. See `docs/agents/issue-tracker.md`.

### Triage labels

Four active roles: `needs-triage`, `needs-info`, `ready-for-agent`, `wontfix`. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context layout — one `CONTEXT.md` + `docs/adr/` at the repo root. See `docs/agents/domain.md`.
