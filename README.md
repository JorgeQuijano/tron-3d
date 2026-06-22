# TRON 3D — Light Cycles with Decaying Trails

A single-file 3D Tron Light Cycles game built with vanilla JavaScript and
Three.js. One twist makes the arena constantly rewrite itself:
**trails fade and disappear after ~7 seconds** — so your own wake
becomes passable again, and baiting opponents into their own freshly
laid walls becomes a real tactic.

Player vs 3 AI opponents, neon glow, bloom, procedural audio, particle
deaths, mobile touch controls, mini-map, and smooth analog turning —
all from one self-contained `index.html`.

> Just open `index.html` in any modern browser — no build, no server,
> no dependencies.

---

## Live Demo

**Play it now:** [https://jorgequijano.github.io/tron-3d/](https://jorgequijano.github.io/tron-3d/)

---

## How to Run

The game is one self-contained HTML file with everything inlined
(geometry, shaders, audio, logic) and Three.js loaded via the
[esm.sh](https://esm.sh) ESM CDN.

### Option 1 — Double click
Open `index.html` in any modern Chromium / Firefox / Safari browser.
That's it.

### Option 2 — GitHub Pages
This repo is configured to be served as a GitHub Page. Push to `main`
and enable Pages → branch `main` / root. The page will work identically
because there is no build step.

### Option 3 — Local server
ESM module imports require the page to be served over `http(s)://` in
some browsers (Chrome blocks ESM over `file://`).

```bash
# from this directory
python3 -m http.server 8000
# then visit http://localhost:8000/
```

Or with Node:
```bash
npx serve .
```

---

## Controls

### Desktop (keyboard)

| Key                          | Action                |
|------------------------------|-----------------------|
| **A** / **←**                | Turn left             |
| **D** / **→**                | Turn right            |
| **W** / **↑**                | Boost (faster)        |
| **S** / **↓**                | Brake (slower)        |
| **Space**                    | Activate shield       |
| **P**                        | Pause / resume        |
| **R** (on game over)         | Restart               |
| **Enter** (on start screen)  | Start                 |
| **Click "ENGAGE"**           | Start / restart       |

Tapping or holding **A** / **D** both work — input is smoothed so
neither feels jarring.

### Mobile / tablet (touch)

An on-screen control overlay appears automatically on touch devices
(detected via `(pointer: coarse)` or a viewport ≤ 900 px wide).

| Button        | Action     |
|---------------|------------|
| ◀ ▶           | Turn       |
| ▲             | Boost      |
| **BRK**       | Brake      |
| **SHIELD**    | Activate shield (one-shot) |
| Tap empty area | Pause / resume |

Pinch-zoom and pull-to-refresh are blocked on the game canvas so
swipes never accidentally scroll or zoom the page.

---

## The Twist: Decaying Trails

In classic Tron, trails are permanent obstacles — once dropped they're
walls forever, and matches are decided quickly by who can box whom in.

Here every trail segment carries a **birth time** in game-time seconds.
Each frame:

- **Alpha** is recomputed as `1 - age / 7s`. After 7 s the segment is
  pruned and the geometry buffer shrinks.
- **Color** is lerped from the cycle's saturated color toward a dim
  cool gray (`#4D4D57`) so older sections visually wash out.
- **Collisions** are still checked against live segments, so a fading
  trail is still a wall until it actually disappears.

Strategic implications:
- **Lure** opponents into their own fresh trail before it fades.
- **Skirt** your own decaying wake — older sections become passable
  again once they're pruned, so you can sometimes run circles around
  an opponent and then slip back through your own old wall.
- **Shield** (Space) lets you phase through your own trail for 4 s.
  Useful to escape a trap. It does NOT make you invulnerable to AI
  trails — those still kill you.

---

## Features

### Core gameplay
- Player cycle (cyan) vs. 3 AI opponents (magenta / orange / red).
- AI uses a forward-ray hazard-avoidance controller: cast a ray, if a
  trail or wall is too close, queue a 90° turn toward the side with
  more room. Adds a dash of randomness so AIs don't synchronise.
- Continuous engine hum that pitch-shifts with speed (Web Audio API).
- One-shot synth sounds for: whoosh on turn, crash (filtered noise),
  power-up (arpeggio), score chime.
- Particle burst on death (instanced tetrahedrons, 60 particles per
  crash, with gravity + drag).
- Game over screen with stats and a **NEW HIGH SCORE** callout.
- High score persisted to `localStorage`.

### Level system

The game now has a 5-level campaign plus an unlockable **Endless Mode**.

| Level | Subtitle | AIs | AI speed | Arena |
|------:|----------|----:|---------:|-------|
| I     | OPEN GRID     | 1 | 0.85× | Empty square — tutorial |
| II    | CENTER PILLAR | 2 | 0.95× | One large pillar in the middle |
| III   | CORNER PYLONS | 2 | 1.05× | 4 pylons in each corner |
| IV    | INNER WALLS   | 3 | 1.10× | Cross-shaped inner walls |
| V     | GRID FRAGMENT | 3 | 1.20× | 5 fragmented blocks |

Each level has a large intro banner with the level number + subtitle, and
kill all AIs to advance. After clearing Level V, **Endless Mode unlocks**
and a magenta ENDLESS button appears on the start screen.

**Endless Mode**: random layout (re-rolled from the campaign levels)
each wave, with AI count and speed scaling up. Waves start at 1 and
go up. A wave counter replaces the LEVEL display in the HUD (`W1`, `W2`,
…). High score from the last endless run is saved separately.

Progress (campaign completion + endless high score) is persisted to
`localStorage` so unlocking survives page reloads.

### Visual polish
- Glowing trails rendered as `LineSegments` with a parallel offset
  "glow" line for thickness under bloom.
- **Trail color aging**: segments fade from saturated color to dim
  gray as they age (visible in the 3D scene and on the mini-map).
- **Boost FX**: player cycle's emissive intensity pulses while
  throttle is held; trailing particles stream behind the cycle;
  HUD speed indicator turns white-yellow.
- **Shield-break shockwave**: cyan ring expands outward at the
  player's position when the shield expires (~400 ms).
- Procedural grid floor with shader-based major/minor grid lines and
  a radial vignette.
- Bounding arena walls that glow on the top edge.
- Per-cycle PointLight for local glow + UnrealBloomPass for the neon
  halo look.
- ACES tonemapping for cinematic color.
- Fog blends distant arena edges into the background.

### UI
- HUD: live score, high score, time survived, speed multiplier.
- AI liveness roster with per-cycle status indicators.
- Shield power-up bar with active / charging / ready states.
- Pause overlay, game-over overlay with full stats.
- **Mini-map (radar)** in the top-right corner — shows the arena
  outline, all cycles as colored dots (with heading direction),
  and their trails as colored polylines that age to gray. Hidden on
  mobile to save space.
- Touch control overlay that auto-hides on desktop.

### Camera
- Third-person chase camera, 32 units back / 22 units up, looking 6
  units ahead of the player.
- Smooth position and look-at lerping at 4 Hz.
- **Wide 80° FOV** so all 3 AIs are typically visible within the
  frame from the start of a match.

### Turn model
- **No discrete snap** — pure continuous steering.
- Press: turn input ramps from 0 → ±1 over **120 ms** (`TURN_RAMP_IN`).
- Hold: continuous turn at **2.6 rad/sec** (~150°/sec).
- Release: turn input ramps back to 0 over **180 ms** (`TURN_RAMP_OUT`).
- Quick taps register a tiny **10° pulse** so they still feel
  responsive without being snappy.

---

## File Map

```
index.html   ← entire game (HTML + CSS + JS + shaders + audio)
README.md    ← this file
LICENSE      ← MIT
```

The whole game is **~2,200 lines** of single-file HTML. No `node_modules`,
no `package.json`, no bundler.

---

## Tech Notes

- **Three.js 0.160.0** via [esm.sh](https://esm.sh/three@0.160.0).
- **Postprocessing**: `EffectComposer` + `RenderPass` +
  `UnrealBloomPass` + `OutputPass`, all from the official
  `examples/jsm` path through esm.sh.
- **ESM import map** maps `three` and `three/addons/` to esm.sh URLs
  — no bundler, no npm, no `node_modules`.
- **Audio**: Web Audio API, entirely procedural (no audio files).
  An `AudioContext` is created on the first user gesture (the
  ENGAGE click) to comply with browser autoplay policies.
  See [Music](#music) below for the synthwave engine.
- **Persistence**: `localStorage` keys: `tron3d.high` (high score),
  `tron3d.music` (music on/off), `tron3d.endlessUnlocked` (campaign
  clear state).
- **Touch + pointer events**: buttons listen to both so they work
  on every browser engine (Chrome / Safari / Firefox / Android
  WebView) and via headless-Chrome synthetic touches.

---

## Music

A full procedural **synthwave soundtrack** runs alongside the SFX,
all generated in Web Audio inside `index.html` (no audio files, no
licensing concerns).

- **130 BPM**, key of **A minor**, 4-bar loop
- **Chord progression**: Am → F → C → G (the classic synthwave move)
- **4 voice layers**, each a different synth:
  1. **Pad** — detuned sawtooths through a slow lowpass
  2. **Bass** — punchy sawtooth with filter envelope
  3. **Arpeggio** — square wave eighth-note patterns
  4. **Lead** — expressive sawtooth with portamento + filter sweep
- **Intensity scales with progress**: starts sparse (pad only at
  level 1), adds bass/arp/lead as you climb the levels and rack
  up kills. Full 4-voice stack at level 4+ with several kills.
- **Auto-ducking**: every `whoosh` / `crash` / `score` SFX briefly
  drops the music gain so effects punch through.
- **MUSIC checkbox** in the start overlay, persisted via
  `localStorage` (`tron3d.music`).

The result is a recognisable synthwave vibe — analog-style sawtooths,
warm lowpass pads, pulsing bass, bright arpeggios — but fully
synthesised rather than sampled.

---

## Performance

- Trails prune oldest segments every 250 ms rather than every frame.
- Geometry rebuilds use a single `Float32Array` per trail and reuse
  `BufferAttribute`. Worst case is ~7 s × 22 u/s × 4 cycles ≈ 600
  segments × 2 vertices × 3 floats — well under any GPU budget.
- Per-frame color aging uses the same persistent buffer
  (`DynamicDrawUsage`); no per-frame allocations.
- Particles use a single 900-instance `InstancedMesh`.
- Boost trail particles use a 30-point pool with reused
  `BufferAttribute`s.
- Mini-map redraws one 120×120 canvas per frame — trivial.
- Pixel ratio is capped at 2 to avoid melting low-end laptops.
- Tail-end geometry, materials, and ring meshes are disposed when
  effects complete (shield-break, particles, etc.) so nothing leaks
  across matches.

---

## Tuning Knobs

All gameplay constants live in a single `CONFIG` object near the
top of the script block in `index.html`. Common ones:

| Constant         | Default | Effect                          |
|------------------|---------|---------------------------------|
| `BASE_SPEED`     | 22      | Normal forward speed (u/sec)    |
| `BOOST_SPEED`    | 36      | Speed when throttle held        |
| `BRAKE_SPEED`    | 10      | Speed when brake held           |
| `TURN_RATE`      | 2.6     | Max turn rate while held (rad/s)|
| `TURN_RAMP_IN`   | 0.12    | Seconds to ramp turn on press   |
| `TURN_RAMP_OUT`  | 0.18    | Seconds to ramp turn on release |
| `TURN_TAP_PULSE` | 0.18    | One-shot pulse on quick tap     |
| `TRAIL_LIFE`     | 7.0     | Seconds before a segment fades  |
| `ARENA`          | 100     | Half-size of the play area      |
| `AI_COUNT`       | 3       | Number of opponent cycles       |
| `POWERUP_DURATION` | 4.0   | Shield active time (s)          |
| `POWERUP_COOLDOWN` | 12.0  | Shield recharge time (s)        |

Camera lives at the same scope:
```js
const camera = new THREE.PerspectiveCamera(80, ..., 0.5, 600);
```

---

## Known Limitations

- Single-player vs AI only — no multiplayer / online.
- Keyboard / touch only — no gamepad support.
- The shield blocks self-trail collisions only; it does NOT block AI
  trails or arena walls.
- AI uses a single shared personality (hazard avoidance + small
  random heading jitter). No difficulty levels yet.

---

## Roadmap (ideas, not commitments)

- AI personalities (aggressive chaser / wall-hugger / erratic)
- Difficulty modes (easy / normal / hard)
- Kill feed in top-center
- Power-up variety (phase, EMP, speed burst)
- Sound on/off toggle on the start overlay
- Procedural arenas (round, hexagonal, mazes)
- Local split-screen 2-player
- Replay system

---

## Credits

- Game design & code: **Jorge Quijano**.
- Three.js: [mrdoob and contributors](https://threejs.org/), MIT.
- Inspired by the light cycle sequences from the Tron franchise
  (1982 / 2010).
- Built with assistance from Hermes Agent.

---

## License

MIT — see [LICENSE](./LICENSE).