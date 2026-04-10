# Wheel of Fortune — Design Spec
**Date:** 2026-04-10  
**Status:** Approved

---

## Overview

A browser-based Wheel of Fortune game. Single-page, frontend-only, deployed as a static site on Vercel. No build step. Configuration is entirely driven by a JavaScript object at the top of the file — non-technical users edit only that block.

---

## File Structure

```
index.html        — entire app (HTML + CSS + JS in one file)
vercel.json       — static site config for Vercel
```

No framework, no build pipeline. Vercel serves `index.html` directly.

`vercel.json`:
```json
{ "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }] }
```

---

## Configuration Object

Located at the top of the `<script>` block in `index.html`. Users edit only this section.

```js
const CONFIG = {
  segments: [
    { label: "Andy",    color: "#a78bfa", textColor: "#1a1a2e" },
    { label: "Petro",   color: "#f9a8d4", textColor: "#1a1a2e" },
    { label: "Jason",   color: "#fde68a", textColor: "#1a1a2e" },
    { label: "Joao",    color: "#86efac", textColor: "#1a1a2e" },
    { label: "Matheus", color: "#f87171", textColor: "#ffffff" },
    { label: "Robson",  color: "#818cf8", textColor: "#ffffff" },
    { label: "Qazi",    color: "#d1a0a0", textColor: "#1a1a2e" },
    { label: "Fellipe", color: "#84a98c", textColor: "#ffffff" },
  ],
  minSpins: 5,          // minimum full rotations before stopping
  maxSpins: 10,         // maximum full rotations
  spinDuration: 4,      // seconds for the full spin animation
  pointerColor: "#000000",  // color of the pointer triangle
  spinDirection: "clockwise", // "clockwise" or "counter-clockwise"
};
```

Each segment has:
- `label` — the name displayed on the segment
- `color` — background fill color of the segment
- `textColor` — color of the label text on that segment

The number of segments is determined by the length of the `segments` array (no separate count field needed).

---

## Layout

Classic layout. Max-width ~900px, centered on the page. Dark background.

```
┌─────────────────────────────────────────────┐
│           Wheel of Fortune                  │
│                                             │
│           ▼  (pointer)                      │
│   [ WHEEL CANVAS ]    [ Winner Panel ]      │
│                           ?  →  Andy        │
│         [ SPIN! ]                           │
└─────────────────────────────────────────────┘
```

- **Pointer:** small downward-pointing triangle centered above the wheel at 12 o'clock, outside the canvas edge. Implemented as an HTML `<div>` with CSS `clip-path: polygon(50% 100%, 0 0, 100% 0)`. Its fill color is driven by `CONFIG.pointerColor`.
- **Center dot:** small black filled circle drawn on the canvas at the wheel's center point.
- **No decorative ring** around the wheel outer edge.
- **Winner panel:** sits to the right of the wheel. Shows `?` on load, stays `?` during spin, updates with winner's name (in that segment's `color`/`textColor`) only after the wheel fully stops.
- **SPIN button:** pill-shaped, styled with segment-inspired purple. Disabled during animation, re-enabled after.
- Responsive: wheel canvas scales down on smaller viewports.

---

## Wheel Rendering

Rendered on a `<canvas>` element using the Canvas 2D API.

- Each segment is drawn as an arc slice: `ctx.arc()` for the pie wedge, filled with `segment.color`
- Segment angle = `(2 * Math.PI) / segments.length` (equal slices)
- Label text: rotated to read outward from center toward the rim, drawn in `segment.textColor`, bold. Font size is calculated as a fraction of the segment arc width at the midpoint radius (e.g. `arcWidth * 0.18`), clamped to a min/max range so text stays legible across segment counts of 2–20.
- Center dot: small filled black circle drawn last (on top of all segments)
- The entire wheel rotates by updating a `currentAngle` value and redrawing on each animation frame

---

## Spin Mechanics

1. **Winner selection:** determined randomly at spin start (before animation begins). The user never sees this pre-selection.
2. **Target angle calculation:** compute the rotation needed so the winning segment lands centered under the pointer (top, 12 o'clock). Direction is applied as a sign multiplier: `+1` for clockwise, `-1` for counter-clockwise.
   - `targetAngle = currentAngle + direction * ((minSpins + rand) * 2π + offsetToWinner)`
3. **Animation:** GSAP tweens `currentAngle` from its current value to `targetAngle` using `Power4.easeOut` over `spinDuration` seconds. The canvas redraws on every GSAP `onUpdate` tick.
4. **Button state:** SPIN button disabled at animation start, re-enabled in GSAP `onComplete`.
5. **Repeat spins:** each spin adds to `currentAngle` (no reset to 0) so the wheel continues from where it stopped — feels natural.

---

## Winner Effect

Triggers in GSAP `onComplete` (when wheel fully stops):

1. **Winner panel** updates: the right-side box fills with the winning segment's `color` and displays the winner's `label` in `textColor`.
2. **Full-screen overlay** fades in (semi-transparent dark background).
3. **Winner name** zooms in center-screen: GSAP animates scale from 0 → 1.1 → 1.0 using `Back.easeOut`. Includes the name in large bold text and a "🏆 Winner!" sub-label.
4. **Dismiss:** overlay disappears on click anywhere, or auto-dismisses after 4 seconds.

---

## CDN Dependencies

- **GSAP** (GreenSock Animation Platform) — loaded from CDN. Used for spin deceleration (`Power4.easeOut`) and winner zoom-in (`Back.easeOut`).

No other external dependencies.

---

## Vercel Deployment

- Push repository to GitHub
- Connect repo to Vercel — it auto-detects a static site (no framework)
- `index.html` is served at the root
- No environment variables, no build command needed
