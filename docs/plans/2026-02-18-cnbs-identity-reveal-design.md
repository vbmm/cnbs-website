# CNBS Identity Reveal Page — Design

## Summary

Single-page website: animated "CNBS" text on pure black background. Glitchy, high-end identity reveal aesthetic. Luxury meets raw energy.

## Deliverable

Single `index.html` file, no build step, deployable to Cloudflare Pages.

## Layout

- Pure black `#000` background, full viewport, no scroll
- "CNBS" centered via flexbox, sized with `clamp(4rem, 20vw, 25rem)`
- Full-screen scanline overlay via `body::after` (CSS-only)
- SVG noise grain overlay via `body::before` at very low opacity

## Fonts (Google Fonts CDN)

~18 fonts across categories:

- **Serif:** Playfair Display, Cormorant Garamond, Bodoni Moda
- **Sans-serif:** Inter, Bebas Neue, Oswald
- **Display/wild:** Unlock, Righteous, Monoton, Bungee Shade, Rubik Glitch, Rubik Wet Paint, Rubik Vinyl, Silkscreen, Honk, Nabla
- **Condensed:** Barlow Condensed
- **Monospace:** Space Mono, JetBrains Mono

All loaded via single `<link>` tag, `display=swap`, bold weights only (700/900).

## Animation System

**Engine:** JavaScript `requestAnimationFrame` loop with timestamp throttle.

**Per tick (~70ms during burst):** Randomly select:
- Font from array
- Color from palette
- Letter-spacing: `-0.05em` to `0.15em`
- Font-size multiplier: `0.95x` to `1.05x` (breathing)

Applied via direct `element.style` writes.

**Color palette:**
- Deep golds: `#C9A84C`, `#FFD700`, `#B8860B`
- Electric blues: `#0057FF`, `#00D4FF`, `#4169E1`
- Cold whites: `#F0F0F0`, `#E8E8E8`, `#FFFFFF`
- Blood reds: `#8B0000`, `#DC143C`, `#FF1744`
- Neon accents: `#39FF14`, `#FF00FF`, `#FF6600`

**Burst + breathe state machine:**
- `BURST`: Frantic cycling at ~70ms for 2-4s (randomized)
- `BREATHE`: Hold current style for 0.8-1.5s, subtle opacity pulse to ~0.85
- Loops indefinitely

**Hover interaction:**
- Desktop: Hover over text slows tick to ~400ms (languid cycling)
- Mobile: Tap triggers 2s slow period, then returns to frantic
- Mouse leave snaps back to frantic speed

## Overlays

**Scanlines:** `body::after`, `pointer-events: none`, repeating-linear-gradient `rgba(255,255,255,0.03)` 1px / transparent 1px at 2px intervals.

**Noise grain:** `body::before` with inline SVG `feTurbulence` filter, opacity ~0.02-0.04.

**Text glow:** `text-shadow: 0 0 20px currentColor` at ~0.3 opacity, shifts with color cycling.

## Technical

- HTML5 boilerplate with charset, viewport meta, black theme-color
- Zero dependencies beyond Google Fonts CDN
- Single file deployment to Cloudflare Pages
