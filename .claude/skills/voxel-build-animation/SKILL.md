---
name: voxel-build-animation
description: Build a single self-contained HTML page that animates a 3D voxel scene assembling itself from the ground up (three.js, instanced cubes), usually recreated from a reference photo of a building or diorama. Use when the user asks to "build X from voxels", "voxel animation of this photo", "make this house/scene out of cubes that builds up", or wants another floating-island voxel build-up scene.
---

# Voxel build-up animations

Deliverable: **one `.html` file** in the project folder (no artifact publishing unless asked). It renders a voxel
scene with three.js on a floating island and animates every voxel into place from the bottom up, then holds the
finished scene. The look is semi-realistic: muted photographic colours, soft light, aerial haze, no visible cube grid. Start
from [template.html](template.html) — it holds the whole engine; you write the scene.

## Reference files (read what the scene needs)

| File | Read when |
|---|---|
| [reference/scale-and-props.md](reference/scale-and-props.md) | always — scale table, real sizes, sprites, cars, boats, trees, animals |
| [reference/colour-and-materials.md](reference/colour-and-materials.md) | always — sampler workflow, saturation limits, real-world albedos, shades per material |
| [reference/terrain.md](reference/terrain.md) | shaping the island, cliffs/plateaus, lakes, diagonal bridges, the fade-out ground |
| [reference/buildings.md](reference/buildings.md) | any building: windows, glass, roofs, towers, railings, the alignment pass |
| [reference/nature.md](reference/nature.md) | trees, meadows, crops, surroundings, imperfection |
| [reference/lighting.md](reference/lighting.md) | light defaults, haze, lamps, AO, sun/sky/camera, night, bloom, fire and smoke |
| [reference/animation.md](reference/animation.md) | changing animation modes or the engine's animation code |
| [reference/inspection.md](reference/inspection.md) | step 6 — prop inspection with subagents |
| [reference/verification.md](reference/verification.md) | step 5 — serving, test hooks, checks, the photo comparison |
| [palette-sampler.html](palette-sampler.html) | step 2 — sampling colours from the photo |

## Engine in brief

- **Two grids.** Plan in *design* units (`M_PER_UNIT`, 0.4 m default); everything rasterises on a *fine* grid
  `S×` finer (`S = 3`, fixed: scenes are built at detail 3 only). `box()/shell()` take design units, `fbox()/fset()` fine cells, `dc(f)` maps
  back. Organic shapes (`blob`, `disc`) rasterise on the fine grid. `cells(m)` converts metres to fine cells.
- **Helper library** (template, between the island and the demo scene) — use these before writing your own:
  - lines: `limb()` (design points), `fline()` (fine points), both face-connected;
  - building: `shell`, `win` / `fwin` (fine cells: arch, voussoir ring, grille, plank door), `hip`, `roofU` (one
    hipped roof over several rectangles), `roofG` + `gable` (gabled/hipped ends per rect, `roofTop` for what stands on
    it), `timberC` (half-timbering), `roundCut` (arcades), `recess` (loggias, balconies), `oculus`, `stair`,
    `railing`, `roundTower` + `rWin`/`rSlit`/`coneRoof`, `gothicDoor`, `archCut`, `lathe` (any round mass whose
    radius varies with height: towers, domes, saucer decks — skin only, `up`/`down` flags for roofs and undersides);
  - light fixtures: `lantern`, `wallLamp`, `uplight`, `uplightAt`, `lampPost`, `fireLamp`; any other glow → `lamp()`;
  - plants on the terrain: `tree` (`{ low, lean, sy, spread }`), `spruce`, `cypress`, `shrub`;
  - structures and props: `deck`, `lattice`, `trussX`, `sprite` / `q` for detailed props;
  - sky: `useSkyTexture()` draws `SKY` behind the scene (on by default; `?bg=css` falls back to the body gradient).
- **Voxel store:** a `Map` keyed by `key(x,y,z)` with a **+256 offset** — fine y goes negative under the island,
  and a smaller offset silently corrupts keys. Scene globals must not reuse engine names (`SX`, `SY`, `SZ`, `PV`,
  `REL`, `VX`, `VY`, `VZ`, `W`, `top`, `bottom`, `idx`, `moved`, `full`, `changed`, `dpr`, `UP`): a clash is a SyntaxError and a blank page.
- **Three instanced meshes** chosen by `fset(x,y,z,color,e)`: `e = false` lit and shadowed; `e = true` glowing
  (fades in after the build); `e = 'glass'` transparent physical material.
- **Tags:** `STAGE` (build order for `stages`), `TAG` (pass behaviour: 4 terrain, ≥ 6 hollowed, 7 animal, 9 scruff grass …),
  `prop(name, [w,h,l])…prop()` (inspection). A voxel overwritten later takes the later values.
- **Passes after building:** scruff → hollowing → connectivity → bake (AO, per-face sun shadows, weathering, lamps;
  it also fits the shadow camera to the voxel bounds) → palette lint → prop report.
- **Look controls:** `P` (palette, linted), `SKY` (texture stops), `LIGHT` (hemi/sun), `HAZE` (depth fade),
  `WEATHER`, `BAKE` (`ao`, `reach`, `gain`, `sunShadow`, `sunReach`); URL flags `?grid=1` (full cube grid),
  `?cam=persp` (perspective hero view), `?bg=css`, `?shadow=lo|hi` (1024 / 4096 shadow map while voxels move; default 2048), `?dpr=N` (pin the pixel ratio; by default it steps down when frames run slow).
- **Animation:** voxels sorted by start time `t0` (*order*) and moved by a `POSE` function (*motion*); matrices
  go straight into `instanceMatrix.array` and a moving `first` pointer keeps each frame to the active window.
- **Frames are drawn only on change.** Code that moves voxels calls `uploadRange()`/`uploadAll()` (they set
  `moved`); anything else that animates over time sets `moved = true` or joins `changed()` — else it freezes once the
  build settles. Details: [reference/animation.md](reference/animation.md).
- **UI:** fps readout top left (always shown). Every control lives in a popover opened by the burger, bottom left
  (Esc closes it): Replay (R), Pause (Space), progress bar, then three sections. *Animation*:
  Order/Motion/Sky (live, synced to the URL), Orbit. *Render*: Camera, Background, Shadow map, Pixel ratio,
  Cube grid: the URL flags above, applied by reloading (`data-q` elements; empty value = default). *Debug*: Force
  time, Baked lamps, Sun shadow, Hero lights: the `__night`/`__bake`/`__shadow`/`__hero` overrides, live, not in
  the URL. A new URL flag gets a `data-q` control there. Keys: F skip to end, D demolish, P pile, H hide burger +
  menu. Dragging hands the camera to OrbitControls.

## Workflow

1. **Read the photo.** List masses (blocks, wings, roof type), openings, materials, time of day and
   surroundings. Imagine the unseen sides (terrace, garden, side door, drive) — the back and sides are part of
   the scene, never left blank.
2. **Sample colours, don't guess them.** Copy the photo to `ref/<name>.webp`, open the sampler from the served
   folder, e.g.
   `/.claude/skills/voxel-build-animation/palette-sampler.html?img=../../../ref/house.webp&crops=stone:0.17,0.47,0.27,0.66;roof:0.76,0.495,0.92,0.51&sky=0.72,0.80,0.47`
   (crops `name:x0,y0,x1,y1` fractions; `sky=x0,x1[,yBottom]`), and read `JSON.stringify(window.__palette)` (hex,
   share and HSL per cluster, plus a `css` gradient). Crop placement, saturation limits, albedo checks and shade
   counts: [reference/colour-and-materials.md](reference/colour-and-materials.md).
3. **Plan in design units** against the scale table. Write key coordinates as constants relative to an origin
   (`X(u) = OX + u`).
4. **Copy `template.html`** to the scratchpad and build from that snapshot with a splice script of asserted string
   replacements (the template can change while a scene is in progress, and asserted anchors fail loudly instead of splicing
   into the wrong place). Rename
   title/palette, set `M_PER_UNIT` (`S` stays 3), fill `SKY` from the sampled sky, and write the scene: `STAGE` per part, every prop wrapped in `prop()` with its real size, light only from fixtures.
5. **Verify in the browser pane** from front, back, both sides, zoomed in, and from the photo's angle with
   `__compare()` against the photo. Read the palette lint. Fix, re-check.
6. **Inspect every prop with subagents** (read-only, 3–5 groups in parallel); apply their fixes, re-inspect what
   failed.
7. **Report briefly:** what's in the scene, controls, the inspection and comparison results, anything not verified.

## Rules for every scene

- **Nothing floats.** The connectivity pass flood-fills from one ground seed through face neighbours and deletes
  the rest (`connectivity: removed N`). Thousands removed means something real broke — find it.
- **Everything touches face-to-face**, never only at an edge or corner: `limb()` for lines and branches (steps
  one axis at a time), sloped surfaces two voxels thick, brackets for wall fixtures, a branch into every leaf
  blob. Cantilevers are fine if attached.
- **Built at S = 3 only:** cell sizes written for S = 2 go through `c2(n)` (`h2(v)` back), metres through
  `cells(m)`, design → fine conversions are floored (x.5 cells at S = 3 touch nothing and get removed). Other
  detail levels are not supported; don't test or fix for them.
- **Props at 1.0–1.2× real size** — 1.5× reads as a toy, 0.7× as a model. If detail is too coarse, draw it in fine
  cells (`fset`, `sprite`), don't enlarge the prop.
- **Light comes only from fixtures** (`lantern()`, `uplight()`, `lampPost()`, lit `win()`, or `lamp()` tied to a
  glowing voxel) — never a light at a bare coordinate, and no real `PointLight`s.
- **Nothing is clean or perfect.** Break every repeat, add wear where things are used and a little clutter of
  life; aim for lived-in, not ruined.
- **Openings read dark** and **glass is clearly visible** with a closed room behind it.
- **Hollow every solid mass** (`TAG ≥ 6`) to keep the voxel count down.
- **Island generously wider than the building** (half-width + 15–20 units), irregular outline, rounded bumps.

### Look

- **Photographic, not toy-like.** Every palette colour is sampled from the photo and stays within sRGB saturation
  ≤ 0.4 and lightness 0.12–0.85 (the lint warns); only glow, flowers and signage (`ACCENT`) may exceed it.
  Saturation lives in small accents, never in grass, walls or roofs.
- **Soft light.** Keep the `LIGHT` defaults (hemisphere 1.0, sun 1.2, exposure 0.95, warm sun, cool sky) unless the
  photo says otherwise; the sun is never more than ~1.5× the hemisphere in daylight. Shadows and AO give form;
  colour contrast does not.
- **Haze and sky.** `HAZE` fades distant voxels toward the sky's horizon band; `SKY` has a lighter band at the
  horizon. Both stay on. The sky colours come from the sampler's `sky` strip.
- **No visible cube grid.** The outline border is near-invisible (`#f4f4f4`); `?grid=1` only for debugging
  geometry. Surfaces read through shade variation, weathering and AO, not through cube edges.
- **Compare with the photo**, not with memory: `__compare('ref/x.webp')` prints render vs photo per region; a
  saturation difference above ~0.1 or lightness above ~0.12 in a material region means resample that material.

## Keeping this skill current

**The skill is the single source of truth.** Everything a new scene needs lives here: the engine in
`template.html`, the rules in this file and `reference/`. Generated scenes (the `.html` files in the project) are
outputs, not sources: they may be outdated, edited by hand or deleted.
- Never name, link, copy from or read a generated scene to build a new one; start from `template.html`.
- A technique, helper or fix that a scene invents moves into `template.html` or the matching reference file,
  written generically (a helper and its signature, a rule with its numbers), and the scene then uses that version.
- Before finishing, grep the skill for scene names and scene file names, and remove any.

When a session learns something — a correction, a look that doesn't work, a bug and its fix, a better default — update the
skill before finishing:
- Put it in the reference file it belongs to, next to related entries; merge with an existing entry rather than
  adding a near-duplicate. Only rules that apply to every scene go in this file.
- Write the rule first, then the reason in one clause, keeping the numbers. Present tense only: no past tense, no
  "the user", no scene names, no session history — a rule and its reason, never the story of how it was learned.
- Keep each entry to 3 lines or fewer. Correct or remove anything that turned out wrong.
- Update `template.html` if the engine changed.
