# TRON 3D — Light Cycles with Decaying Trails

A single-file 3D Tron Light Cycles game built with vanilla JavaScript and
Three.js, with one twist that makes the arena constantly rewrite itself:
**trails fade and disappear after ~7 seconds.**

> Just open `index.html` in a modern browser — no build, no server, no dependencies.

---

## How to Run

The game is a single self-contained HTML file with everything inlined
(geometry, shaders, audio, logic) and Three.js loaded via the
[esm.sh](https://esm.sh) ESM CDN.

### Option 1 — Double click
Open `index.html` in any modern Chromium/Firefox/Safari browser.
That's it.

### Option 2 — GitHub Pages
This repo is intended to be served as a GitHub Page. Push `main` and
enable Pages → branch `main` / root. The page will work identically
because there is no build step.

> Note: ESM module imports require the page to be served over `http(s)://`
> in some browsers. GitHub Pages serves over HTTPS so you're fine. If you
> open the file via `file://` and Chrome blocks ESM modules, run a tiny
> local server: `python3 -m http.server` from this directory, then visit
> `http://localhost:8000/`.

---

## Controls

| Key                          | Action                |
|------------------------------|-----------------------|
| **A** / **←**                | Turn left             |
| **D** / **→**                | Turn right            |
| **W** / **↑**                | Boost (faster)        |
| **S** / **↓**                | Brake (slower)        |
| **Space**                    | Activate shield       |
| **P**                        | Pause / resume        |
| **R** (on game over)         | Restart               |
| **Click "ENGAGE" / "RE-ENGAGE"** | Start / restart   |

Hold **A** or **D** for continuous turning, or tap them for a 90° snap turn.

---

## The Twist

In classic Tron, trails are permanent obstacles — once dropped, they're
walls forever, and matches are decided quickly by who can box whom in.

Here, every trail segment carries a **birth time**. Each frame the
trail's overall alpha is recomputed as `1 - age / 7s`. After ~7 seconds
the segment is pruned and the geometry buffer shrinks. Visually the
arena is a palimpsest — your own trail is fading behind you while the
AI's bright walls are dissolving in front of them.

Strategic implications:
- **Lure** opponents into their own fresh trail before it fades.
- **Skirt** your own decaying wake — older sections are still solid
  for the first few seconds but become passable again once alpha hits
  zero.
- **Shield** (Space) lets you phase through your own trail for 4
  seconds. Useful to escape a trap. It does NOT make you invulnerable
  to AI trails — those still kill you.

---

## What's In The Box

- Player cycle (cyan) vs. 3 AI opponents (magenta / orange / red).
- AI uses a simple hazard-avoidance controller: cast a ray forward,
  if a trail or wall is too close, queue a 90° turn toward the side
  with more room. Adds a dash of randomness so they don't synchronise.
- Glowing trails rendered as `LineSegments` with a parallel offset
  "glow" line for thickness under bloom.
- Procedural grid floor with shader-based major/minor grid lines and
  a radial vignette.
- Bounding arena walls that glow on the top edge.
- Per-cycle PointLight for local glow + UnrealBloomPass for the neon
  halo look.
- Particle burst on death (instanced tetrahedrons, 60 particles per
  crash, with gravity + drag).
- Continuous engine hum that pitch-shifts with speed (Web Audio).
- One-shot synth sounds for: whoosh on turn, crash (filtered noise),
  power-up (arpeggio), score chime.
- HUD: live score, high score (localStorage), time survived, speed
  multiplier, AI liveness roster, shield charge bar.
- Game over screen with stats and a NEW HIGH SCORE callout when
  applicable.
- Press **P** to pause.

---

## File Map

```
index.html   ← entire game (HTML + CSS + JS + shaders + audio)
README.md    ← this file
LICENSE      ← MIT
```

---

## Tech Notes

- **Three.js 0.160.0** via [esm.sh](https://esm.sh/three@0.160.0).
- **Postprocessing**: `EffectComposer` + `RenderPass` +
  `UnrealBloomPass` + `OutputPass`, all from the official `examples/jsm`
  path through esm.sh.
- **ESM import map** maps `three` and `three/addons/` to esm.sh URLs —
  no bundler, no npm, no node_modules.
- **Audio**: Web Audio API, entirely procedural (no audio files).
  An AudioContext is created on the first user gesture (the ENGAGE
  click) to comply with browser autoplay policies.
- **Persistence**: `localStorage` key `tron3d.high` for high score.

---

## Performance Notes

- Trails prune oldest segments every 250ms rather than every frame.
- Geometry rebuilds use a single `Float32Array` per trail and reuse
  `BufferAttribute`. Worst case is ~7s × 22 u/s × 4 cycles ≈ 600
  segments × 2 vertices × 3 floats — well under any GPU budget.
- Particles use a single 900-instance `InstancedMesh`.
- Pixel ratio is capped at 2 to avoid melting low-end laptops.

---

## Credits

- Game design & code: Jorge Quijano.
- Three.js: [mrdoob and contributors](https://threejs.org/), MIT.
- Inspired by the light cycle sequences from the Tron franchise
  (1982 / 2010).
- Built with assistance from Hermes Agent.

License: MIT.
