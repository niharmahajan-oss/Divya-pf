# Divya Roy — Portfolio

A dreamy, slightly futuristic 3D marketing portfolio — one continuous,
buttery-smooth scrolling page. **Home scene:** the flower garden — move the
cursor (or drag a finger) and flowers bloom along the path, with pollen
sparkles, a lagging cursor halo and a soft bloom pass. **Content scene:** the
mascot stage — four glassy section cards arranged symmetrically around her,
reached by *scrolling*, not clicking. Built with **React + Vite + React Three
Fiber + GSAP ScrollTrigger + Lenis**.

## Run it

```bash
npm install
npm run dev      # http://localhost:5173
npm run build    # production build → dist/
```

## How the site is structured

One page, two fixed scenes, one scroll story:

- Everything visible is `position: fixed` (WebGL canvas → scene layers →
  grain). `#scroll-space` is a transparent ~370vh spacer — the only in-flow
  element — so Lenis has something to scroll.
- **ScrollDirector** runs Lenis (eased inertia) plus a single GSAP
  ScrollTrigger with `scrub: 1.1` that maps scroll progress (0 → 1) to the
  whole story. Scroll is the single source of truth: scrolling back replays
  the transition in reverse automatically, and nothing ever jumps.
- Scenes are stacked as fixed layers, so **adding more sections later**
  (About, Skills, Work, Contact) means adding another fixed layer plus a
  chapter on the master timeline — no routing involved.

## The scroll story (scrubbed, soft)

| progress | what happens |
| -------- | ------------ |
| 0.00–0.30 | hero rises −40px, fades, blurs to 6px; flowers drift up and fade gently (no bursts) |
| 0.15–0.65 | background gradient interpolates continuously home (`#485247`→black) → content (amethyst `#9B71B2` heart→black edges) inside the WebGL scene |
| 0.38–0.66 | mascot floats up 40px into place; the four cards fade in one by one with a small upward drift (`stagger: 0.06`) |
| 0.66+ | scroll-up cue fades in; everything settles |

A softly pulsing *"scroll"* cue (Pinyon Script `#FFEE08`) sits at the bottom of
the home scene and fades out the moment scrolling starts; *"▴ back to the
garden"* mirrors it at the top of the content scene.

## The content scene

- **Mascot:** dead-center in the viewport (≈62vh tall, ≥3vh below her boots).
  Her head/body turn subtly toward the cursor (spring-eased rotateY/rotateX,
  ±8°/±5°) and toward whichever section card is hovered. Behind her: a soft
  circular amethyst radial glow (no hard edges) plus petals orbiting her
  center point, fading in with scroll progress.
- **Cards:** CSS grid — mascot in the center column, one card per side cell.
  All four are identical (`clamp(240px, 22vw, 340px)` × 150px, labels wrap so
  "Skills & Certification" never stretches a card), top pair shares one top
  offset, bottom pair mirrors it, and both sides sit equidistant from the
  center axis. Verified at 1440×900, 1920×1080, 1280×720 and 390×844 (2×2
  below/around her on mobile).
- **Hover:** a glowing petal-trail connector draws from her heart to the card.
  Card clicks log `[section] <id> clicked` — the inner section pages are not
  built yet.

## Structure

```
public/fonts/                 ← drop sergio-trendy.woff2 here
public/mascot/                ← mascot_glow.png
src/
  App.jsx                     ← fixed layer stack + #scroll-space spacer
  lib/
    ScrollDirector.js         ← Lenis + master ScrollTrigger (the whole story)
    pageStore.js              ← scroll progress/section store + scrollProxy
    palette.js                ← brand palette
    pointerStore.js           ← mutable pointer state (no re-renders)
    easing.js, fontProbe.js   ← helpers, conditional Sergio Trendy @font-face
  hooks/
    usePointerTracking.js     ← one global listener (+ hides the halo when
                                the cursor leaves the window)
    useReducedMotion.js
  pages/
    Home/
      HomePage.jsx            ← fixed home scene: hero + scroll cue
      Hero.jsx                ← "Portfolio" + "Divya Roy"
    Content/
      ContentPage.jsx         ← fixed content scene: grid stage + cues
      Mascot.jsx              ← head/body tracking, halo, pendant, clip
      SectionCard.jsx         ← glassy bob/tilt card, fires card:hover
      ConnectorLayer.jsx      ← 2D canvas petal-trail to the hovered card
  components/three/
    Scene.jsx                 ← Canvas + SceneBackground (scroll-interpolated
                                gradient) + mascot-anchored petal orbit
    CameraDirector.jsx        ← cursor parallax + scroll drift
    FlowerGarden.jsx          ← pooled instanced flowers (setGlobalFade drift)
    CursorTrail.jsx           ← throttled spawner; density eases down with scroll
    Pollen.jsx, CursorHalo.jsx, AmbientParticles.jsx,
    BackgroundRings.jsx, OrbitingPetals.jsx, flowerGeometry.js
  styles/global.css           ← scenes, grid, mascot accents, cues, mobile
```

## Drop in the Sergio Trendy font

Sergio Trendy isn't on Google Fonts, so it is self-hosted. Save your file as
**`public/fonts/sergio-trendy.woff2`** (`.woff`, `.otf` and `.ttf` also work)
and it activates automatically — no code changes. Until then headings fall
back to Fraunces Black. The runtime probe (`src/lib/fontProbe.js`) only
injects the `@font-face` when the file actually exists.

## Performance notes

- Capped DPR (1–1.75), pooled instancing (~4 draw calls for the whole garden),
  one bloom pass, GPU-only transforms (`will-change` on the scrubbed layers).
- Scroll runs through Lenis; ScrollTrigger is kept in step via
  `lenis.on('scroll', ScrollTrigger.update)`.
- The background gradient interpolates in a tiny shader — no DOM repaints.
- `prefers-reduced-motion`: no scrubbed transforms (opacity-only crossfades
  with an instant scrub), muted mascot/rings/trail, no halo/animation loops.
