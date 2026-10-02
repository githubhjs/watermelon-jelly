# 🍉 Watermelon Jelly

An interactive 3D jelly watermelon "material study" — drag it to squish it, tap to poke it,
shake the plate, tune its firmness, orbit the camera around it. Single HTML file, loaded via
CDN `<script type="importmap">`, no build step, no bundler.

**Live demo: https://githubhjs.github.io/watermelon-jelly/**

## What it does

- A real 3D soft-body physics simulation deforms a rounded, subdivided box mesh (Three.js
  `RoundedBoxGeometry`) — drag the jelly and the mesh stretches toward your pointer; let go and
  it springs back home. Tap/click without dragging to poke it instead.
- Watermelon coloring (green skin, white rind, red/pink/orange flesh) is painted per-vertex from
  each vertex's height, not a texture — it deforms naturally with the jelly since it's baked into
  the mesh data itself.
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
- Official Three.js addons: `OrbitControls`, `RoundedBoxGeometry`, `BufferGeometryUtils`
  (`mergeVertices`, so the subdivided box shares vertices across faces instead of duplicating them
  per-face — required for the jelly deformation to look continuous instead of seamed), and
  `RoomEnvironment` for the lighting environment.
- The jelly physics itself (structural distance constraints between mesh edges + a per-vertex pull
  back toward its own rest position, with damping) is hand-written — there isn't a drop-in
  "jelly" package for this, but the technique is a standard, well-known one for squishy meshes.

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
