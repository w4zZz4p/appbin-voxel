# Scale and props

Proportions are the first thing that makes a scene look like a toy. Check every element against these numbers.

## Scale table (houses: 1 design unit = 40 cm; at `S = 3` a fine voxel is ~13 cm)

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
| Garden tree | trunk 2 fine, height 9–12, crown radius 3.5–5 | woodland 11–20; next to a big subject use real heights (below) |
| Person (if any) | 4.5 high | |

## Choosing the scale

- **Big subjects** (rocket, tower, ship, factory): pick metres per design unit from the photo (measure two known
  sizes in pixels), set `M_PER_UNIT` and write it at the top of the scene. Large structures (towers, rockets, industry) work at ≈ 1–1.2 m, a castle complex at ≈ 0.8 m, a whole city (100–230 m towers) at 2 m (1 m cells at `S = 2`, ~310k voxels
  with towers as `lathe()`/box skins); props then lean on the one-cell tolerance (a car is 2 × 2 × 5 cells).
- **Small or organic subjects** (an animal): 0.2 m units at `S = 3` (≈ 7 cm cells); at 0.4 m a horse is ~12
  cells long and eyes, ears and a horn can't resolve.
- **Props too coarse at real size?** Make the grid finer (`S = 3`) instead of enlarging them. ~620k voxels at
  `S = 3` run ≈ 47 fps with a 0.8 s bake.
- **Measure openings from the photo** with a scale bar on the same wall (eave height or a door), including sill
  and head heights, not just widths — eyeballed windows land 0.6–1 unit too low and attic windows 30% too wide.
  Perspective enlarges the near side; compare with a reference at the same depth.

## Every prop is 1.0–1.2× its real size

Vehicles at 1.5–1.7× read as toys and a car at 0.7× as a model; the island may grow to make room instead.
Small fixtures get one cell (a 2×2 fine block is already ~27 cm at `S = 3`).

Real sizes (w × h × l, metres): pickup 2.0 × 1.9 × 5.9 · hatchback 1.8 × 1.5 × 4.1 · semi tanker 2.5 wide, 18
long · 20 ft container 2.44 × 2.59 × 6.06 · crawler-crane track ≈ 9 long · door 2.1 high · jersey barrier
0.6 × 0.8 × 3.7 · traffic cone 0.7 · bollard 1 · wheelie bin 0.6 × 1.05 × 0.75 · wheelbarrow 1.4 long · bench
seat 0.45 / back 0.85 · table 0.75 · wheel ≈ 0.65 · globe wall lamp 0.25 · slim metal flue 0.35.

Lanterns: at `S = 3` use `wallLamp()` (one cell), not the 2×2 `lantern()` (0.74 m of glass). `cells(m)` converts
metres to fine cells for real-size parts.

## Sprites (vehicles, machinery, furniture)

- Author props with the template's `sprite(ox, oy, oz, rot)`: local cells of `M_PER_UNIT / SPRITE_S` metres (0.13 m
  at the defaults), `s(x,y,z,c,e)` / `s.box(…)`, 90° rotations, `s.w()` → design coordinates for `limb()`, `s.f()` →
  fine cells for `lamp()`. `q(n) = round(n·S/SPRITE_S)` rescales them to `S`. Size them from that cell size —
  `__props` flags it if you don't.
- **`q(0)` must be 0** — never `Math.max(1, …)`: rows 0 and 1 then both map to cell 1, every sprite floats a
  cell up (the connectivity pass deletes it) and loses a row of width.
- Rotations 2 and 3 extend a sprite toward −x / −z from its origin: place it by the far corner, or a parked car lands
  in the next bay, overlaps its neighbour and inspects as one 3.5 m prop. Parked cars get dark lenses and no
  lamp; lit headlights on a parked car read wrong.
- Detail that reads at real size: separate wheels with hubs and arches, glass with seats behind it, pillars,
  mirrors, grille + glowing head/tail lights, bumpers, cargo; container corrugation, corner castings, door seam,
  lock bars, rust, stencil; crane lattice boom with zigzag lacing, pendants, hoist rope, striped hook block, a
  hung load, hazard-striped counterweight, glazed cab, tracks with rollers on mats.
- **Cars match the body type's side profile**, not just the size. A hatchback has a short hood (~0.8 m), a raked
  screen, and a roof running to a near-vertical tailgate with a spoiler; a separate low deck behind the cabin
  reads as a sedan. Keep 0.8 m overhangs and a ~2.5 m wheelbase.
- Vehicle paint is the one place a saturated colour is allowed (a red car is red), but keep it to the body panels,
  add 2–3 shades plus dust at the sills, and keep the photo's lightness; a single bright hex reads as a toy car.
- Lattice structures: `lattice()` (corner columns, ring beams, X braces) and `trussX()` (chords, verticals,
  diagonals). Keep a crane boom ~1/3 of the crane body's width or it reads as a solid plank.

## Boats

- Hull rows from a half-beam profile (full amidships, fine bow, narrower transom). The bottom row *replaces* the
  water-surface cell so the boat sits at the waterline — so colours on row 0 never show: paint the boot stripe
  on row 1.
- Sloop: red antifouling, white topsides, teak deck, cabin trunk with ports, mast + boom with sail cover,
  stays as face-connected lines (`fline()`). Tour launch: navy bottom, saloon with pillared windows (some glowing
  + one shadowless lamp), upper-deck rail and benches, wheelhouse, flag.
- Leave one cell of water between hull and pier.

## Trees at real size

Next to a big subject the table's 9–12-unit garden trees (4–5 m) read as shrubs. Holm oak 7–9 m (crown Ø 4–6 m),
Aleppo/stone pine 10–12 m with a flat, broken crown (`sy ≈ 0.5`, wide `spread`, more lean), cypress 8 m as a
narrow column, spruce 17–26 m on a far shore. Trees below a cliff stay under the plateau top so they don't hide
the building in the hero view. Never two trees with the same height/radius. Shapes: see [nature.md](nature.md).

## Animals and other organic subjects

- Don't sprite them. List parts in the animal's own frame (x forward, y up from the hoof sole, z to its near
  side): ellipsoids for barrel, chest, quarters, withers; tapered capsules (`zs` flattens across) for neck, head,
  legs; a cylinder per hoof. For every fine cell in a bounding box, transform the cell centre into that frame
  and test the parts, later parts winning (eyes, nostrils, horn last). Rotation is then free.
- Hair: cubic-Bézier strands of overlapping balls drawn *before* the body so it covers their roots; lighten each
  lock toward the tip. Tail locks stacked in z show only the outermost colour from the side — stagger them in x
  and interleave colours.
- Horse proportions: withers 1.6 m, belly 0.73 m, body ≈ withers height long, head 0.6 m, legs ±0.8 m apart.
  One foreleg raised (forearm forward, cannon hanging back from the knee) reads as prancing; the raised hoof must
  be ≥ 0.4 m up — at 18 cm it reads as a short leg standing on a flower.
- Small parts need size to read: ears r ≥ 0.38 units for the pink inside to show; a horn r 0.55 → 0.12 units
  with glowing *rings* (`t·5`) in deep gold (`0xd49a2c` / `0xffe07a`) — 1–2 cells wide shows no spiral and pale
  gold reads as coat. Put a horn lamp at mid-horn in front of the face, not above the tip.
- Tag it `TAG = 7`: hollowed to a 2-cell skin, and weathering skips it (no grime or moss on a white coat).
