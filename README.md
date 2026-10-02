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
  positions are copied back into the mesh's `BufferGeometry`.
- **Shape memory is jelly, not a balloon.** It's modeled on an earlier 2D canvas version of this
  same project, which gets its shape retention entirely from *shape matching*
  ([Müller et al., "Meshless Deformations Based on Shape Matching"](https://www.beosil.com/download/MeshlessDeformations_SIG05.pdf)) --
  each frame, find the rotation that best aligns the object's rest pose to its current (deformed)
  pose, then pull every point back toward "rest shape, rotated by that angle, centered on the
  current centroid." There is no internal pressure/inflation involved, which matters: pressure's
  natural equilibrium is a sphere, and an earlier pass that leaned on Bullet's own `kPR` (pressure)
  for shape retention visibly re-sphered the wedge's flat faces the higher it was pushed -- a
  balloon, not jelly. This build's `btSoftBody` has no pose-matching exposed in its JS bindings
  (checked by enumerating `btSoftBody.prototype` at runtime), so it's implemented as an explicit
  per-frame post-process: a 3x3 cross-covariance/polar-decomposition rotation recovery, applied as a
  critically-damped spring-toward-goal through velocity (not a direct position snap -- that
  plateaued at a constant wobble energy forever, since Bullet tracks velocity as its own explicit
  state and a silent position change left the old velocity fighting the correction every frame).
  `kPR` is kept at a small fixed value regardless of firmness, just enough to stop the shell caving
  in on first contact (this build's soft bodies are surface-only, no internal volume elements).
  Firmness instead drives shape-match strength plus Bullet's own link stiffness (`kAST`/`kLST`).
- Dragging captures, once at grab-start, every node within range with a distance-based falloff
  weight and a fixed offset from the grab point, then blends each one toward
  "grab target + its offset" every frame (not an absolute teleport, and not re-pulled toward a
  moving target, which clumps the mesh instead of carrying its shape). The soft blend matters for
  more than fidelity to the reference: an earlier hard position+velocity override could shove the
  mesh through the ground rigid body faster than one collision pass could catch, leaving it
  free-falling forever even after release.

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
