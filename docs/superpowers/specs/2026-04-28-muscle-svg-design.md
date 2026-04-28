# Muscle SVG Redesign

**Date:** 2026-04-28  
**Scope:** `index.html` only — MMAP const + `muscleSVG()` function

## Goal

Improve the muscle highlight diagram on the exercise card. Two problems today:
1. Body shape uses mismatched rects/paths with poor proportions
2. Highlights are axis-aligned rectangles that don't follow muscle contours

## Body Shape

Replace current body paths with **Refined Athletic** proportions (viewBox `0 0 60 68`, rendered 72×72):

| Part | Path / Shape |
|------|-------------|
| Head | `<ellipse cx="30" cy="7" rx="5.5" ry="6.5"/>` |
| Neck | `M27.5,13 L32.5,13 L32,16 L28,16 Z` |
| Torso | `M17,16 L43,16 L41,40 Q38,44 30,44 Q22,44 19,40 Z` — V-taper with curved bottom |
| Pelvis | `M21,43 L39,43 L40.5,48 L19.5,48 Z` — separate hip block |
| Left upper arm | `M17,16 L10,19 L9,36 L14,37 L18,24 Z` |
| Right upper arm | `M43,16 L50,19 L51,36 L46,37 L42,24 Z` |
| Left forearm | `M9,36 L7,42 L9,48 L13,47 L14,37 Z` |
| Right forearm | `M51,36 L53,42 L51,48 L47,47 L46,37 Z` |
| Left thigh | `M19.5,48 L26.5,48 L26,63 L20,63 Z` |
| Right thigh | `M33.5,48 L40.5,48 L40,63 L34,63 Z` |
| Left calf | `M20,63 L26,63 L24.5,68 L21,68 Z` |
| Right calf | `M34,63 L40,63 L38.5,68 L35,68 Z` |

Fill: `#1c1f2a` · Stroke: `#2a2f42` · Stroke-width: 0.6–0.9 (thicker on torso)

## Highlight System

### MMAP change

From `{x, y, w, h}` → `{d: "svg path string"}`.

Each path is a single SVG path `d` attribute. Bilateral muscles (chest, shoulders, etc.) use two subpaths in one `d` string (two `M` commands).

### Highlight rendering

`muscleSVG()` changes from:
```js
`<rect x="${p.x}" y="${p.y}" width="${p.w}" height="${p.h}" rx="3" fill="#fb923c" opacity=".75"/>`
```
to:
```js
`<path d="${p.d}" fill="#fb923c" opacity=".75" filter="url(#mg)"/>`
```

### Glow filter

Added once inside the SVG:
```html
<filter id="mg" x="-30%" y="-30%" width="160%" height="160%">
  <feGaussianBlur in="SourceGraphic" stdDeviation="1.5" result="blur"/>
  <feMerge>
    <feMergeNode in="blur"/>
    <feMergeNode in="SourceGraphic"/>
  </feMerge>
</filter>
```

Blur renders first (the halo), then the solid shape on top. Net effect: soft orange glow around each highlighted muscle.

### Muscle path definitions (all 16 groups)

| Group | Path `d` |
|-------|----------|
| chest | `M19,18 Q23,16 28.5,18 Q30,24 27,27 Q22,28 19.5,24 Z M31.5,18 Q37,16 41,18 Q41,24 40.5,27 Q36,28 30,24 Z` |
| shoulders | `M17,16 L10,19 L11,26 L16,25 Z M43,16 L50,19 L49,26 L44,25 Z` |
| biceps | `M10,22 L9,32 L13,33 L14,25 Z M50,22 L51,32 L47,33 L46,25 Z` |
| triceps | `M13,22 L11,33 L14,37 L17,34 L18,24 Z M47,22 L49,33 L46,37 L43,34 L42,24 Z` |
| upper back | `M18,16 L42,16 L41,24 Q36,27 30,27 Q24,27 19,24 Z` |
| lats | `M19,22 L17,32 L19,40 L22,41 L21,30 Z M41,22 L43,32 L41,40 L38,41 L39,30 Z` |
| core | `M25.5,26 L34.5,26 L34,40 Q30,41.5 26,40 Z` |
| glutes | `M20,42 L40,42 L41.5,49 L18.5,49 Z` |
| quads | `M19.5,49 Q18.5,55 20,62 L26,62 Q27.5,55 26.5,49 Z M33.5,49 Q33,55 34,62 L40,62 Q41,55 40.5,49 Z` |
| hamstrings | `M20,49 Q18,57 20.5,62 L25.5,62 Q27,57 26.5,49 Z M34,49 Q33.5,57 34.5,62 L39.5,62 Q41,57 40,49 Z` |
| calves | `M20,62 L26,62 L24.5,68 L21,68 Z M34,62 L40,62 L38.5,68 L35,68 Z` |
| hips | `M19,40 L41,40 L40.5,47 L19.5,47 Z` |
| ankles | `M21,66 L25,66 L24.5,68 L21.5,68 Z M35,66 L39,66 L38.5,68 L35.5,68 Z` |
| full body | `M17,16 L43,16 L41,40 Q38,44 30,44 Q22,44 19,40 Z M21,43 L39,43 L40.5,48 L19.5,48 Z M17,16 L10,19 L9,36 L14,37 L18,24 Z M43,16 L50,19 L51,36 L46,37 L42,24 Z M9,36 L7,42 L9,48 L13,47 L14,37 Z M51,36 L53,42 L51,48 L47,47 L46,37 Z M19.5,48 L26.5,48 L26,63 L20,63 Z M33.5,48 L40.5,48 L40,63 L34,63 Z M20,63 L26,63 L24.5,68 L21,68 Z M34,63 L40,63 L38.5,68 L35,68 Z` — all body regions as subpaths |
| forearms | `M9,36 L7,42 L9,48 L13,47 L14,37 Z M51,36 L53,42 L51,48 L47,47 L46,37 Z` |
| traps | `M22,10 L38,10 L40,17 L20,17 Z` |

### Frontal-view limitation

Single frontal view only. Back-surface muscles (hamstrings, triceps, upper back) highlight the same region as their front counterparts. This matches the current behavior — no change to this constraint.

## Implementation

One file: `index.html`

1. **MMAP const** (line 157): replace all 16 `{x,y,w,h}` entries with `{d:"..."}` path strings
2. **`muscleSVG()` body**: replace rect-based body paths with the Refined Athletic paths above
3. **`muscleSVG()` highlight loop**: change `<rect>` to `<path d="${p.d}">` + add glow filter definition
4. **No other changes**: `renderExercise()`, `DAYS`, `MDB` untouched
