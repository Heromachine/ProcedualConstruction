# Procedural Construction — Project Plan

Inspired by Oscar Ståhlberg's work on *Townscaper* and *Bad North*.
The goal: infinite, handcrafted-feeling environments via a **dual grid + tile-matching system**.

---

## Core Concept

Traditional procedural generation = unpredictable, soulless output.
Ståhlberg's insight: **constrain the algorithm so every output is hand-authored**.

- A small set of hand-made tile models covers ALL possible states
- The algorithm only decides *which* tile to place, never *what* it looks like
- Players (or code) drive inputs → the system guarantees beautiful outputs

---

## The Dual Grid System (Key Insight)

### Problem with a regular grid
- Each cell is either land or water
- 256 neighbor configurations → even after rotation/mirror deduplication → **15 distinct models**
- Convex corners OR concave corners can be round, but not both

### Solution: Dual Grid
- **Main grid** — points that the user clicks (land or water per point)
- **Dual grid** — offset by half a cell; each cell's appearance is driven by its 4 corner points
- Because type is defined **per corner** (not per cell), there are only 16 combinations
- After rotation/mirror deduplication → **6 distinct tile shapes**
- Both convex AND concave corners can be round ✓

```
Main grid points:       Dual grid cells:
·  ·  ·  ·            [ ][ ][ ]
·  ·  ·  ·            [ ][ ][ ]
·  ·  ·  ·            [ ][ ][ ]
·  ·  ·  ·

Each dual cell's 4 corners = 4 surrounding main grid points
```

### The 6 Tile Types (bit encoding: TL=8, TR=4, BR=2, BL=1)

| Type | Bits | Description              | Shape       |
|------|------|--------------------------|-------------|
| 0    | 0000 | All water                | Empty       |
| 1    | 0001 | One corner land          | Quarter arc |
| 2    | 0011 | Two adjacent corners     | Half + curve|
| 3    | 0101 | Two diagonal corners     | Two arcs    |
| 4    | 0111 | Three corners land       | Concave cut |
| 5    | 1111 | All land                 | Full fill   |

The remaining 10 of 16 cases are rotations of the above 6.

---

## Phases

### ✅ Phase 1 — Dual Grid Foundation (CURRENT)
**File:** `index.html`

- [x] Main grid of boolean points (land / water)
- [x] Dual grid rendering — all 16 tile types drawn with arcs & bezier curves
- [x] Mouse interaction — click & drag to paint land, click again to remove
- [x] Clear button
- [x] Visual polish (shading, subtle highlights)

**Tech:** Vanilla HTML5 Canvas, no dependencies.

---

### ✅ Phase 2 — Tile Variants & Randomization
**File:** `phase2.html`

- [x] 4 named variants per tile (Forest, Meadow, Deep, Scrub)
- [x] Per-variant bezier bulge factor `b` — controls curve roundness
- [x] Per-variant arc radius factor `r` — controls corner size
- [x] Per-variant land color + edge highlight color
- [x] Per-cell seeded RNG (`XorShift32`) for stable, reproducible detail placement
- [x] Detail dot pass — random dots clipped to land shape via `ctx.clip()`
- [x] Re-seed on placement — new land configs always get a fresh variant
- [x] Randomize button — re-seeds all existing land cells
- [x] Main menu + back navigation added

**Tech:** Vanilla HTML5 Canvas, no dependencies.

---

### ✅ Phase 3 — Organic Grid
**File:** `phase3.html`

- [x] Jittered interior grid points (border points stay fixed on canvas edges)
- [x] 5-iteration Laplacian relaxation (each point → avg of 4 neighbors)
- [x] Quad-based path builder — all 16 tile types via `quadraticCurveTo` with actual corner coords
- [x] Edge midpoints computed from real point positions (not assumed square)
- [x] Water background per-cell fills the actual quad polygon (not fillRect)
- [x] Brute-force nearest-point mouse detection for irregular grid
- [x] Generate button — new random organic layout, clears land
- [x] Variant color system + seeded detail dots carried forward from Phase 2
- [x] Grid overlay draws actual quad outlines

**Tech:** Vanilla HTML5 Canvas, no dependencies.

---

### ✅ Phase 4 — Multi-tile Special Pieces
**File:** `phase4.html`

- [x] `findSpecialPieces()` scans for patterns each render, priority order (largest first)
- [x] **Mountain** — 2×2 block of type-15 cells → gradient dome with shadow + highlight
- [x] **Horizontal Ridge** — 1×3 strip → elongated gradient ellipse
- [x] **Vertical Ridge** — 3×1 strip → same visual, transposed proportions
- [x] `claimed[r][c]` grid — prevents double-rendering; claimed cells skip land/dots/edge pass
- [x] Special pieces drawn in a dedicated second pass, before grid overlay
- [x] Piece geometry computed from actual organic grid point positions (adapts to jittered grid)
- [x] Radial gradients + shadow layer + specular highlight per piece type

**Tech:** Vanilla HTML5 Canvas, no dependencies.

---

### ✅ Phase 5 — Wave Function Collapse (Autonomous Generation)
**File:** `phase5.html`

- [x] No player painting — algorithm runs autonomously on page load
- [x] `initWFC()` — pins border points to water, applies center-bias weight falloff to interior points
- [x] Center seed — anchors island by collapsing center point to LAND first
- [x] `propagateWFC(r, c, value)` — cardinal neighbors +0.24/×0.76 influence, diagonal ±0.12
- [x] `collapseOne()` — selects minimum-entropy point (highest |weight−0.5| + noise), collapses probabilistically
- [x] `renderWFC()` — ghost tint visualization during collapse (green→likely land, blue→likely water)
- [x] `tick()` / `startWFC()` — RAF animation loop, configurable speed (1×/5×/20×)
- [x] `#wfc-status` indicator — shows remaining points during collapse, final mountain/ridge count after
- [x] **Generate** button — new organic grid + new WFC run
- [x] **Step** button — advances one collapse step manually (good for slow inspection)
- [x] **Pause/Resume** button — halts/resumes the animation
- [x] **Speed** toggle — cycles between 1×, 5×, 20× steps per frame
- [x] Special pieces (mountains, ridges) rendered automatically after WFC completes
- [x] All Phase 4 rendering inherited: variants, detail dots, edge strokes, gradient domes

**Tech:** Vanilla HTML5 Canvas, no dependencies.

---

## Key References

- **Oscar Ståhlberg** — Creator of Townscaper & Bad North
- **Townscaper** (2021) — Player-driven island builder
- **Bad North** (2018) — Auto-generated islands via WFC
- **Wave Function Collapse** — Maxim Gumin (2016), based on Paul Merrell's Model Synthesis (2007)

---

## File Structure

```
Procedual Construction/
├── PLAN.md          ← This file
├── index.html       ← Main menu
├── phase1.html      ← Phase 1: Dual grid canvas
├── phase2.html      ← Phase 2: Tile variants & detail pass
├── phase3.html      ← (future) Organic grid
├── phase4.html      ← Phase 4: Multi-tile special pieces
└── phase5.html      ← Phase 5: Wave Function Collapse
```
