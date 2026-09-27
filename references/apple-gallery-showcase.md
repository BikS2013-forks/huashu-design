# Apple Gallery Showcase · Gallery Wall Animation Style

> Inspiration: Claude Design website hero video + Apple product page "gallery wall" arrangement
> Real-world origin: huashu-design release hero v5
> Use cases: **Product launch hero animations, skill capability demos, portfolio showcases** — any scenario where you need to display "many high-quality outputs" simultaneously while guiding the viewer's attention

---

## Trigger Decision: When to Use This Style

**Suitable for**:
- 10+ real outputs to display on the same screen (PPTs, apps, web pages, infographics)
- Audience is professional (developers, designers, product managers), sensitive to "texture"
- Desired tone: "restrained, exhibition-style, premium, with a sense of space"
- Need both focus and global view simultaneously (see details without losing the whole)

**Not suitable for**:
- Single product focus (use frontend-design's product hero template)
- Emotional / story-heavy animations (use the timeline narrative template)
- Small screens / portrait orientation (tilted perspective gets blurry on small canvases)

---

## Core Visual Tokens

```css
:root {
  /* Light gallery palette */
  --bg:         #F5F5F7;   /* Main canvas background — Apple website gray */
  --bg-warm:    #FAF9F5;   /* Warm off-white variant */
  --ink:        #1D1D1F;   /* Primary text color */
  --ink-80:     #3A3A3D;
  --ink-60:     #545458;
  --muted:      #86868B;   /* Secondary text */
  --dim:        #C7C7CC;
  --hairline:   #E5E5EA;   /* 1px card border */
  --accent:     #D97757;   /* Terracotta orange — Claude brand */
  --accent-deep:#B85D3D;

  --serif-cn: "Noto Serif SC", "Songti SC", Georgia, serif;
  --serif-en: "Source Serif 4", "Tiempos Headline", Georgia, serif;
  --sans:     "Inter", -apple-system, "PingFang SC", system-ui;
  --mono:     "JetBrains Mono", "SF Mono", ui-monospace;
}
```

**Key principles**:
1. **Never use pure black background**. Black backgrounds make work look like a movie, not "results worth adopting"
2. **Terracotta orange is the only accent hue**, everything else is grayscale + white
3. **Three-font stack** (serif EN + serif CN + sans + mono) creates a "publication" rather than "internet product" feel

---

## Core Layout Patterns

### 1. Floating Card (the basic unit of the entire style)

```css
.gallery-card {
  background: #FFFFFF;
  border-radius: 14px;
  padding: 6px;                          /* Inner padding is the "matting paper" */
  border: 1px solid var(--hairline);
  box-shadow:
    0 20px 60px -20px rgba(29, 29, 31, 0.12),   /* Main shadow, soft and long */
    0 6px 18px -6px rgba(29, 29, 31, 0.06);     /* Second-layer near light, creates float effect */
  aspect-ratio: 16 / 9;                  /* Unified slide ratio */
  overflow: hidden;
}
.gallery-card img {
  width: 100%; height: 100%;
  object-fit: cover;
  border-radius: 9px;                    /* Slightly smaller than card border-radius, for visual nesting */
}
```

**Anti-pattern**: Don't use edge-to-edge tiles (no padding, no border, no shadow) — that's infographic density expression, not an exhibition.

### 2. 3D Tilted Gallery Wall

```css
.gallery-viewport {
  position: absolute; inset: 0;
  overflow: hidden;
  perspective: 2400px;                   /* Deeper perspective, tilt isn't exaggerated */
  perspective-origin: 50% 45%;
}
.gallery-canvas {
  width: 4320px;                         /* Canvas = 2.25× viewport */
  height: 2520px;                        /* Leave pan space */
  transform-origin: center center;
  transform: perspective(2400px)
             rotateX(14deg)              /* Tilt back */
             rotateY(-10deg)             /* Turn left */
             rotateZ(-2deg);             /* Slight tilt, removes too-tidy feel */
  display: grid;
  grid-template-columns: repeat(8, 1fr);
  gap: 40px;
  padding: 60px;
}
```

**Parameter sweet spots**:
- rotateX: 10-15deg (more becomes a wine reception VIP backdrop)
- rotateY: ±8-12deg (left-right symmetry)
- rotateZ: ±2-3deg (the "this isn't placed by a machine" human touch)
- perspective: 2000-2800px (under 2000 fish-eyes, over 3000 approaches orthographic projection)

### 3. 2×2 Four-Corner Convergence (selection scene)

```css
.grid22 {
  display: grid;
  grid-template-columns: repeat(2, 800px);
  gap: 56px 64px;
  align-items: start;
}
```

Each card slides in from its corresponding corner (tl/tr/bl/br) toward the center + fade in. Corresponding `cornerEntry` vectors:

```js
const cornerEntry = {
  tl: { dx: -700, dy: -500 },
  tr: { dx:  700, dy: -500 },
  bl: { dx: -700, dy:  500 },
  br: { dx:  700, dy:  500 },
};
```

---

## Five Core Animation Patterns

### Pattern A · Four-Corner Convergence (0.8-1.2s)

4 elements slide in from the viewport's four corners, simultaneously scaling 0.85→1.0, with ease-out. Suitable as an opening that "displays multi-directional choices".

```js
const inP = easeOut(clampLerp(t, start, end));
card.style.transform = `translate3d(${(1-inP)*ce.dx}px, ${(1-inP)*ce.dy}px, 0) scale(${0.85 + 0.15*inP})`;
card.style.opacity = inP;
```

### Pattern B · Selected Zoom + Others Slide Out (0.8s)

The selected card scales 1.0→1.28, other cards fade out + blur + drift back to corners:

```js
// Selected
card.style.transform = `translate3d(${cellDx*outP}px, ${cellDy*outP}px, 0) scale(${1 + 0.28*easeOut(zoomP)})`;
// Not selected
card.style.opacity = 1 - outP;
card.style.filter = `blur(${outP * 1.5}px)`;
```

**Key**: Unselected cards must blur, not just plain fade. Blur simulates depth of field, visually "pushing out" the selected one.

### Pattern C · Ripple Expansion (1.7s)

From center outward, delay by distance, each card fades in + scales from 1.25x to 0.94x ("camera pulls back"):

```js
const col = i % COLS, row = Math.floor(i / COLS);
const dc = col - (COLS-1)/2, dr = row - (ROWS-1)/2;
const dist = Math.sqrt(dc*dc + dr*dr);
const delay = (dist / maxDist) * 0.8;
const localT = Math.max(0, (t - rippleStart - delay) / 0.7);
card.style.opacity = easeOut(Math.min(1, localT));

// Meanwhile overall scale 1.25→0.94
const galleryScale = 1.25 - 0.31 * easeOut(rippleProgress);
```

### Pattern D · Sinusoidal Pan (continuous drift)

Use sine wave + linear drift combination, avoiding the marquee-style "has start, has end" loop feel:

```js
const panX = Math.sin(panT * 0.12) * 220 - panT * 8;    // Horizontal left drift
const panY = Math.cos(panT * 0.09) * 120 - panT * 5;    // Vertical up drift
const clampedX = Math.max(-900, Math.min(900, panX));   // Prevent edge exposure
```

**Parameters**:
- Sine period `0.09-0.15 rad/s` (slow, about 30-50 seconds per oscillation)
- Linear drift `5-8 px/s` (slower than viewer's blink)
- Amplitude `120-220 px` (large enough to feel, small enough not to be dizzying)

### Pattern E · Focus Overlay (focus switching)

**Key design**: focus overlay is a **flat element** (not tilted), floating above the tilted canvas. The selected slide scales from tile position (about 400×225) to screen center (960×540), background canvas doesn't change tilt but **dims to 45%**:

```js
// Focus overlay (flat, centered)
focusOverlay.style.width = (startW + (endW - startW) * focusIntensity) + 'px';
focusOverlay.style.height = (startH + (endH - startH) * focusIntensity) + 'px';
focusOverlay.style.opacity = focusIntensity;

// Background cards dim, but still visible (key! Don't 100% mask)
card.style.opacity = entryOp * (1 - 0.55 * focusIntensity);   // 1 → 0.45
card.style.filter = `brightness(${1 - 0.3 * focusIntensity})`;
```

**Sharpness iron rules**:
- Focus overlay's `<img>` must have `src` directly linked to original image, **don't reuse compressed thumbnails from the gallery**
- Preload all original images into a `new Image()[]` array in advance
- Overlay's own `width/height` calculated per frame, browser resamples original image each frame

---

## Timeline Architecture (reusable skeleton)

```js
const T = {
  DURATION: 25.0,
  s1_in: [0.0, 0.8],    s1_type: [1.0, 3.2],  s1_out: [3.5, 4.0],
  s2_in: [3.9, 5.1],    s2_hold: [5.1, 7.0],  s2_out: [7.0, 7.8],
  s3_hold: [7.8, 8.3],  s3_ripple: [8.3, 10.0],
  panStart: 8.6,
  focuses: [
    { start: 11.0, end: 12.7, idx: 2  },
    { start: 13.3, end: 15.0, idx: 3  },
    { start: 15.6, end: 17.3, idx: 10 },
    { start: 17.9, end: 19.6, idx: 16 },
  ],
  s4_walloff: [21.1, 21.8], s4_in: [21.8, 22.7], s4_hold: [23.7, 25.0],
};

// Core easing
const easeOut = t => 1 - Math.pow(1 - t, 3);
const easeInOut = t => t < 0.5 ? 4*t*t*t : 1 - Math.pow(-2*t+2, 3)/2;
function lerp(time, start, end, fromV, toV, easing) {
  if (time <= start) return fromV;
  if (time >= end) return toV;
  let p = (time - start) / (end - start);
  if (easing) p = easing(p);
  return fromV + (toV - fromV) * p;
}

// Single render(t) function reads timestamp, writes all elements
function render(t) { /* ... */ }
requestAnimationFrame(function tick(now) {
  const t = ((now - startMs) / 1000) % T.DURATION;
  render(t);
  requestAnimationFrame(tick);
});
```

**Architecture essence**: **All state derived from timestamp t**, no state machines, no setTimeout. This way:
- Playing to any moment `window.__setTime(12.3)` jumps instantly (convenient for playwright frame-by-frame capture)
- Loop is naturally seamless (t mod DURATION)
- Can freeze any frame when debugging

---

## Texture Details (easy to overlook but fatal)

### 1. SVG noise texture

Light backgrounds fear "too flat" most. Overlay an extremely faint fractalNoise:

```html
<style>
.stage::before {
  content: '';
  position: absolute; inset: 0;
  background-image: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='200' height='200'><filter id='n'><feTurbulence type='fractalNoise' baseFrequency='0.85' numOctaves='2' stitchTiles='stitch'/><feColorMatrix values='0 0 0 0 0.078  0 0 0 0 0.078  0 0 0 0 0.074  0 0 0 0.035 0'/></filter><rect width='100%' height='100%' filter='url(%23n)'/></svg>");
  opacity: 0.5;
  pointer-events: none;
  z-index: 30;
}
</style>
```

Looks no different, but you'll know once you remove it.

### 2. Corner brand mark

```html
<div class="corner-brand">
  <div class="mark"></div>
  <div>HUASHU · DESIGN</div>
</div>
```

```css
.corner-brand {
  position: absolute; top: 48px; left: 72px;
  font-family: var(--mono);
  font-size: 12px;
  letter-spacing: 0.22em;
  text-transform: uppercase;
  color: var(--muted);
}
```

Only shown in the gallery wall scene, fade in/out. Like a museum exhibit label.

### 3. Brand closing wordmark

```css
.brand-wordmark {
  font-family: var(--sans);
  font-size: 148px;
  font-weight: 700;
  letter-spacing: -0.045em;   /* Negative letter-spacing is key, tightens letters into a logo */
}
.brand-wordmark .accent {
  color: var(--accent);
  font-weight: 500;           /* Accent character is thinner, visual contrast */
}
```

`letter-spacing: -0.045em` is the standard Apple product page large-text approach.

---

## Common Failure Modes

| Symptom | Cause | Solution |
|---|---|---|
| Looks like a PPT template | Cards have no shadow / hairline | Add two-layer box-shadow + 1px border |
| Tilt feels cheap | Only used rotateY without rotateZ | Add ±2-3deg rotateZ to break the tidy feel |
| Pan feels "stuttering" | Used setTimeout or CSS keyframes loop | Use rAF + sin/cos continuous functions |
| Text unclear when focused | Reused gallery tile's low-res image | Independent overlay + original image src direct link |
| Background too empty | Pure color `#F5F5F7` | Overlay SVG fractalNoise at 0.5 opacity |
| Fonts feel too "internet" | Only Inter | Add Serif (CN+EN) + mono three-stack |

---

## References

- Complete implementation sample: `hero-animation-v5.html` (in the original author's local workspace; not included in this repository)
- Original inspiration: claude.ai/design hero video
- Reference aesthetics: Apple product pages, Dribbble shot collection pages

When facing animation needs to "display many high-quality outputs", directly copy the skeleton from this file, swap content + adjust timing.
