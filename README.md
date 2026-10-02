# 🍉 Watermelon Jelly

An interactive 3D jelly watermelon "material study" — drag it to squish it, tap to poke it,
shake the plate, tune its firmness, orbit the camera around it. Single HTML file, loaded via
CDN `<script type="importmap">`, no build step, no bundler.

**Live demo: https://githubhjs.github.io/watermelon-jelly/**

## What it does

- A real 3D soft-body physics simulation (Ammo.js/Bullet `btSoftBody`, running in WASM) deforms a
  rounded wedge mesh — a genuine watermelon-slice cross-section (a curved rind arc tapering to a
  rounded tip, built as a `THREE.Shape`) extruded into a prism with `THREE.ExtrudeGeometry`. It
  falls under real gravity and lands/settles on a physical plate (a static rigid body) on its own —
  there's no scripted "resting pose," it topples and comes to rest the way an actual soft object
  would. Grab the jelly and carry it around; let go and gravity + its own squishiness take back
  over. Tap/click without dragging to poke it instead.
- Watermelon coloring (green skin, white rind, red/pink/orange flesh) is painted per-vertex from
  each vertex's position relative to the wedge's own 2D cross-section outline, not a texture — it
  deforms naturally with the jelly since it's baked into the mesh data itself.
- `MeshPhysicalMaterial` (`transmission`/`thickness`/`clearcoat`) gives it a glossy, faintly
  translucent, candy-like look; a procedural `RoomEnvironment` provides soft studio lighting with
  no external HDRI file needed.
- Drag the jelly itself to squish it; drag empty space to orbit the camera around it
  (`OrbitControls`, with a slow idle auto-rotate when you're not interacting). A raycaster tells
  the two apart.
- "The Specimen" panel: a flesh-color palette, Firmness/Internal-dampening sliders (live-tunable
  physics constants), Give-it-a-nudge/Shake-the-plate/Reset/Pause, and a live spec-bar readout
  (mass/volume flavor text + a real "wobble energy" reading computed from the mesh's own vertex
  velocities).
- A tiny synthesized "boing"/"shake" sound (Web Audio oscillators, no audio files) on interaction.

## Tech

- [Three.js](https://threejs.org/) r186, loaded via an import map pointing at jsDelivr — no local
  install, no bundler.
- Official Three.js addons: `OrbitControls`, `BufferGeometryUtils` (`mergeVertices`, so the
  extruded wedge shares vertices across faces instead of duplicating them per-face — required for
  the jelly deformation to look continuous instead of seamed), and `RoomEnvironment` for the
  lighting environment. Also `three-subdivide` (`LoopSubdivision`) for one smoothing pass that
  rounds the low-poly extrusion into a plumper, jelly-like shape.
- Physics: [Ammo.js](https://github.com/kripken/ammo.js) (Bullet physics compiled to WASM), the
  same build Three.js's own official `physics_ammo_volume.html` example uses — a `btSoftBody` per
  mesh vertex (1:1, since the geometry is already a single welded indexed mesh), plus a static
  rigid-body ground/plate. Per frame, `physicsWorld.stepSimulation()` runs and the resulting node
  positions are copied back into the mesh's `BufferGeometry`. Dragging captures a fixed offset from
  the grab point for every node once, at grab-start, and moves them rigidly together each frame
  (not re-pulled toward a moving target, which clumps the mesh instead of carrying its shape).
  Shape retention relies on angular/linear link stiffness (`kAST`/`kLST`) plus a small constant
  internal pressure (`kPR`) — just enough to keep the shell from caving in under gravity, since
  this build's soft bodies are surface-only (no internal volume elements) and don't expose
  pose-matching (`setPose`) in its JS bindings.

## Running locally

It's a single static file that uses ES module imports, so it needs to be served over HTTP (not
opened directly as a `file://` URL):

```bash
python3 -m http.server 8080
```

## Earlier 2D version

An earlier iteration rendered the jelly as a flat 2D canvas wedge (perimeter spring ring, no real
3D). See the git history (commits before the Three.js rewrite) if you want to look at that
simpler approach.

## Author

Chih-Hsueh "Josh" HUANG <huangjs@gmail.com>
