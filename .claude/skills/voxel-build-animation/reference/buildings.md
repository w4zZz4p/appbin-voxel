# Buildings

## Walls

- One-voxel `shell()`s with a material function: fieldstone (offset courses + mortar), boards (per-board hash
  colour), concrete (vertical streak noise). Add a plinth course and a string course between floors.
- Material irregularity: stones a voxel proud here and there, a darker repaired patch, moss in retaining-wall
  joints, ivy on one corner, a sagging gutter end.
- Wall colours come from the sampler and stay muted (concrete/render s ≤ 0.1, brick s ≤ 0.4; see
  [colour-and-materials.md](colour-and-materials.md)); the bake's grime, streaks and hue drift add the variation.

## Towers and skyscrapers

- Facades at 1 m cells: storey 4 cells (1 spandrel + 3 glazing), mullions every 1.5 units, lit windows as glow
  voxels. Light offices in runs of 2–5 bays with a share per floor (some floors dark); per-bay random lit read as a
  checkerboard. A glazed lobby (glass skin, lit hall ring 2 cells in, a ceiling) gives each foot a real window.
- At 0.5 m cells (`M_PER_UNIT 1`, `S 2`): a `block()` in fine cells (shell, spandrel row every 7, parapet, roof slab,
  `roofStuff`), panes as glass at the wall plane with a closed backdrop one cell inside (2 voxels per pane; lit = glow
  backdrop, ~30% blinds or dim) — a 9-building block came to ~195k voxels. A sloped top: per-column `topX(fx)`, walls
  filled to it, roof 2 rows thick, parapet at top+1; masts stand on roof cells, never on the parapet line.
- A lattice (`lattice()`) is hollow: put a solid platform on its top ring before standing a mast on it, or the mast
  floats off. A lamp whose source cell depends on a pattern (sign letters, a helipad ring) takes the nearest glowing
  voxel (`findGlow(fx, fy, fz, R)`) rather than a guessed coordinate.
- Blade signs on facing walls of a narrow alley merge into one column from the street: stagger their heights per wall
  (one low, one high) and keep a one-cell gap between the two walls' blades. Hang cables and lanterns below the lowest
  blade (4–5 m) or they vanish behind it. Anything under an awning with a valance is invisible: hang lanterns below it.
- `prop()` doesn't nest: to name a sign on a tower, save the outer `PROP`, `prop(sub)`, then restore it (`subProp` /
  `endSub`), or the tower's later voxels go unnamed.

## Windows and doors (`win()`)

- Frame + mullions on the wall plane, **glass recessed one voxel**, the frame continuing inward as a reveal, the
  **room two voxels behind the glass** (warm glowing when lit, with curtains; dim otherwise), stone sill, arch
  band, keystone; round-arched heads where the photo has them. Doors the same with warm glass.
- **Close the room backdrop behind every pane, mullions included**, and give arched heads a reveal. Walls are
  hollow shells; a gap shows the empty, lamp-lit inside of the house and reads as "something behind the glass".
- No two windows dressed the same: curtains open/closed/half, a plant, a lamp, one dark.
- **Openings must read dark.** Arcades, porches and deep doorways only read as openings when clearly darker than
  the facade. AO does most of it; also keep lamps out of the opening's line and darken porch back walls and
  ceilings (× ~0.3) where light reaches inside.

## Glass (must be clearly visible)

- `MeshPhysicalMaterial({ transparent, opacity ≈ 0.55, roughness ≈ 0.03, clearcoat: 1, envMap: RoomEnvironment
  PMREM, envMapIntensity ≈ 0.9, depthWrite: false })`, `renderOrder = 1`, no shadows.
- Tint a muted blue (`0x6b96b4`); too pale or too transparent and it vanishes into the wall colour, too saturated
  and it reads as a toy's cyan.
- Panes big enough to read (mullions every 4 fine cells, not 3) with something behind them (room colour,
  plants, furniture) so the transparency shows.
- At night `setSky` fades the env reflection (× (1 − 0.85·night)); otherwise the RoomEnvironment's light bars
  show as identical pale stripes in every pane.

## Roofs

- Hipped/gable with shingle rows, darker hips and ridge, cornice and gutter line; chimneys with cap and pots.
  `hip()` adds replaced tiles in another shade and a few lifted ones.
- **Combined footprints** (L-shapes, bays) get one roof: the upper envelope (`max`) of per-rectangle hipped
  roofs gives correct valleys; separate overlapping roofs poke through each other (`roofU()`).
- **Sloped surfaces are two voxels thick** so adjacent steps share a face (`ceil(slope) + 1` when steeper than
  one cell per cell).
- **Steep roofs lift off their walls:** with a 1-unit overhang at slope ≥ 1 the roof is 2+ cells above the eave
  where it crosses the wall, and the connectivity pass deletes the whole roof (~13k voxels). Fill each column
  down to the wall top over the eave band (dark fascia, `roofU`) and make a cone solid down to its eave over the
  tower ring (`coneRoof`).
- Things on a roof plane mustn't overlap: keep ≥ 1 fine cell between roof furniture (a skylight's flashing and a
  ladder rail on one cell overwrite each other).

## Round towers (`roundTower`, `rWin`, `rSlit`, `coneRoof` in the template)

- `T = roundTower(cx, cz, R, y0, top, { RW, wall(arc, fy) })` rasterises the ring on the fine grid and clears the
  inside; `T` carries `dist/ang/arc/outer` per fine column for the other helpers.
- Openings on the curved wall (`rWin(T, th, y0, y1, w, { lit, door, frame })`, `rSlit`): the arc offset `angle·d·S` stands in for the plane coordinate; outer cells are
  removed, the next layer is glass, a closed room backdrop sits just inside, the surrounding cells take the frame
  colour.
- Checker bands and crenellations go by arc position (`round((θ+π)·R·S)`; merlon = even slot of ~28 around a
  2.2 m radius).
- Build the tower **after** any roof that runs into it (it clears roof cells inside its ring).
- Gothic pointed arches: `gothicDoor(axis, l, dir, a0, a1, y0, y1, { ring })` on a wall, `archCut(axis, l0, l1, …)`
  for a gate passage through a thick wall.

## Railings

At `S = 2` a bottom rail plus balusters every 2 cells reads as a row of solid black frames. Use a handrail on
balusters every 3 cells, no bottom rail; between short stucco posts even that is heavy — keep the iron dark
grey, not black.

## Fixtures

- Lanterns by doors and on every wall people use, lamp posts on paths, uplights on facades — through
  `lantern()`, `lampPost()`, `uplight()` so the light is registered at the fixture (see
  [lighting.md](lighting.md)).
- Lanterns, sconces, signs need a bracket voxel touching the wall.

## Alignment pass (check logically before screenshots)

- **Stack openings on shared centre lines.** An upper window over a door or window shares its centre line unless
  the photo clearly breaks it; 0.5–0.7 units off reads as a mistake. Same for paired windows under a gable apex.
- **Fixtures sit relative to the opening they serve:** wall lamps symmetric about a door, on the outer corners of
  its surround, never overlapping a slab or sill above; steps and mats as wide as the door.
- **Architectural edges line up:** a loggia that is "half the gable" ends exactly on the ridge line; a downpipe
  sits on a corner, not just past it.
- **Interior props inside the footprint:** canted bays have diagonal limits (`x - z >= …`); test an interior
  prop's coordinate against the footprint, or a plausible-looking floor lamp stands in the garden.
- **Added fixtures stay off sight lines:** a lamp post goes at a corner of the hard surface, never between the
  hero camera and a door.
