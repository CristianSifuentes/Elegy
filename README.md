# Elegy
### *A gallery for grief that learned to hang on a wall.*

A single-file, dark, melancholic 3D art gallery. Vanilla HTML, CSS, and JavaScript, rendered through [Three.js](https://threejs.org/) — a fractured icosahedron drifting in fog and ash behind a scrollable hall of nine paintings and one triptych. No build step, no framework, no asset folder. Everything — fonts, script, styles — lives inside one `index.html`. Open it and the room is already dark.

![Three.js](https://img.shields.io/badge/Three.js-WebGL-black?style=for-the-badge&logo=three.js&logoColor=white)
![HTML5](https://img.shields.io/badge/HTML5-E34C26?style=for-the-badge&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)

---

## Features

| | Feature | Description |
|---|---|---|
| ✓ | **Three.js gallery core** | A fractured icosahedron (`IcosahedronGeometry(1.65, 1)`) drifts under `FogExp2(0x0a090d, 0.155)`, never fully clearing. A crimson wireframe at 16% opacity traces its broken faces. No bloom pass, no `EffectComposer` — the glow comes from material and light, not post-processing. |
| ✓ | **Three-point cinematic lighting** | Ambient base (`#201c24`, 0.7) holds the room's dark. A crimson point light (`#c24048`, intensity 3.0) orbits the core and pulses via `sin(t·1.7)`. A cold-blue point light (`#3a5a82`) answers from behind, quieter. A near-invisible directional cream fill (0.18) keeps it just barely readable. |
| ✓ | **Drifting ash particles** | A 1,500-point `BufferGeometry` rises at a constant 0.0026px/frame, wrapping from y=−8 back to y=8 — ash that never settles. Rotates independently of the core mesh. |
| ✓ | **Cursor-driven camera parallax** | `pointermove` maps cursor position to a target camera offset, eased at 0.04/frame. The only "camera control" in the gallery — no orbit, no drag, you just look. |
| ✓ | **Trailing cursor glow** | A 620px radial gradient (`rgba(178,59,59,0.13)`), `screen`-blended and blurred 14px, follows the pointer via direct ref transform writes — no re-render per frame. Suggests illumination rather than casting it. |
| ✓ | **Scroll reveal system** | `IntersectionObserver` (threshold 0.12) fades and lifts each tagged element 28px on entry. Backed by a passive-scroll fallback sweep and a hard 1400ms timeout, so content is guaranteed visible even if the observer never fires. |
| ✓ | **Hover-lift artwork frames** | Each of the nine collection tiles rises 8px on hover, border warming to `rgba(178,59,59,0.55)` under a dual shadow (`0 30px 70px -30px` outer, `inset 0 0 80px` glow). A `[ medium ]` watermark holds the center until the title takes the moment. |
| ✓ | **Asymmetric masonry grid** | 12-column CSS grid, `grid-auto-flow: row dense`, each work spanning 3–5 columns at its own aspect ratio — the hall reads as hand-hung, not templated. |
| ✓ | **Film grain overlay** | Animated SVG `feTurbulence` fractal noise at 5% opacity, `overlay` blend mode, looping an 11-stop keyframe with `steps(7)` timing every 5.5s. Keeps the dark from reading as flat digital black. |
| ✓ | **Vignette & depth layering** | Fixed radial + linear gradient layers darken the frame edges and the seams between sections, reinforcing one continuous hall rather than stacked `<div>`s. |
| ✓ | **Difference-blend navigation** | The fixed nav uses `mix-blend-mode: difference`, so its text stays legible whether it's sitting over near-black background or a lit frame. |
| ✓ | **Self-contained bundle** | Three.js, the component runtime, fonts, markup, and styles all live inside one `index.html`. A small loader decodes a base64+gzip asset manifest into blob URLs at load (`DecompressionStream('gzip')`) and swaps in the real document. No server, no build step. |
| ✓ | **Embedded variable webfonts** | Cormorant Garamond (300–600, italic + normal) and Space Mono (400/700) ship as inline `woff2`, split across latin / latin-ext / cyrillic / cyrillic-ext / vietnamese unicode ranges. The gallery never makes a font network request. |
| ✓ | **Fluid responsive type** | Every headline sized with `clamp()` — the hero `h1` alone runs from 78px to 260px depending on viewport. Padding and gaps scale the same way, so the hall doesn't break on mobile. |
| ✓ | **On-page error sink** | A `window.onerror` listener writes failures directly into a fixed panel on the page itself — diagnostic visibility for a single-file project with no console-attached build pipeline. |

---

## The Premise

> *I do not paint to be looked at. I paint the rooms where grief keeps its furniture — the cold side of the bed, the chair that faces the window. Beauty is only the wound, dressed well enough to be approached.*

That's the manifesto printed near the top of the page, in the voice of "an unnamed hand." Below it: nine works between 2021 and 2024, and one centerpiece triptych, hung in a hall that never turns the lights all the way on.

The instruction given to the visitor is simple: *"Move your cursor through the hall. Let the light find what it finds."* There is no WASD, no orbit drag, no pointer-lock walk-through. The only camera you control is attention — where you let the cursor rest.

---

## The Hall

A `<canvas>` sits fixed behind the page (`z-index: 0`), rendering a single Three.js scene that never stops turning, while the HTML content — title, manifesto, the collection grid, the footer — scrolls on top of it.

```js
scene.fog = new THREE.FogExp2(0x0a090d, 0.155);   // the room never fully clears
camera    = new THREE.PerspectiveCamera(54, aspect, 0.1, 100);
camera.position.set(0, 0, 6.2);
```

At the center, a fractured form:

```js
const core = new THREE.IcosahedronGeometry(1.65, 1);
const mesh = new THREE.MeshStandardMaterial({
  color:     0x12101a,   // dark purple-black, almost absent
  roughness: 0.5,
  metalness: 0.45,
});
const wire = new THREE.LineBasicMaterial({ color: 0x7c1d2b, transparent: true, opacity: 0.16 });
```

A faint crimson wireframe traces its broken faces — the wound, dressed well enough to be approached. Around it, **1,500 particles** drift like ash that hasn't decided whether to settle.

**Lighting is a three-point argument between warmth and cold:**

| Light | Color | Intensity | Role |
|---|---|---|---|
| Ambient | `#201c24` | 0.7 | The base dark the room never escapes |
| Point — crimson | `#c24048` | 3.0 | The dominant wound-light, front-right |
| Point — deep blue | `#3a5a82` | 1.5 | A cold answer, back-left, quieter |
| Directional — pale cream | `#e7e2d6` | 0.18 | The barest fill, almost an afterthought |

No bloom pass, no `EffectComposer`. The glow you see is the material and the light doing the work, not a post-processing shader pretending to.

---

## The Palette

| Token | Value | Feeling |
|---|---|---|
| Background | `#08070a` / `#0a090d` | Near-black, the room before your eyes adjust |
| Core mesh | `#12101a` | Dark matter, barely a shape |
| Accent — crimson | `#7c1d2b` | The wound: logo, hover glow, vignette |
| Accent — warm light | `#c24048` | The point light that does the dominant work |
| Accent — cold light | `#3a5a82` | The quiet, distant answer |
| Primary text | `#efe9dd` | Title, the brightest thing in the room |
| Secondary text | `#cfc7ba` / `#ddd5c8` | Body copy, dimmed by design |

The whole scheme resolves around one fact: there is exactly one warm accent, and the entire gallery is built to make you notice it.

---

## The Collection

Nine pieces in a masonry grid, plus a triptych centerpiece called **"The Hours Between."**

| No. | Title | Medium | Year |
|---|---|---|---|
| 01 | The Weight of Absence | Oil & ash on linen | 2024 |
| 02 | Hymn for the Drowned | Charcoal on paper | 2023 |
| 03 | Sleep, and Its Failures | Graphite & wax | 2022 |
| 04 | What the Wound Remembers | Oil on canvas | 2024 |
| 05 | Lullaby in a Minor Key | Ink on cotton | 2023 |
| 06 | Salt | Conceptual | 2021 |
| 07 | A Room That Keeps the Cold | Mixed media | 2023 |
| 08 | Mourning Becomes the Light | Oil on board | 2024 |
| 09 | The Long Goodbye | Oil on board | 2024 |

— *"nine works · 2021–2024"*, as the section summary puts it. The page itself moves through four rooms: **Title → Manifesto → The Collection → Visit**, with a quiet prompt between the first two — *"scroll into the dark ↓."*

---

## Interaction — How It Feels

**The cursor glow.** A 620px radial gradient (`rgba(178,59,59,0.13)`), blurred and blended with `screen`, follows the pointer across the page. It doesn't illuminate — it suggests illumination, the way a candle held in the next room throws light you can't quite trace to its source.

**Hovering a piece.** Each frame in the grid lifts 8px on hover, its border warming to `rgba(178,59,59,0.55)`, and a soft double shadow blooms around it:

```css
box-shadow:
  0 30px 70px -30px rgba(124,29,43,0.7),
  inset 0 0 80px rgba(124,29,43,0.12);
```

The piece doesn't announce itself. It rises slightly, the way something does when you finally look directly at it.

**Scroll reveal.** Fifteen elements across the page carry a reveal state — opacity and a small vertical drift — that resolves as each crosses into view. The hall assembles itself as you walk through it, not all at once.

**Film grain.** A faint SVG fractal-noise overlay, 5% opacity, looping on a 5.5s cycle in `overlay` blend mode. It's the thing that keeps the darkness from feeling digital — a texture borrowed from film stock, not screens.

---

## Typography

Two typefaces carry the entire voice of the gallery:

- **Cormorant Garamond** — serif, several weights and italics — for the title, the manifesto, the piece names. Literary, unhurried.
- **Space Mono** — monospace, regular and bold — for catalogue numbers, captions, and the small technical line under the title: *"WebGL · oil & ash · est. MMXXIV."*

Both are bundled as embedded `woff2` data, with extended Latin/Cyrillic/Vietnamese coverage baked in — the gallery never makes a network request for a letterform.

---

## Quick Start

```bash
# No install. No build. No bundler to run — it already ran.
start index.html       # Windows
open index.html        # macOS
xdg-open index.html    # Linux
```

`index.html` is a self-contained build artifact: fonts, script, and styles are inlined directly into the document. There is nothing else to fetch and nothing else to install.

---

## Project Structure

```
elegy/
├── index.html     everything — markup, styles, Three.js scene, fonts, copy
├── LICENSE         Apache 2.0
└── README.md       you are here
```

There is intentionally no `src/`, `assets/`, or build config in this repository — what's committed is the finished, bundled page itself. To make changes, edit the inline `<style>` and `<script>` blocks in `index.html` directly.

---

## Customisation

**Change the accent.** The crimson that anchors the whole palette is `#7c1d2b` (and its warm sibling `#c24048` in the point light). Search-and-replace both inside `index.html` to shift the entire emotional register of the room — cooler blues read as winter rather than wound; ochres read as candlelight rather than fire.

**Add or edit a piece.** Each grid entry is a small block of markup (number, title, medium, year) inside the collection section. Copy an existing entry, change the four fields, and the masonry grid absorbs it.

**Adjust the fog.** `FogExp2(0x0a090d, 0.155)` — raise the second value to thicken the dark closer to the camera; lower it to let more of the particle field breathe before it's swallowed.

---

## Browser Support

Requires WebGL and modern CSS (custom properties, `mix-blend-mode`, CSS filters).

| Browser | Support |
|---|---|
| Chrome 90+ | ✅ Full |
| Firefox 90+ | ✅ Full |
| Safari 15+ | ✅ Full |
| Edge 90+ | ✅ Full |
| IE 11 | ✗ Not supported (no WebGL/ES module support) |

---

## Deployment

```bash
# GitHub Pages
git init && git add . && git commit -m "init"
git branch -M main
git remote add origin https://github.com/YOUR/repo.git
git push -u origin main
# then enable Pages in repository Settings → Pages → main branch
```

Netlify and Vercel work identically — connect the repository and deploy. There is no build command to configure; `index.html` is the output.

---

## Design Philosophy

The gallery is built around a single tension: a fractured, almost-absent form, held in fog, lit by one warm wound of color and one cold, distant answer. Nothing moves quickly. Nothing brightens fully. The grain, the drift, the slow reveal — every effect is in service of the same idea the manifesto states outright: beauty here is the wound, dressed well enough to be approached, never resolved.

---

## License

[Apache License 2.0](LICENSE)

---

**Made with intention by [Cristian Sifuentes](https://github.com/CristianSifuentes)**
