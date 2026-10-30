---
name: voxel-build-animation
description: Build a single self-contained HTML page that animates a 3D voxel scene assembling itself from the ground up (three.js, instanced cubes), usually recreated from a reference photo of a building or diorama. Use when the user asks to "build X from voxels", "voxel animation of this photo", "make this house/scene out of cubes that builds up", or wants another scene in the same style as the existing voxel-*.html files.
---

# Voxel build-up animations

Deliverable: **one `.html` file** in the project folder (no artifact publishing unless asked). It renders a voxel
scene with three.js and animates every voxel dropping into place, layer by layer from the bottom up, then holds
and loops. Start from [template.html](template.html) — it contains the whole engine; you only write the scene.

**Keep this skill current.** Whenever a session learns something new — a user correction, a rejected look, a bug
and its fix, a technique that worked, a better default — update this SKILL.md (and `template.html` if the engine
changed) before finishing, in the section it belongs to. Write the lesson and why it matters, not the session's
history. Remove or correct anything that turned out to be wrong.

Working examples in this project (read the one closest to the request before writing):
`voxel-starship-launch.html` (non-building subject at real scale, `S = 3`, lattice tower, fire + smoke, bloom,
detailed vehicle/crane/container sprites — copy its prop helpers),
`voxel-fieldstone-house.html` (baked AO + baked uplights, arcade, union hip roof, terraced ground),
`voxel-manor.html` (glass, glow, detailed facade, imagined back garden, full orbit),
`voxel-cabin.html` (hilly terrain heightmap), `voxel-concrete-house.html` (night scene, bloom),
`voxel-cottage.html` (first style, square base).

## Workflow

1. **Read the photo.** List the masses (blocks, wings, roof type), openings (windows, doors, arches), materials,
   lighting/time of day, and surroundings. Decide what the unseen sides look like — the user wants
   the hidden back and sides *imagined*, not left blank (terrace, garden, side door, drive, etc.).
2. **Sample the colours, don't guess them** (see *Palette from the photo*): wall, roof, trim, lawn, foliage and
   the sky gradient. Eyeballed hex values were the reason the sky had to be redone after delivery.
3. **Plan in design units using the scale table** (see *Scale*). Write the key coordinates down as constants
   (footprints, floor heights, eave height, plane positions).
4. **Copy `template.html`**, rename title/palette, and write the scene section with the helpers below. Light
   comes only from fixtures: `lantern()`, `uplight()`, `lampPost()`, lit `win()` rooms, or `lamp()` tied to a
   glowing voxel (see *Baked lighting*) — never a light at a bare coordinate.
5. **Verify in the browser pane** (see *Verification*) from front, back and both sides, zoomed in, and once from
   the photo's angle next to the photo. Fix, re-check.
6. Report briefly: what is in the scene, controls, anything not verified.

**Upgrading an older `voxel-*.html` to the current template** ("regenerate X according to the skill"): keep its
scene section verbatim and splice it between the template's helpers and engine (a small script with asserted
string replacements is safer than hand-editing 1000 lines; back up the old file first). Then: delete its own
`bakedLights` array, add `LAMPS`/`lamp()`, register a lamp for every glowing fixture it has (globes, lit rooms,
floor lamps, lamp posts — and add one or two so `sky=night` has light), set `STAGE` before ground/walls/
openings/roof/props and `TAG = 4` around the terrain loop, and move scene-specific engine values (VIEW, target,
camAz, hemi/sun colours and intensities, BAKE.ao, WEATHER) into the new engine — `setSky` must scale the
scene's own hemi/sun intensities, not the template's 1.6. Check the console: "lamps without a glowing source
voxel dropped" means a `src` cell isn't glow — e.g. `A0 + 3` falls outside a 3-cell-wide window; use the pane
centre.

## Scale (1 design unit = 40 cm; at `S = 2` a fine voxel is 20 cm)

Proportions are the first thing that makes a scene look like a toy. Check every element against this table.

| Element | Design units | Notes |
|---|---|---|
| Storey (floor to floor) | 8 | one-and-a-half storey wing ≈ 13 to the eave |
| Plinth / string course | 1 fine voxel proud | string course on the upper floor line |
| Door | 2.5–3 wide × 6–7 high | threshold step in front |
| Window | 3–6 wide × 4–5 high | sill ≈ 2 above the floor; an arched head adds about half the width |
| Arcade arch | 4 wide, pillars 1.5–2.5 | springs ≈ 3.5 above the ground; the interior must stay dark |
| Eave overhang | 1 (cottage) – 2 (Italianate) | soffit + fascia; hipped roofs rise 0.4–0.55 fine per fine run |
| Chimney | 2–4 square | a tall stack ends 1–2 storeys above the ridge; cap + pots |
| Retaining wall / step | 2 high / 0.5 rise, 1 deep | |
| Chair / table | 2×2 fine; seat 1 fine up, table 3 fine high | |
| Car | 3 wide × 7 long × 3 high | |
| Garden tree | trunk 2 fine, height 9–12, crown radius 3.5–5 | woodland 11–20 |
| Person (if any) | 4.5 high | |

### Big subjects and props at real size
- The table is for houses. For a large subject (rocket, tower, ship, factory) pick metres per design unit from the
  photo (measure two known sizes in pixels) and write it at the top of the scene; `voxel-starship-launch.html`
  uses 1 unit ≈ 1.1 m.
- **Every prop must be 1.0–1.2× its real size.** The user rejected vehicles at 1.5–1.7× ("make everything max
  1.1–1.2x original size"). Check each against real dimensions: pickup 2.0 × 5.9 × 1.9 m, semi tanker 2.5 × 18 m,
  20 ft container 2.44 × 2.59 × 6.06 m, crawler-crane track ≈ 9 m, door 2.1 m, jersey barrier 0.6 × 0.8 × 3.7 m,
  traffic cone 0.7 m, bollard 1 m, hatchback 1.8 × 4.1 × 1.5 m, wheelie bin 0.6 × 0.75 × 1.05 m, wheelbarrow
  1.4 m long, garden bench seat 0.45 / back 0.85 m, globe wall lamp 0.25 m, slim metal flue 0.35 m. The island
  may grow to make room. This applies to houses too: the green house's car had been modelled at 0.7× and its
  globe lamps as 40 cm cubes (a 2×2 fine block is already 40 cm at `S = 2`; small fixtures get 1 cell).
- Measure openings from the photo with a scale bar on the same wall (eave height or a door), including sill and
  head heights, not just widths: windows had been placed ~0.6–1 unit too low and attic windows 30% too wide.
  Perspective makes the near side of a photo larger; compare items with a reference at the same depth.
- If real-size props come out too coarse ("details on cars, crane etc. are very low"), make the grid finer
  (`S = 3`, 0.37 m cells at 1.1 m/unit) instead of enlarging the props. ~620k voxels at `S = 3` ran ≈ 47 fps with
  a 0.8 s bake — acceptable.
- Build props as **sprites** authored in fine cells (`sprite(ox, oy, oz, rot)` with `q(n) = round(n·S/3)` so they
  scale with `?res`): a local box/put API, 90° rotations, and `.w()` to get design coordinates for `limb()`.
  The starship file has `pickup` (SUV variant), `tanker`, `container`, `dish`, `latticeBoom`, a crawler crane,
  a generator, ladders on tanks. Detail that reads at this size: separate wheels with hubs and arches, glass with
  seats behind it, pillars, mirrors, grille + glowing head/tail lights, bumpers, cargo; container corrugation,
  corner castings, door seam, lock bars, rust, stencil; crane lattice boom with zigzag lacing, pendants, hoist
  rope, striped hook block, a hung load, hazard-striped counterweight, glazed cab, tracks with rollers on mats.
- **Cars: match the body type's side profile**, not just the size. Read it from the photo (hatchback, sedan,
  estate, SUV): a hatchback has a short hood (~0.8 m), a raked screen, and a roof running to a near-vertical
  tailgate with a spoiler; a separate low deck behind the cabin makes it read as a sedan (the user flagged it).
  Keep 0.8 m overhangs and a ~2.5 m wheelbase. `voxel-green-house.html` has a hatchback sprite in 20 cm cells
  (`q(n) = round(n·S/2)`): wheel arches with hubs, inset sills, wrap-round tail lights, A/B/wide-C pillars with
  a quarter light, seats and headrests, mirrors, door seam and handles, plates, shark fin.
- Lattice structures: `lattice()` (corner columns, ring beams, X braces on all faces) and `trussX()` (chords,
  verticals, diagonals). Keep a crane boom's width ~1/3 of the crane body or it looks like a solid plank.

## Palette from the photo

[palette-sampler.html](palette-sampler.html) clusters crops of the photo into 3–4 colours each and turns a
vertical strip of sky into gradient stops. Copy the photo into the project (`ref/<name>.webp`) and open the
sampler from the served folder in the browser pane, for example:
`/.claude/skills/voxel-build-animation/palette-sampler.html?img=../../../ref/house.webp&crops=stone:0.17,0.47,0.27,0.66;roof:0.76,0.495,0.92,0.51;lawn:0.1,0.82,0.9,0.98&sky=0.72,0.80,0.47`
(crops are `name:x0,y0,x1,y1` fractions; `sky=x0,x1[,yBottom]`). Read the result with
`JSON.stringify(window.__palette)`; it includes a ready `css` gradient for `body`.
- Choose the sky strip where nothing (trees, chimneys) crosses it; the last stop is often contaminated — drop it.
- Sampled colours already include the photo's lighting. Treat them as the *rendered* target: put them in the
  palette, render, and compare a screenshot of the same area. Lamp-lit areas (uplit stone) should come out
  right through the baked lamps, not by brightening the base colour.
- Keep 3–4 shades per material plus a distinct mortar/joint colour; the cluster shares say how often each
  shade should appear.

## Engine model (what the template gives you)

- **Two grids.** Plan in *design* units; everything is rasterised on a *fine* grid `S×` finer
  (`S = 2` default, `?res=1..4` in the URL). `box()/shell()` take design units; `fbox()/fset()` take fine cells;
  `dc(f)` maps a fine cell back to design space. Organic shapes (`blob`, `disc`, `pine`, `cone`) rasterise on
  the fine grid so curves stay smooth.
- **Voxel store** is a `Map` keyed by `key(x,y,z)` with a **+256 offset** — fine y goes negative under the
  island; a small offset silently corrupts keys and whole regions vanish.
- **Three instanced meshes**, chosen per voxel by `fset(x,y,z,color,e)`:
  - `e = false` → lit `MeshStandardMaterial` (casts/receives shadows);
  - `e = true` → glowing `MeshBasicMaterial` (lamps, lit rooms, candles, headlights) — fades in after the build;
  - `e = 'glass'` → semi-transparent `MeshPhysicalMaterial` (see *Glass*).
- **Animation:** voxels sorted by a start time `t0` (the *order*), each animated for `DUR` by a `POSE` function
  (the *motion*); see *Animation modes*. Matrices are written straight into `instanceMatrix.array`
  (`setM(v, x, y, z, s, sy)`, so scale can differ vertically); a moving `first` pointer keeps the per-frame
  loop to the active window, so 250k voxels stay smooth.
- **Bake pass** (after connectivity, before rendering): ambient occlusion and the lamps registered by fixtures
  (`LAMPS` → `bakedLights`) are written into the lit mesh: AO into instance colours, lamp light into a per-instance
  `lamp` attribute added as emission — see *Baked lighting*.
- **UI:** Replay (R), Pause (Space), F skips to the end, D demolishes, P piles, H hides the UI, Loop and Orbit checkboxes,
  Intro/Order/Motion/Detail/Ending/Sky dropdowns (Detail = `?res`, reloads the page because the grid is rebuilt), info line
  with voxel count. Dragging hands the camera to OrbitControls (`userMoved`). Test hooks:
  `__seeAz(angle)` frames any side, `__zoom(z, x, y, z)` zooms the ortho camera onto a world point,
  `__bake(true|false)` forces baked lamps on/off for comparison (`__bake()` returns to automatic),
  `__hero(false)` hides hero PointLights, `__lamps` lists the baked lamps.

## Animation modes

`ANIM = { intro, order, motion, outro, sky }` at the top of the template sets the scene's defaults; `?intro=`, `?order=`, `?motion=`,
`?outro=`, `?sky=` in the URL override them. The UI has **Intro / Order / Motion / Ending / Sky dropdowns** that switch live: order
and motion re-run `plan()` (recompute `t0`, re-sort, new `DUR`/`BUILD_END`) and replay; the URL is updated
with `history.replaceState` so a combination can be shared. Picking *3D print* also switches the motion to
*Grow*. Default is `layers/drop/reset`, the original look. Anything indexed by position in `vox` (the demolish
arrays `REL`/`VX`/…) must be dropped when `plan()` re-sorts.

| Key | Value | What it does |
|---|---|---|
| order | `layers` | bottom-up, hash jitter + a little centre-out spread |
| | `scan` | 3D printer: strict layers (`SCAN` s each), rows rastered along z in alternating directions, a translucent orange plate rides on the current layer. Defaults the motion to `grow` |
| | `stages` | by the `STAGE` tag set in the scene: each stage builds bottom-up from its own lowest voxel and the next starts at ~80% of it |
| motion | `drop` | falls `6*S` cells with an ease-out-back scale pop (`DUR` 0.55 s) |
| | `grow` | extrudes upward from the voxel's floor with slight overshoot (0.45 s) |
| | `swarm` | starts in a cloud up to `32*S` cells out and above, swirls in on a curve, scales up (1.6 s) |
| | `tornado` | rides a funnel around the island centre: wide and high, spiralling down and in (1.8 s) |
| | `slide` | slides `9*S` cells in along its open side facing away from the centre (walls, windows); voxels with no open side (lawn, roof) drop in (0.7 s) |
| | `pop` | no travel, elastic scale-up in place (0.6 s) |
| | `tumble` | falls from higher while spinning about a per-voxel random axis, lands square (0.9 s; uses `setR`) |
| | `squash` | accelerates down stretched, flattens on impact and springs back, volume kept (0.75 s) |
| | `flash` | drops in glowing hot white and cools to its real colour (0.9 s; writes instance colours) |
| intro | `none` | build starts at once |
| | `blueprint` | a blue outline of every exposed non-terrain voxel fades in for `BLUEPRINT` (2.5 s; `reset` starts `t` negative), the build fills it in, it fades out when complete |
| outro | `reset` | hold 30 s, restart |
| | `demolish` | hold 30 s, then come apart in **reverse build order** (pieces thrown outward, gravity, shrink), then rebuild. `D` triggers it any time |
| | `pile` | everything above the terrain arcs onto a heap where the building stood (last built at the bottom), holds, then flies back in build order so the heap unstacks from the top. `P` triggers it |
| sky | `day` | stays as the photo |
| | `night` | 2 s after the build, dusk fades in: `#night` gradient overlay, hemisphere and sun dim, hemisphere tints blue, glow and hero point lights are pushed to ×2.2 (baked lamps are emissive and need no boost); the UI switches to light text (`body.dark`) |

- **Stages:** set `STAGE = n` before each part of the scene (template: 0 ground, 1 walls, 2 windows/doors/
  lanterns, 3 roof, 4 trees/props; scruff added later inherits the last value). A voxel overwritten later takes
  the later stage, so windows punched into a stage-1 wall appear as stage 2. Unset scenes fall back to one stage.
- **Demolish order must follow `t0`, not height.** Keyed to absolute height, the lawn and the house released
  together (the deep island makes everything on top "high") and it read as one explosion.
- The island interior is culled, so `scan` shows a hollow bowl while printing the underside — it reads as
  printer infill; leave it.
- Add a motion by adding a `POSE.name(v, p)` that ends at `(v.x, v.y, v.z)` with scale 1 as `p → 1`, its
  duration to `DURS`, and an `<option>` to the Motion dropdown. `setM` takes separate x/y/z scales and always
  clears the rotation terms, so a rotated voxel (`setR`) lands square. A motion that changes colour needs a
  `POSE.name.done(v)` to restore it when the voxel settles (see `flash`); switching motion calls
  `restoreColours()` so nothing is left tinted.
- **Blueprint ghost writes depth** and draws last. With `depthWrite: false` the hundreds of overlapping translucent
  faces stacked into a milky white blob; it is also scaled 0.96 so landed voxels hide it.
- Effects that move "the building" must skip terrain with `v.y > topAt(v.x, v.z)`, not `v.y >= TOP`: lawn
  hummocks are above `TOP`, and pulling them away exposed the culled hollow island as dark holes.
- **Morph to another scene** is not built yet: `pile` already pairs every voxel with a slot, so morphing means
  using another scene's voxels as targets (paired by colour). That needs scenes exported as data, since each
  `voxel-*.html` generates its scene inline.
- `slide` must choose the open side facing **away from the island centre**: the first open side found is often
  the inside of the room, so side walls slid in from inside the house.

## Ground: the floating island (the style the user settled on)

- **Outline:** irregular, never a circle/oval. Rounded-square base (superellipse norm `(|dx|^4+|dz|^4)^¼`), then
  `edgeR()` adds **lobes** (harmonics 2, 3, 5 with random phase/strength → bays and peninsulas), a **ragged
  coastline** (2D value noise at scales 9 and 3.5), and soft rounded rim bumps. Change `ISLAND` (the outline seed)
  for a different shape; keep the lowest point of the outline clear of the building and props. Rim bumps are a
  rounded bell `sin²(π·f)` per segment — **never pointy** (the user rejected the sharp `tri^3.5` thorns) — and
  **few and mostly low**: `K ≈ 14` around the rim, amplitude `0.6 + hash² × 5` so only a handful stand out;
  30–40 sharp spikes looked like a sea urchin. Make the island **generously wider than the
  building** (the user asked twice for more room): radius ≈ building half-width + 15–20 units.
- **Underside:** **one** iceberg-like rock mass tapering to a single off-centre point, faceted with stepped noise
  (`floor(noise*5)/5`). *Not* many separate hanging spikes. Thin at the rim (`min(1, e/4)`), deep in the middle.
- **Surface:** lawn / moss / gravel / mulch by region functions; grass tufts and flowers as 1–3 voxel stalks.
  Avoid dark patches (dark heather, exposed brown soil on slope steps) — they read as **holes**. On slopes the
  top `2*S` voxels of every column use the surface colour so steps look grassy.
- **Hidden-voxel culling** (only for terrain columns): skip a voxel when every neighbour column covers it, using
  *strict* comparisons so one layer of overlap remains: hide if `nbot < fy && ntop > fy` for all 4 neighbours.
  With `<=`/`>=` the shell is joined only diagonally and the connectivity pass deletes it.

## Rules that keep the scene solid

- **Nothing floats.** After building, flood-fill from **one seed on the ground surface** through the 6 face
  neighbours and delete everything unreached; log the count (`connectivity: removed N floating voxels`).
  Do not seed from every bottom voxel — detached pillars would count as "grounded".
  A large removed count (thousands) means something real broke — find it, don't ignore it.
- **Everything must touch face-to-face**, never only at an edge or corner:
  - lines/branches/poles: use `limb()`, which steps one axis at a time;
  - sloped surfaces (roofs, tent tops, roof glass): make them **two voxels thick** so adjacent steps share a face;
  - canopies: run a branch from the trunk into each leaf blob;
  - lanterns, sconces, signs: give them a bracket voxel touching the wall.
- Cantilevers and overhangs are fine if they are attached.

## Detail checklist ("details matter")

Buildings
- Walls as one-voxel `shell()`s with a material function (fieldstone: offset courses + mortar; boards: per-board
  hash colour; concrete: vertical streak noise). Add a plinth course and a string course between floors.
- Windows via `win()`: frame + mullions on the wall plane, **glass recessed one voxel**, frame continues inward as
  a reveal, the **room two voxels behind the glass** (warm glowing when lit, with curtains; dim otherwise),
  stone sill, arch band, keystone; round-arched tops where the photo has them. Doors the same with warm glass.
- **The room backdrop must be closed behind every pane, mullions included.** Walls are hollow shells; a gap
  in the backdrop (it used to be skipped behind mullions/transoms) shows the empty inside of the house through
  the glass, lit by lamps and hero point lights — it read as "something behind the glass". Arched heads need the
  reveal too.
- Roofs: hipped/gable with shingle rows, darker hips and ridge, cornice and gutter line; chimneys with a cap and pots.
- Lanterns by doors and on every wall people use, lamp posts on paths, uplights on facades: use `lantern()`,
  `lampPost()`, `uplight()` — each builds the fixture and registers its light at the fixture.
- **Openings must read dark.** Arcades, porches and deep doorways only read as openings when their interior is
  clearly darker than the facade. AO does most of it; also keep lamps out of the opening's line, and darken the
  porch back wall and ceiling (multiply by ~0.3) when light reaches inside.
- Combined footprints (L-shapes, bays) get one roof: the **upper envelope of per-rectangle hipped roofs**
  (`max` of each rectangle's height inside its own footprint) gives correct valleys; separate overlapping
  roofs poke through each other.

Alignment (check it logically before screenshots; the user asked for this pass explicitly)
- **Stack openings on shared centre lines.** An upper window over a door or window below shares its centre line
  unless the photo clearly breaks it; 0.5–0.7 units off reads as a mistake. Same for paired windows under a gable
  apex.
- **Fixtures sit relative to the opening they serve:** wall lamps symmetric about a door, on the outer corners
  of its surround (as photographed), never overlapping a slab or sill above; steps and mats as wide as the door.
- **Architectural edges line up with each other:** a recessed loggia or balcony that is "half the gable" ends
  exactly on the ridge line; a downpipe sits on a corner, not just past it.
- **Check every interior prop is inside the footprint:** canted bays have diagonal limits (`x - z >= …`), and a
  floor lamp at a plausible-looking coordinate stood outside the house in the garden.
- **Things on a roof plane mustn't overlap:** a skylight's flashing and a roof ladder's rail landed on the same
  cell. Keep ≥ 1 fine cell between roof furniture.
- **Keep added fixtures off the main sight lines:** a lamp post placed in the parking stood right in front of a
  door in the hero view. Put it at a corner of the hard surface.

Glass (the user must clearly *see* it)
- Material: `MeshPhysicalMaterial({ transparent, opacity ≈ 0.55, roughness ≈ 0.03, clearcoat: 1,
  envMap: RoomEnvironment PMREM, envMapIntensity ≈ 0.9, depthWrite: false })`, `renderOrder = 1`, no shadows.
- Tint a clear blue (`0x5f9cc4`); too pale/too transparent and it disappears into the wall colour.
- At night fade the env reflection (`envMapIntensity` × (1 − 0.85·night), done in `setSky`): the
  RoomEnvironment's bright light bars show as identical pale stripes in every pane and look like objects inside.
- Panes must be big enough to read (mullions every 4 fine cells, not 3) and have something behind them
  (room colour, plants, furniture) so the transparency shows.

Surroundings
- Recreate what is in the photo (trees by type, shrubs, topiary cones, flower beds, chairs, tents, cars, rocks).
- Imagine the unseen sides: back terrace with balustrade and furniture, formal garden/fountain, pergola,
  conservatory, side door and drive, woodland ring. Keep tall trees off the main sight lines.
- Small props at fine resolution: chairs (2×2 fine), tables, umbrellas, loungers, urns, benches, string lights.

## Imperfection: nothing is clean or perfect

The user's standing complaint: generated items look too "clean" and "perfect". Real places are worn, uneven and
a little messy. The template does part of it automatically; the scene must do the rest.

Automatic (template; strengths in `WEATHER` and `SCRUFF`, 0 = off):
- **Weathering** in the bake: broad colour blotches, odd off-shade voxels, grime splashed up the foot of every
  wall (~1.8 units), vertical rain streaks above that, moss on upward faces above ground (roofs, caps, sills).
- **Scruff** before connectivity: weeds and the odd flower at the foot of anything standing in the lawn; leaf
  litter on the ground under every `tree()` crown. Define `hardAt(fx, fz)` for paving/gravel/decks so no weeds
  grow there: weed cubes around car wheels, bins and steps on paving read as debris.
- **Lawn**: low hummocks (`heightAt`) and worn, sun-scorched patches (`P.lawnDry`).
- **Roofs** (`hip()`): replaced tiles in another shade, a few lifted/slipped tiles.
- **Trees** (`tree()`): leaning trunk with a kink, 3–5 uneven crown clumps with gaps, root flare, ~5% yellowed
  leaves. Never place two trees with the same height/radius.

By hand, in every scene:
- **Break every repeat.** No two windows identically dressed (curtains open/closed/half, a plant, a lamp, one
  dark), no two shrubs the same size, chairs not perfectly aligned — rotate one, push one back.
- **Wear where things are used:** worn path across the lawn to the door, dirt by the gate, scuffed step edges,
  darker patches under benches and the car.
- **Clutter of life:** a hose, watering can, bucket, bike against a wall, stacked logs, a ladder, toys, a
  wheelbarrow, bins by the side door, a doormat. Two or three per side is enough.
- **Material irregularity:** stones a voxel proud here and there, a cracked or missing paver, a darker repaired
  patch of wall, moss in the joints of a retaining wall, ivy climbing one corner, a sagging gutter end.
- **Organic edges:** hedges and beds with uneven tops and edges, plants spilling over path edges, gravel
  scattered onto the lawn.
- Keep it believable, not ruined: the subject still reads as cared-for; aim for "lived-in".

## Baked lighting (AO + lamps)

Real-time point lights cost per pixel, three.js slows down past ~15–20 of them, and they cast no contact
shadows. The template bakes both into the voxel colours once, after the scene is built:
- **Ambient occlusion:** from every exposed face, 9 short rays (normal ± tangents, reach `BAKE.reach` design
  units) through a `Uint8Array` occupancy grid; near hits darken by up to `BAKE.ao` (0.6). Glass does not
  occlude. Window reveals, eaves, porches, corners, foliage and wall bases get depth for free.
  Use 0.7–0.8 for overcast scenes, 0.4–0.5 at night where lamps dominate.
- **Only light sources emit light.** The user saw pools of light at "randomly selected spots": hand-placed
  `bakedLights` coordinates drifted from the fixtures (or had none — a facade uplight with no housing, a lamp
  post's light offset from its head). Now every lamp is registered by `lamp(src, fx, fy, fz, opts)` in **fine
  cells**, where `src` is a glowing voxel of its fixture; after connectivity, lamps whose `src` is not a glow
  voxel are dropped with a console warning. The fixture helpers call it for you:
  `lantern(axis, l, dir, a, y, opts)` (light inside its 2×2 glass), `uplight(axis, l, dir, a, opts)` (ground
  housing + lens 3 cells off the wall, spot up), `lampPost(x, z)`, and `win()` for lit rooms/doors (weak spot
  out through the glass). Write a custom fixture the same way: build the glow voxels, then `lamp()` at them.
  `window.__lamps` lists what made it into the bake.
- Lamp fields: `{ color, power, r (design units), dir?, cone?, shadow?, wrap?, hero? }`. Contribution =
  `power × (wrap + (1 − wrap)·facing) × (1 − d/r)²` (wrap 0.35 default) with a shadow ray to the lamp; glowing voxels don't block it, so a lamp sits
  inside its own glass. `dir` + `cone` (cosine of the half angle) make a spot. Uplights: `dir: [0,1,0],
  cone: 0.15`, power ~5, r ≈ 1.5× the wall height (28 for a 17-unit facade) so the wash reaches the eaves;
  lanterns power ~1.6, r 3.5 (a tight pool; at r 6 two door lanterns lit the whole 16-unit facade evenly and
  read as extra hidden sources); lamp posts 1.2, r 8–10; windows 1, r 9; rooms and tents 1–1.5. Dozens are fine;
  the bake logs its time and flags lamps that reach almost nothing.
- **Baked lamp light is emission, not albedo.** It used to be added to the instance colour, which the sun and
  hemisphere then multiply — so on a wall facing away from the sun, and at night, a lantern lit nothing (the
  user: "why not all lights produce light?"; only lanterns next to a real PointLight showed). Now the bake
  writes each voxel's lamp RGB into the `lamp` instance attribute and the lit material adds
  `diffuseColor × lamp × lampK` to its emission (`onBeforeCompile`), so it reads the same on every side, day
  or night. `BAKE.gain` (1) scales it; `setLights(k)` sets `lampK` so lamps fade in with the glow material.
  Baked light does not reach glass or glowing voxels.
- The shadow ray stops 2.5 cells short of the lamp: a lantern's own caps and bracket otherwise cast a dark
  stripe up and down the wall.
- **Avoid real `PointLight`s** (`hero: true`). They cast no shadows, so even at intensity 10 / distance 10 the
  two door lanterns' point lights washed the entire facade, past the windows to the corners — the user saw
  "other light sources". The template has none. Use one only for something baked light can't reach (glass,
  moving parts) and check with `__hero(false)` that it doesn't spread light where no fixture is.

## Lighting and camera

- `HemisphereLight` for sky fill + one shadow-casting `DirectionalLight` (4096 map, shadow camera covering the
  whole island, `bias -0.0006`, `normalBias 0.02`) + ACES tone mapping. Match the photo's time of day:
  overcast = strong hemisphere, weak sun; dusk = warm low sun from behind, a cool fill from the front, warm
  baked lamps; night = blue ambient + emissive glass + optional `UnrealBloomPass`.
- Give the hemisphere a mid earth-tone ground colour (e.g. `0x8a7c66`); a dark one turns the island
  underside into a black hole.
- Background: CSS gradient on `body` with an `alpha` renderer. If you add post-processing (bloom), the canvas
  loses transparency — set `scene.background` to a gradient `CanvasTexture` instead.
- Orthographic camera. Front-facing photo → start near the photo's angle; once the scene has imagined sides,
  orbit the full 360° slowly (`camAz += dt * 0.12`) and hold the finished scene ~30 s before looping.
- Soft voxel outlines come from a 32×32 canvas texture (white centre, light grey border) used as `map`.
- With bloom (`EffectComposer` + `UnrealBloomPass` + `OutputPass`), use threshold ≈ 0.78 so glowing voxels bloom
  and lit surfaces don't; ramp strength with the lamps. Skip `composer.render()` while `innerWidth` is 0 (hidden
  pane) or WebGL floods the console with zero-size framebuffer warnings.

## Fire, smoke and steam
- Voxels carry a `TAG` (0 prop, 1 smoke, 2 rocket/clean metal, 3 steel, 4 terrain, 5 flame) so passes can treat
  them differently: smoke and polished metal get no moss/grime (moss on clouds turned them green), steel gets rust
  where houses get moss.
- Smoke: noisy ellipsoid puffs (two `vnoise` octaves on the radius) that never overwrite structures, only terrain
  and smoke. Keep a 2-voxel skin and delete the interior (hundreds of thousands of voxels saved). A tall steam
  column is a stack of overlapping puffs drifting sideways.
- Flames are glow voxels placed **before** the smoke and **straddling the smoke surface** — buried flames are
  invisible. Make each jet one ragged tongue of densely overlapping puffs (step ≈ 1/3 of the gap), not a row of
  separate balls, and colour by heat: white-yellow near the engines, orange at the tail.
- Light the smoke and the vehicle from the fire with baked lamps with `shadow: false` (they sit inside the steam);
  a vertical row of weak lamps in front of a tall lit body (the booster) gets an even wash up its length.

## Verification (in the Claude browser pane)

- `file://` pages render as static snapshots in the pane — serve the folder instead:
  `.claude/launch.json` with `python -m http.server 8765`, then `preview_start` and navigate to
  `http://localhost:8765/<file>.html`. Add `?v=N` to bust the cache after edits.
- If another session already serves the folder, `preview_start` reassigns the port but `python -m http.server`
  still binds the hard-coded one, so navigation fails. Use a second config with its own port and
  `"autoPort": false` (the project has `static-2` on 8771).
- While the pane is hidden, animation frames are throttled and a screenshot right after load can show an empty
  sky. Check the `#bar i` width before judging the build. The same throttling leaves the lamps half faded in after
  `F`, so the scene looks dull — call `__bake(true)` before judging lighting.
- If the session borrowed another session's server and it goes away, `preview_start` `static-2` yourself.
- The first seconds after load are slow while 100k+ voxels are generated; wait ~5 s.
- Typical check: `dispatchEvent(new KeyboardEvent('keydown',{code:'KeyF'}))`, wait 3 s, untick Orbit,
  `__seeAz(0.3)` (front), `__seeAz(Math.PI + 0.3)` (back), `±Math.PI/2` (sides); screenshot, then
  `__zoom(3, x, y, z)` onto windows, arches, glass and props (the screenshot `zoom` action only crops pixels).
- Photo check: frame the photo's angle, screenshot, and compare with the photo for silhouette, proportions
  (scale table), the main material colours, and where the light falls. `__bake(false)` / `__bake(true)` shows
  what the lamps contribute.
- Read the console: no errors, a small connectivity count, and the bake time (`bake: N ms`; up to ~2 s is fine). Compare the voxel count with expectations
  (60k–260k is normal at `S = 2`). When the connectivity count is in the hundreds or more, read `window.__lost`
  (removed voxels bucketed by 4-unit design cells) to find what came loose. Typical culprits: an arm whose root only
  meets the carriage at a corner, a window room with no reveal joining it to the wall, a dish not joined to its
  mast, a hose one cell off the tank.
- Zoom onto every prop group from the side it is seen from, and look for occluders (a palm in front of the trucks)
  and for dark single-voxel specks on grass (tufts/weeds read as debris; keep tufts ≈ 1.5% and light-coloured).
- If the user dragged the pane, the camera stops auto-moving (`userMoved`); reload before judging framing.
