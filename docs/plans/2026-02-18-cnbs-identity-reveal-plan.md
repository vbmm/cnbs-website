# CNBS Identity Reveal Page — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a single `index.html` with an animated "CNBS" text reveal — frantic font/color cycling with burst+breathe rhythm, hover slowdown, scanline/noise overlays.

**Architecture:** Single HTML file. Google Fonts loaded via `<link>`. CSS handles layout, overlays, and transitions. JavaScript `requestAnimationFrame` loop drives the animation with a burst/breathe state machine.

**Tech Stack:** Vanilla HTML/CSS/JS, Google Fonts CDN. No build step. Deploy target: Cloudflare Pages.

---

### Task 1: HTML Shell + Layout + Google Fonts

**Files:**
- Create: `index.html`

**Step 1: Create the HTML file with boilerplate, Google Fonts link, and centered text**

The file should contain:

1. HTML5 doctype and `<html lang="en">`
2. `<head>` with:
   - `<meta charset="UTF-8">`
   - `<meta name="viewport" content="width=device-width, initial-scale=1.0">`
   - `<meta name="theme-color" content="#000000">`
   - `<title>CNBS</title>`
   - Single Google Fonts `<link>` loading all 18 fonts (bold weights, `display=swap`)
   - Inline `<style>` block with:
     - CSS reset: `*, *::before, *::after { margin: 0; padding: 0; box-sizing: border-box; }`
     - `html, body { width: 100%; height: 100%; overflow: hidden; background: #000; }`
     - `body { display: flex; justify-content: center; align-items: center; }`
     - `.cnbs` element: `font-size: clamp(4rem, 20vw, 25rem); font-weight: 900; color: #fff; text-transform: uppercase; user-select: none; cursor: default;`
3. `<body>` with a single `<div class="cnbs" id="cnbs">CNBS</div>`

**Font families to load (Google Fonts URL):**
Playfair Display:900, Cormorant Garamond:700, Bodoni Moda:900, Inter:900, Bebas Neue, Oswald:700, Unlock, Righteous, Monoton, Bungee Shade, Rubik Glitch, Rubik Wet Paint, Rubik Vinyl, Silkscreen:700, Honk, Nabla, Barlow Condensed:700, Space Mono:700, JetBrains Mono:700

**Step 2: Open in browser and verify**

Open `index.html` in browser. Should see: white "CNBS" text centered on black background, large and responsive. Resize window to confirm `clamp()` scaling works.

---

### Task 2: Scanline + Noise Overlays

**Files:**
- Modify: `index.html` (CSS `<style>` block)

**Step 1: Add scanline overlay**

Add to CSS:

```css
body::after {
  content: '';
  position: fixed;
  inset: 0;
  z-index: 10;
  pointer-events: none;
  background: repeating-linear-gradient(
    to bottom,
    rgba(255, 255, 255, 0.03) 0px,
    rgba(255, 255, 255, 0.03) 1px,
    transparent 1px,
    transparent 2px
  );
}
```

**Step 2: Add noise grain overlay**

Add an inline SVG filter definition inside the `<body>` (hidden, zero-size):

```html
<svg style="position:absolute;width:0;height:0">
  <filter id="noise">
    <feTurbulence type="fractalNoise" baseFrequency="0.65" numOctaves="3" stitchTiles="stitch"/>
  </filter>
</svg>
```

Add to CSS:

```css
body::before {
  content: '';
  position: fixed;
  inset: 0;
  z-index: 9;
  pointer-events: none;
  filter: url(#noise);
  opacity: 0.03;
}
```

**Step 3: Verify in browser**

Open `index.html`. Scanlines should be barely visible — faint horizontal stripes. Noise should be an extremely subtle static grain. Both should be non-interactive (clicks pass through).

---

### Task 3: Core Animation Engine (Burst Mode Only)

**Files:**
- Modify: `index.html` (add `<script>` before `</body>`)

**Step 1: Write the animation script**

Add a `<script>` tag before `</body>` with:

1. **Configuration constants:**
   - `BURST_INTERVAL = 70` (ms between ticks in burst mode)
   - Font array with all 18 font family strings
   - Color array with all 15 colors from the palette
   - `LETTER_SPACING_MIN = -0.05`, `LETTER_SPACING_MAX = 0.15` (in em)
   - `SIZE_SCALE_MIN = 0.95`, `SIZE_SCALE_MAX = 1.05`

2. **Element reference:** `const el = document.getElementById('cnbs');`
   - Store base font size: `const baseFontSize = parseFloat(getComputedStyle(el).fontSize);`

3. **Helper:** `rand(arr)` returns random element from array. `randRange(min, max)` returns random float in range.

4. **Animation loop:**
   ```
   let lastTick = 0;
   function animate(timestamp) {
     if (timestamp - lastTick >= BURST_INTERVAL) {
       el.style.fontFamily = `"${rand(fonts)}", sans-serif`;
       el.style.color = rand(colors);
       el.style.letterSpacing = randRange(LETTER_SPACING_MIN, LETTER_SPACING_MAX) + 'em';
       el.style.fontSize = clampSize(baseFontSize * randRange(SIZE_SCALE_MIN, SIZE_SCALE_MAX));
       el.style.textShadow = `0 0 20px ${el.style.color}33`;
       lastTick = timestamp;
     }
     requestAnimationFrame(animate);
   }
   requestAnimationFrame(animate);
   ```

   Note: `clampSize` should respect the original `clamp()` bounds — simplest approach is to use the CSS `clamp` and only adjust via a CSS custom property or `transform: scale()` instead of changing `fontSize` directly. Better approach: keep `font-size` as the CSS `clamp()` value and use `transform: scale(randRange(0.95, 1.05))` for the breathing effect. This avoids fighting with clamp.

**Step 2: Verify in browser**

Open `index.html`. The CNBS text should be frantically cycling through fonts, colors, spacing, and size. ~70ms per change. Chaotic and aggressive.

---

### Task 4: Burst + Breathe State Machine

**Files:**
- Modify: `index.html` (JS `<script>` block)

**Step 1: Add state machine to the animation loop**

Replace the simple interval check with a state machine:

1. **State variables:**
   - `let state = 'BURST';`
   - `let stateStart = 0;`
   - `let stateDuration = 0;`
   - `let currentInterval = BURST_INTERVAL;`

2. **`enterBurst()` function:**
   - Set `state = 'BURST'`
   - Set `stateDuration` to random value between 2000-4000ms
   - Set `currentInterval = BURST_INTERVAL`
   - Record `stateStart = timestamp`

3. **`enterBreathe()` function:**
   - Set `state = 'BREATHE'`
   - Set `stateDuration` to random value between 800-1500ms
   - Set `currentInterval = Infinity` (no ticks — hold current style)
   - Add opacity pulse: `el.style.transition = 'opacity 0.4s ease-in-out'; el.style.opacity = '0.85';`
   - After half the duration, set `el.style.opacity = '1';`

4. **State transition logic in `animate()`:**
   - If `timestamp - stateStart >= stateDuration`: transition to opposite state
   - In BURST: apply random styles per tick at `currentInterval`
   - In BREATHE: skip style changes, just check for state expiry

5. **Initialize:** Call `enterBurst()` before first `requestAnimationFrame`.

**Step 2: Verify in browser**

Open `index.html`. Should see: frantic cycling for 2-4 seconds, then a pause where the text holds its last style with a subtle opacity pulse, then erupts into chaos again. The rhythm should feel intentional.

---

### Task 5: Hover Slowdown + Mobile Tap

**Files:**
- Modify: `index.html` (JS `<script>` block)

**Step 1: Add hover interaction**

1. **State variable:** `let isSlowed = false;`
2. **`SLOW_INTERVAL = 400`** constant.

3. **Desktop hover:**
   ```javascript
   el.addEventListener('mouseenter', () => {
     isSlowed = true;
     if (state === 'BURST') currentInterval = SLOW_INTERVAL;
   });
   el.addEventListener('mouseleave', () => {
     isSlowed = false;
     if (state === 'BURST') currentInterval = BURST_INTERVAL;
   });
   ```

4. **Mobile tap:**
   ```javascript
   el.addEventListener('touchstart', (e) => {
     e.preventDefault();
     isSlowed = true;
     if (state === 'BURST') currentInterval = SLOW_INTERVAL;
     setTimeout(() => {
       isSlowed = false;
       if (state === 'BURST') currentInterval = BURST_INTERVAL;
     }, 2000);
   }, { passive: false });
   ```

5. **Integration with state machine:** When `enterBurst()` is called, check `isSlowed` and use `SLOW_INTERVAL` if true.

**Step 2: Add CSS cursor hint**

Add to `.cnbs` styles: `cursor: crosshair;` — a subtle hint that it's interactive without being obvious.

**Step 3: Verify in browser**

Desktop: hover over text — cycling should slow to languid ~400ms pace. Move away — snaps back to frantic. Mobile (use devtools device mode): tap text — slows for 2 seconds then returns.

---

### Task 6: Final Polish + Deploy Readiness

**Files:**
- Modify: `index.html`

**Step 1: Add will-change hint**

Add to `.cnbs` CSS: `will-change: font-family, color, letter-spacing, transform, text-shadow;` to hint GPU compositing.

**Step 2: Add transition smoothing on text-shadow**

Add to `.cnbs` CSS: `transition: text-shadow 0.1s ease;` so the glow doesn't snap too harshly between colors.

**Step 3: Ensure BREATHE state resets opacity transition**

In `enterBurst()`, clear the transition: `el.style.transition = 'text-shadow 0.1s ease'; el.style.opacity = '1';`

**Step 4: Final browser verification**

Full test checklist:
- [ ] Black background, centered text, responsive sizing
- [ ] Frantic font/color/spacing cycling during burst
- [ ] Clean pause during breathe with opacity pulse
- [ ] Hover slows cycling on desktop
- [ ] Tap slows cycling on mobile (devtools)
- [ ] Scanlines visible on close inspection
- [ ] Noise grain barely perceptible
- [ ] Text glow shifts with color
- [ ] No console errors
- [ ] No layout shift or overflow at any viewport size

**Step 5: Verify Cloudflare Pages readiness**

The file should work when served as a static file from any root. Confirm:
- No relative paths that break
- Google Fonts loaded via HTTPS
- No external dependencies besides Google Fonts
