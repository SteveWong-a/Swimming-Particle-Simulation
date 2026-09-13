# AQUAKINETICS — Cyber-Aquatic Swim Telemetry

An interactive 3D particle simulation that puts competitive swimming dynamics and
computer-vision telemetry side by side. 11,500 GPU particles render a shader-animated
25-yard competition lane and a procedurally generated freestyle swimmer, with a mode
toggle that contrasts how camera-based tracking fails on a pool deck against clean
sensor-derived kinematics.

Everything lives in a single [index.html](index.html). No build step, no bundler, no
assets — open the file and it runs.

## Running it

Open `index.html` directly in any modern browser, or serve the folder:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

Three.js r128 and OrbitControls are pulled from CDN, so the first load needs a network
connection.

## Controls

| Input | Action |
| --- | --- |
| Drag | Orbit the pool (damped) |
| Scroll / pinch | Zoom |
| Move or drag the pointer over the water | Raycast a ripple shockwave into the particle grid |
| `1` `2` `3` | Deck Side, Underwater Profile, Top-Down Lane |
| `M` | Toggle simulation mode |
| Track Athlete | Lock the camera rig to the swimmer while preserving your orbit angle |

## What's being simulated

**System 1 — wave pool (8,000 particles).** A 100 x 80 jittered grid spanning the lane,
displaced entirely in a custom vertex shader: two crossed sine/cosine swells, a
travelling chop wave, three octaves of 3D simplex noise, a trailing V-wake anchored to
the swimmer, and up to 12 simultaneous damped radial shockwaves from pointer input.
Lane-line particles pick up rope-float banding so the course geometry stays readable.

**System 2 — parametric swimmer (3,500 particles).** Body parts are encoded per vertex
(`part`, `t`, `ring`, `side`) and evaluated in the vertex shader: a tapered ellipsoid
torso, a streamlined head with a breathing yaw, two arms sweeping a catch/pull/recovery
arc 180 degrees out of phase, and legs driven by a travelling six-beat flutter wave.
The body rolls on the swim axis, surges within each stroke cycle, and spins through a
streamlined turn at each wall.

**Mode A — Algorithmic Noise (Computer Vision View).** Submerged particles are jittered
by refraction noise and punched out by occlusion and frame dropout. A canvas overlay
projects the swimmer's landmarks into screen space and draws flickering bounding boxes,
phantom splash detections, and a confidence score that collapses as turbulence rises.
Stir the water with the pointer and watch the tracker lose the athlete.

**Mode B — Streamlined Kinematics (Human-Centered View).** Noise amplitude collapses,
particles channel toward lane centres and stream down the lane, and the HUD reports lap
splits, stroke rate, velocity, and distance per stroke computed from the same stroke
phase that drives the shader — so the numbers always match the stroke on screen.

## Implementation notes

- All particle motion runs in `ShaderMaterial` vertex shaders; the JS loop only advances
  time, eases the mode mix, maintains the ripple ring buffer, and derives HUD values.
- The swimmer body constants live in a single `B` object that is interpolated into the
  GLSL source and reused by the JS landmark solver, so the HUD boxes cannot drift away
  from the particles.
- Pixel ratio is clamped to 2, point sizes are clamped in pixels to bound overdraw, the
  loop allocates nothing per frame, and rendering pauses when the tab is hidden.
- Portrait viewports automatically pull the camera back and pitch it down so the lane
  still fills the frame on a phone.
