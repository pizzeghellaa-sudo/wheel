# Wheel of Fortune Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-file browser-based Wheel of Fortune game with configurable segments, GSAP-powered spin animation, and a zoom-in winner overlay, deployable to Vercel as a static site.

**Architecture:** Everything lives in `index.html` — HTML structure, CSS styles, and JavaScript logic in one file. A `CONFIG` object at the top of the `<script>` block is the only thing non-technical users need to touch. GSAP (loaded from CDN) drives both the spin deceleration and the winner overlay animation.

**Tech Stack:** Vanilla HTML5 / CSS3 / JavaScript (ES2020), Canvas 2D API, GSAP 3 (CDN), Vercel static hosting.

---

## File Map

| File | Purpose |
|------|---------|
| `index.html` | Entire app — layout, styles, CONFIG, wheel rendering, spin logic, winner effect |
| `vercel.json` | Tells Vercel to serve `index.html` for all routes |

---

## Task 1: Scaffold — project files and git

**Files:**
- Create: `index.html`
- Create: `vercel.json`

- [ ] **Step 1: Create `vercel.json`**

```json
{ "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }] }
```

- [ ] **Step 2: Create `index.html` with boilerplate only**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Wheel of Fortune</title>
</head>
<body>
  <p>Scaffold OK</p>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/gsap.min.js"></script>
  <script>
    console.log('GSAP loaded:', typeof gsap !== 'undefined');
  </script>
</body>
</html>
```

- [ ] **Step 3: Verify in browser**

Open `index.html` directly in a browser (double-click or `file://` URL).
Expected: page shows "Scaffold OK". Open DevTools console — should print `GSAP loaded: true`.

- [ ] **Step 4: Git init and first commit**

```bash
cd C:/Data/Developers/wheel
git init
git add index.html vercel.json
git commit -m "chore: scaffold — empty HTML boilerplate + vercel config"
```

---

## Task 2: CONFIG object + HTML structure + CSS

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Replace `index.html` with full structure**

Replace the entire file content with:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Wheel of Fortune</title>
  <style>
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      background: #1a1a2e;
      color: #fff;
      font-family: 'Segoe UI', sans-serif;
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
    }

    .page {
      width: 100%;
      max-width: 900px;
      padding: 24px 16px;
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 24px;
    }

    h1 {
      font-size: 2rem;
      letter-spacing: 2px;
      color: #e0d7ff;
    }

    /* ── Game row: wheel + winner panel ── */
    .game {
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 40px;
      flex-wrap: wrap;
    }

    /* ── Wheel column ── */
    .wheel-section {
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 0;
    }

    /* ── Pointer triangle ── */
    .pointer {
      width: 24px;
      height: 20px;
      clip-path: polygon(50% 100%, 0% 0%, 100% 0%);
      margin-bottom: 4px;
      /* background-color set from CONFIG.pointerColor via JS */
    }

    canvas#wheel {
      display: block;
      max-width: 100%;
    }

    /* ── Spin button ── */
    #spinBtn {
      margin-top: 24px;
      padding: 12px 36px;
      border: none;
      border-radius: 999px;
      background: #7c3aed;
      color: #fff;
      font-size: 1rem;
      font-weight: 700;
      letter-spacing: 2px;
      cursor: pointer;
      transition: background 0.2s, opacity 0.2s;
    }
    #spinBtn:hover:not(:disabled) { background: #6d28d9; }
    #spinBtn:disabled { opacity: 0.5; cursor: not-allowed; }

    /* ── Winner panel ── */
    .winner-panel {
      width: 180px;
      min-height: 100px;
      border-radius: 12px;
      display: flex;
      align-items: center;
      justify-content: center;
      background: #2a2a4e;
      transition: background 0.4s, color 0.4s;
    }
    #winnerName {
      font-size: 1.8rem;
      font-weight: 700;
    }

    /* ── Winner overlay ── */
    .overlay {
      position: fixed;
      inset: 0;
      background: rgba(0, 0, 0, 0.75);
      display: flex;
      align-items: center;
      justify-content: center;
      opacity: 0;
      pointer-events: none;
      z-index: 100;
      transition: opacity 0.3s;
    }
    .overlay.active {
      opacity: 1;
      pointer-events: all;
    }
    .overlay-content {
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 12px;
      transform: scale(0);
    }
    .overlay-name {
      font-size: 4rem;
      font-weight: 900;
      padding: 16px 40px;
      border-radius: 16px;
      /* background and color set from winner segment via JS */
    }
    .overlay-label {
      font-size: 1.4rem;
      color: #fde68a;
    }

    /* ── Responsive ── */
    @media (max-width: 600px) {
      canvas#wheel { width: 300px; height: 300px; }
      .winner-panel { width: 140px; min-height: 80px; }
      #winnerName { font-size: 1.3rem; }
    }
  </style>
</head>
<body>

  <div class="page">
    <h1>Wheel of Fortune</h1>
    <div class="game">

      <div class="wheel-section">
        <div class="pointer" id="pointer"></div>
        <canvas id="wheel" width="500" height="500"></canvas>
        <button id="spinBtn">SPIN!</button>
      </div>

      <div class="winner-panel" id="winnerPanel">
        <span id="winnerName">?</span>
      </div>

    </div>
  </div>

  <!-- Winner overlay -->
  <div class="overlay" id="overlay">
    <div class="overlay-content" id="overlayContent">
      <div class="overlay-name" id="overlayName"></div>
      <div class="overlay-label">🏆 Winner!</div>
    </div>
  </div>

  <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/gsap.min.js"></script>
  <script>
    // ─────────────────────────────────────────────
    // CONFIG — edit this block to customise the game
    // ─────────────────────────────────────────────
    const CONFIG = {
      segments: [
        { label: 'Andy',    color: '#a78bfa', textColor: '#1a1a2e' },
        { label: 'Petro',   color: '#f9a8d4', textColor: '#1a1a2e' },
        { label: 'Jason',   color: '#fde68a', textColor: '#1a1a2e' },
        { label: 'Joao',    color: '#86efac', textColor: '#1a1a2e' },
        { label: 'Matheus', color: '#f87171', textColor: '#ffffff' },
        { label: 'Robson',  color: '#818cf8', textColor: '#ffffff' },
        { label: 'Qazi',    color: '#d1a0a0', textColor: '#1a1a2e' },
        { label: 'Fellipe', color: '#84a98c', textColor: '#ffffff' },
      ],
      minSpins:      5,            // minimum full rotations before stopping
      maxSpins:      10,           // maximum full rotations
      spinDuration:  4,            // seconds for the spin animation
      pointerColor:  '#000000',    // color of the pointer triangle
      spinDirection: 'clockwise',  // 'clockwise' or 'counter-clockwise'
    };
    // ─────────────────────────────────────────────

    console.log('CONFIG loaded, segments:', CONFIG.segments.length);
  </script>
</body>
</html>
```

- [ ] **Step 2: Verify in browser**

Open `index.html`.
Expected:
- Dark page with "Wheel of Fortune" title
- Black pointer triangle visible
- Empty canvas area (white/transparent rectangle)
- Dimmed winner panel showing "?"
- Purple "SPIN!" button
- DevTools console prints `CONFIG loaded, segments: 8`

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add CONFIG, HTML layout, and CSS styles"
```

---

## Task 3: Wheel drawing — static render

**Files:**
- Modify: `index.html` — add `drawWheel()` inside `<script>`, call it on load

- [ ] **Step 1: Add wheel drawing code**

Inside the `<script>` block, after the CONFIG block, add:

```js
// ── DOM refs ──
const canvas   = document.getElementById('wheel');
const ctx      = canvas.getContext('2d');
const pointer  = document.getElementById('pointer');
const spinBtn  = document.getElementById('spinBtn');
const winnerPanel = document.getElementById('winnerPanel');
const winnerName  = document.getElementById('winnerName');
const overlay     = document.getElementById('overlay');
const overlayContent = document.getElementById('overlayContent');
const overlayName    = document.getElementById('overlayName');

// Apply pointer color from CONFIG
pointer.style.backgroundColor = CONFIG.pointerColor;

// ── Wheel state ──
let currentAngle = 0;

// ── Helper: positive modulo ──
function mod(a, n) { return ((a % n) + n) % n; }

// ── Draw the wheel at a given rotation angle (radians) ──
function drawWheel(angle) {
  const { segments } = CONFIG;
  const N = segments.length;
  const segAngle = (2 * Math.PI) / N;
  const cx = canvas.width  / 2;
  const cy = canvas.height / 2;
  const radius = Math.min(cx, cy) - 4;

  ctx.clearRect(0, 0, canvas.width, canvas.height);

  segments.forEach((seg, i) => {
    const startAngle = angle + i * segAngle - Math.PI / 2;
    const endAngle   = startAngle + segAngle;

    // Segment fill
    ctx.beginPath();
    ctx.moveTo(cx, cy);
    ctx.arc(cx, cy, radius, startAngle, endAngle);
    ctx.closePath();
    ctx.fillStyle = seg.color;
    ctx.fill();

    // Thin white divider line
    ctx.strokeStyle = '#ffffff';
    ctx.lineWidth = 1.5;
    ctx.stroke();

    // Label text — rotated outward from center
    ctx.save();
    ctx.translate(cx, cy);
    ctx.rotate(startAngle + segAngle / 2);
    ctx.textAlign = 'right';
    ctx.textBaseline = 'middle';
    ctx.fillStyle = seg.textColor;
    // Font size: fraction of arc width at 70% radius, clamped 10–24px
    const arcWidth = 2 * (radius * 0.7) * Math.sin(segAngle / 2);
    const fontSize = Math.max(10, Math.min(24, arcWidth * 0.45));
    ctx.font = `bold ${fontSize}px 'Segoe UI', sans-serif`;
    ctx.fillText(seg.label, radius - 12, 0);
    ctx.restore();
  });

  // Center dot
  ctx.beginPath();
  ctx.arc(cx, cy, 8, 0, 2 * Math.PI);
  ctx.fillStyle = '#000000';
  ctx.fill();
}

// Draw initial static wheel
drawWheel(currentAngle);
```

- [ ] **Step 2: Verify in browser**

Reload `index.html`.
Expected:
- The wheel canvas shows all 8 coloured segments with names
- Each name reads outward from center toward the rim
- A small black dot is visible at the center
- The black pointer triangle is visible above the canvas
- The winner panel still shows "?"

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: implement static wheel drawing with Canvas 2D"
```

---

## Task 4: Spin mechanics — GSAP animation + winner determination

**Files:**
- Modify: `index.html` — add `spin()` function and wire up the SPIN button

- [ ] **Step 1: Add spin logic**

Inside the `<script>` block, after `drawWheel(currentAngle)`, add:

```js
// ── Spin state ──
let spinning = false;

// ── Spin ──
function spin() {
  if (spinning) return;
  spinning = true;
  spinBtn.disabled = true;

  const { segments, minSpins, maxSpins, spinDuration, spinDirection } = CONFIG;
  const N         = segments.length;
  const segAngle  = (2 * Math.PI) / N;
  const direction = spinDirection === 'clockwise' ? 1 : -1;

  // Pick winner before animation starts
  const winnerIndex  = Math.floor(Math.random() * N);
  // Angle of winner segment's center in wheel's local frame (before rotation)
  const winnerCenter = winnerIndex * segAngle + segAngle / 2;

  // The angle value at which the winning segment's center is under the pointer (top = -π/2):
  //   currentAngle + winnerCenter - π/2 = 0  →  finalAngle = π/2 - winnerCenter
  const finalAngleBase = Math.PI / 2 - winnerCenter;

  // Random number of full rotations in [minSpins, maxSpins)
  const fullSpins = Math.floor(minSpins + Math.random() * (maxSpins - minSpins));

  // Extra rotation needed beyond full spins to land on winner
  let extra;
  if (direction === 1) {
    // Clockwise (positive): extra must be in [0, 2π)
    extra = mod(finalAngleBase - mod(currentAngle, 2 * Math.PI), 2 * Math.PI);
  } else {
    // Counter-clockwise (negative): extra must be in [0, 2π), then subtract
    extra = mod(mod(currentAngle, 2 * Math.PI) - finalAngleBase, 2 * Math.PI);
  }

  const targetAngle = currentAngle + direction * (fullSpins * 2 * Math.PI) + direction * extra;

  // GSAP tween via proxy object
  const proxy = { val: currentAngle };
  gsap.to(proxy, {
    val:      targetAngle,
    duration: spinDuration,
    ease:     'power4.out',
    onUpdate() {
      currentAngle = proxy.val;
      drawWheel(currentAngle);
    },
    onComplete() {
      currentAngle = targetAngle;
      spinning = false;
      spinBtn.disabled = false;
      showWinner(winnerIndex);
    },
  });
}

spinBtn.addEventListener('click', spin);
```

- [ ] **Step 2: Add a temporary `showWinner` stub so the button works without errors**

Directly after the spin function add:

```js
// Temporary stub — replaced in Task 5
function showWinner(index) {
  console.log('Winner:', CONFIG.segments[index].label);
}
```

- [ ] **Step 3: Verify in browser**

Reload and click SPIN! several times.
Expected:
- Wheel spins with a fast start and smooth deceleration
- Console prints the winner's name after each spin
- The SPIN button is greyed out while spinning, re-enables after
- Spinning again after a stop continues from the stopped position (wheel doesn't reset to 0)
- Try setting `spinDirection: 'counter-clockwise'` in CONFIG, reload — wheel should spin the other way and still land on a valid segment

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: implement GSAP spin mechanics with winner pre-selection and direction support"
```

---

## Task 5: Winner effect — overlay + winner panel

**Files:**
- Modify: `index.html` — replace `showWinner` stub with full implementation

- [ ] **Step 1: Replace the stub with the real `showWinner` function**

Remove the temporary stub and replace it with:

```js
// ── Winner effect ──
let dismissTimer = null;

function showWinner(index) {
  const seg = CONFIG.segments[index];

  // 1. Update winner panel (right side box)
  winnerPanel.style.background = seg.color;
  winnerName.style.color = seg.textColor;
  winnerName.textContent = seg.label;

  // 2. Populate overlay name card with segment colors
  overlayName.textContent = seg.label;
  overlayName.style.background = seg.color;
  overlayName.style.color = seg.textColor;

  // 3. Fade in overlay
  overlay.classList.add('active');

  // 4. Zoom in overlay content with GSAP Back.easeOut
  gsap.fromTo(
    overlayContent,
    { scale: 0, opacity: 0 },
    { scale: 1, opacity: 1, duration: 0.6, ease: 'back.out(1.7)' }
  );

  // 5. Auto-dismiss after 4 seconds
  if (dismissTimer) clearTimeout(dismissTimer);
  dismissTimer = setTimeout(dismissOverlay, 4000);
}

function dismissOverlay() {
  if (dismissTimer) { clearTimeout(dismissTimer); dismissTimer = null; }
  gsap.to(overlayContent, {
    scale: 0,
    opacity: 0,
    duration: 0.3,
    ease: 'power2.in',
    onComplete() {
      overlay.classList.remove('active');
      // Reset scale for next time
      gsap.set(overlayContent, { scale: 0, opacity: 0 });
    },
  });
}

overlay.addEventListener('click', dismissOverlay);
```

- [ ] **Step 2: Verify in browser**

Reload and click SPIN!
Expected:
- After the wheel stops, the winner panel on the right fills with the winning segment's color and shows the winner's name in the correct text color
- A dark semi-transparent overlay appears over the whole page
- The winner's name zooms in with a satisfying bounce (Back.easeOut)
- "🏆 Winner!" appears below the name in gold
- Clicking anywhere on the overlay dismisses it with a quick zoom-out
- After ~4 seconds without clicking, the overlay auto-dismisses
- Spinning again after dismissal works correctly and the winner panel updates each time

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: winner overlay with GSAP zoom-in and winner panel color update"
```

---

## Task 6: Final polish and Vercel deployment

**Files:**
- Modify: `index.html` — minor polish
- `vercel.json` — already created in Task 1

- [ ] **Step 1: Add page title and meta description for Vercel**

Inside `<head>`, after the `<title>` tag, add:

```html
<meta name="description" content="Spin the wheel and discover the winner!" />
<link rel="icon" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><text y='.9em' font-size='90'>🎡</text></svg>" />
```

- [ ] **Step 2: Verify full game flow one more time**

Reload `index.html`. Run through this checklist manually:
- [ ] Pointer triangle visible above wheel at 12 o'clock
- [ ] All 8 segment names and colors render correctly
- [ ] SPIN! button spins the wheel with smooth ease-out deceleration
- [ ] Winner segment lands under the pointer after each spin
- [ ] Winner panel stays "?" during spin, updates after wheel stops
- [ ] Overlay zooms in with winner name in correct segment colors
- [ ] Overlay dismisses on click and auto-dismisses after 4 seconds
- [ ] Changing `spinDirection` to `'counter-clockwise'` in CONFIG and reloading works
- [ ] Changing the segments array (add/remove entries) works without code changes
- [ ] Page looks reasonable at narrow viewport width (≤600px)

- [ ] **Step 3: Final commit**

```bash
git add index.html
git commit -m "feat: add favicon and meta description, wheel of fortune complete"
```

- [ ] **Step 4: Deploy to Vercel**

```bash
# Option A — Vercel CLI (if installed)
npx vercel

# Option B — via GitHub
# 1. Create a GitHub repo and push:
git remote add origin <your-github-repo-url>
git push -u origin main   # or 'master' depending on your default branch

# 2. Go to vercel.com → New Project → Import your GitHub repo
# 3. Leave all settings at defaults (no framework, no build command)
# 4. Click Deploy
```

Expected: Vercel gives you a public URL (e.g. `https://wheel-of-fortune-xyz.vercel.app`). Open it and verify the full game flow works online.

---

## Self-Review Notes

Spec requirements covered:

| Requirement | Task |
|---|---|
| Configurable segments (label, background color, text color) | Task 2 CONFIG |
| Configurable number of segments (length of array) | Task 2 CONFIG |
| Configurable min/max spins and duration | Task 2 CONFIG |
| Configurable spin direction (clockwise / counter-clockwise) | Task 4 |
| Configurable pointer color | Task 3 |
| Canvas-rendered wheel with text outward | Task 3 |
| Fixed pointer triangle at 12 o'clock | Task 2 HTML/CSS |
| GSAP Power4.easeOut deceleration | Task 4 |
| Winner determined before animation | Task 4 |
| Winner lands under pointer | Task 4 |
| Winner panel updates only after wheel stops | Task 5 |
| Full-screen overlay with zoom-in (Back.easeOut) | Task 5 |
| Overlay shows winner name in segment colors | Task 5 |
| Overlay auto-dismiss (4s) and click-dismiss | Task 5 |
| Vercel static deployment | Task 1 + Task 6 |
| Responsive layout | Task 2 CSS |
