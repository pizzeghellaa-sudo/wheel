# Wheel of Fortune

A browser-based Wheel of Fortune game. Single-page, frontend-only, static site deployed on Vercel. No build step, no framework, no dependencies beyond GSAP loaded from CDN.

## File Structure

```
index.html    — entire app (HTML + CSS + JS in one file)
vercel.json   — Vercel static site config (SPA rewrite rule)
.gitignore    — excludes .claude/, *.png, .superpowers/
```

Design spec: `docs/superpowers/specs/2026-04-10-wheel-of-fortune-design.md`  
Implementation plan: `docs/superpowers/plans/2026-04-10-wheel-of-fortune.md`

## How to Run

Open `index.html` directly in a browser — no server needed.

## How to Deploy

Push to GitHub, connect the repo to Vercel. It auto-detects a static site. No build command, no environment variables.

## How to Configure

Edit only the `CONFIG` object at the top of the `<script>` block in `index.html`:

```js
const CONFIG = {
  segments: [
    { label: 'Andy',    color: '#a78bfa', textColor: '#1a1a2e' },
    // add/remove entries freely — segment count is driven by array length
  ],
  minSpins:      5,            // minimum full rotations before stopping
  maxSpins:      10,           // maximum full rotations
  spinDuration:  4,            // seconds for the spin animation
  pointerColor:  '#000000',    // color of the pointer triangle above the wheel
  spinDirection: 'clockwise',  // 'clockwise' or 'counter-clockwise'
};
```

Each segment:
- `label` — name shown on the segment and in the winner overlay
- `color` — background fill color of the segment (hex)
- `textColor` — text color on that segment (hex) — use dark for light backgrounds, light for dark

## Architecture

Everything is in `index.html`. The script block is structured as:

1. `CONFIG` — the only block users need to touch
2. DOM refs — all element references captured once on load
3. `mod(a, n)` — positive modulo helper (used in spin angle math)
4. `drawWheel(angle)` — Canvas 2D rendering function, called every animation frame
5. `spin()` — picks winner, computes target angle, runs GSAP tween
6. `showWinner(index)` + `dismissOverlay()` — winner overlay logic

## Key Implementation Details

**Wheel rendering:**
- Canvas 2D API, 500×500 logical pixels (CSS-scaled to 300×300 on narrow screens)
- Segment 0 starts at 12 o'clock (`-Math.PI/2` offset in draw)
- Font size = `arcWidth * 0.45`, clamped 10–24px — scales automatically with segment count
- Pointer is an HTML `<div>` with `clip-path: polygon(50% 100%, 0% 0%, 100% 0%)`, colored from `CONFIG.pointerColor`

**Spin mechanics:**
- Winner is chosen with `Math.random()` **before** the animation starts
- Target angle formula: the pointer is at canvas angle `-π/2` (12 o'clock)
  - `finalAngleBase = -winnerCenter` where `winnerCenter = winnerIndex * segAngle + segAngle/2`
  - `extra = mod(finalAngleBase - mod(currentAngle, 2π), 2π)` for clockwise
  - `targetAngle = currentAngle + direction * (fullSpins * 2π + extra)`
- GSAP proxy pattern: `gsap.to({ val: currentAngle }, { val: targetAngle, ease: 'power4.out', ... })`
- `currentAngle` accumulates across spins (never reset to 0) — feels natural

**Winner effect:**
- Panel (right side) updates with segment `color`/`textColor`/`label` only in `onComplete` (after wheel stops)
- Overlay: CSS fade-in via `.overlay.active`, content zooms in with `gsap.fromTo(..., { ease: 'back.out(1.7)' })`
- Dismiss: click anywhere or auto-dismiss after 4 seconds
- `gsap.killTweensOf(overlayContent)` called before any new dismiss tween to prevent double-dismiss race

## CDN Dependencies

- **GSAP 3.12.5** — `https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/gsap.min.js`
  - Used for spin deceleration (`power4.out`) and winner zoom-in (`back.out(1.7)`)
  - Requires internet connection (no offline support)

## Known Limitations / Future Work

- No CONFIG validation — empty `segments` array will crash on first spin
- Spin button can be clicked again while winner overlay is visible (starts a new spin under the overlay)
- No SRI hash on the GSAP CDN script tag
- No accessibility: no `aria-live` region for the winner announcement
