# Terrain: the floating island

The floating island is the default ground; the fade-out ground (below) is the alternative for wide, flat subjects.

## Outline

- Irregular, never a circle or oval. Rounded-square base (superellipse norm `(|dx|^4+|dz|^4)^¼`), then
  `edgeR()` adds **lobes** (harmonics 2, 3, 5 with random phase/strength → bays and peninsulas), a **ragged
  coastline** (2D value noise at scales 9 and 3.5) and soft rim bumps. Change `ISLAND` (the outline seed) for a
  different shape; keep the outline's lowest point clear of the building and props.
- Rim bumps are a rounded bell `sin²(π·f)` per segment — **never pointy** (sharp `tri^3.5` thorns read as
  spines) — and **few, mostly low**: `K ≈ 14` around the rim, amplitude `0.6 + hash² × 5`. 30–40 spikes look
  like a sea urchin.
- **Generously wider than the building:** radius ≈ building half-width + 15–20 units; a tight island reads as a
  pedestal.
- **Props near the rim are placed by test, not by eye:** check the footprint + eaves against `edgeR() −
  superellipse distance > 3` and take the nearest spot that fits; one placed by eye hangs over the edge, seen from below.

## Underside

A **shallow earth slab**, not a hanging rock mass: 2.5–4 design units thick, cut vertically at the rim, its bottom
undulating in flat steps (`floor(noise*4)/4`). The rim face shows turf (`2*S` layers), then topsoil to 2 units,
then `P.under` rock. A deep tapering cone draws the eye down from the subject and reads as a game asset; a slab
that thickens inward hangs an inverted stair under the rim whose risers sit in shadow and read as a black band.

## Surface

- Lawn / moss / gravel / mulch by region functions; low hummocks in `heightAt`, worn patches (`P.lawnDry`).
- Lawn colours are olive, not green: sampled grass sits at s 0.25–0.38, l 0.3–0.42 (see
  [colour-and-materials.md](colour-and-materials.md)); the bake's hue drift and moss supply the variation.
- Avoid dark patches (dark heather, exposed brown soil on slope steps) — they read as **holes**. On slopes the
  top `2*S` voxels of every column use the surface colour so steps look grassy.
- Grass tufts ≈ 1.5% of columns from a paler palette (`P.tuft`), tagged `TAG 9` so the bake skips grime, streaks and
  side-face AO on them; a tuft in lawn colour with normal weathering comes out near-black, and dark single-voxel
  specks read as debris.
- Define `hardAt(fx, fz)` for paving, gravel, decks and water so scruff grows no weeds or leaf litter there.

## City slab (streets instead of lawn)

- A `groundKind(fx, fz)` zone function (road bands along x or z, plaza, parking lot, planters) drives the colour and
  `hardAt` (everything but planters). Roads one fine cell below the pavement (`top = TOP - 1 - (road ? 1 : 0)`) give
  kerbs for free; a lighter kerb cell where pavement meets road.
- Markings: dashed centre line (every 5 cells) except inside junctions, zebra stripes in the 3 units before a crossing
  band, bay lines every 5 cells in the lot. Wet night asphalt: 20% lighter blue-grey speckle plus `vnoise > 0.74`
  puddles; a plain 0x1c2128 road read as black. Rim: the outermost 1.3 units in kerb colour, 1 m bollards along it.
- A road band runs the whole slab: anything "beyond" the cross street (a bus shelter, a pedestrian) placed by eye lands
  on the avenue's continuation, half a cell above the road. Stand props on `topAt() + 1`, never on `BY`, and check
  `groundKind` at their cells.

## Fade-out ground (optional, instead of the island)

For a street, a campus or a skyline the island's rim competes with the subject. A fade-out ground keeps the
terrain loop and everything standing on `topAt()` but dissolves its edge into the haze:

- `edgeR()` returns a large constant (≈ 2.5× the subject's half-width) with the ragged noise but no lobes or rim
  bumps; `heightAt` keeps gentle hummocks only.
- No underside: `bottom = TOP - 1 - S` for every column (two layers, so the hidden-voxel culling still finds a
  shell), and the `P.under` shades are unused.
- `surfaceC` blends toward the haze colour with distance from the centre (`lerp(lawn, SKY.day[2], smoothstep(0.55,
  1, d / R))`) and `HAZE.strength` goes to 0.45 with `far` ≈ R, so the last rows of ground and the sky meet without
  a visible edge. Keep the camera elevation ≤ 20° (`ORBIT_EL`, `?cam=persp`) so the edge stays near the horizon.
- The connectivity seed, `hardAt`, scruff and the prop report are unchanged. Effects that key on the island
  centre (`tornado`, `slide`, `pile`) still work; `demolish` throws pieces farther on a flat ground, so lower
  `DEMO.span`.

## Hidden-voxel culling (terrain columns only)

Skip a voxel when every neighbour column covers it, with *strict* comparisons so one layer of overlap remains:
hide if `nbot < fy && ntop > fy` for all 4 neighbours. With `<=`/`>=` the shell joins only diagonally and the
connectivity pass deletes it.

## Cliff-top subjects (castles, villas on a spur)

- `heightAt` = max(a slope/hill, a plateau). The plateau is a rectangle around the building (+ a few units of
  courtyard) with a **ragged, stepped edge** (noise on the distance, `floor(…/1.5)*1.5` ledges): steep on the
  photo's cliff side, gentle at the back so a path and a few steps reach a gate.
- Store a per-column kind (slope / cliff / plateau). Colour cliff sides as limestone **strata** (a shade per
  2-cell band, wobbled by noise, the odd dark crack); exposed sides above the island base are rock, below it the
  darker underside.
- Keep scruff weeds **off cliff columns**: every ledge counts as a "foot of a wall" and gets lined with confetti.
- Stand everything on the plateau at `BY = GY + ROCK`, and trees/shrubs/lamp posts on `topAt(…) + 1`. The
  template's plants and fixtures (`tree`, `spruce`, `cypress`, `shrub`, `lampPost`, `uplightAt`) stand on `topAt()`;
  custom props must too.

## Island in a lake

- Each column is one of three kinds: subject island (a shape function `e(x, z)` > 0: superellipse + noise + lobes), far-shore
  ring (`e < shoreW`, measured in from the rim, narrower in front so it doesn't hide the subject), or lake
  (h = 0, one surface cell; the column below is culled).
- Water top: navy with a shallow band near land, sparse ripple streaks, **very few glints (~0.1%)** — at 0.5%
  they read as a starry sky. `hardAt()` returns "water", and the leaf-litter pass must skip hard cells too or
  leaves float on the lake.
- A moat is a band where that shape function is forced negative; give its banks fieldstone sides.
- Piers and bridges: deck over water on piles, a footing over a bank (`deck(x0, z0, x1, z1, y, { water, footing, rail })`); they stop where the
  ground reaches deck height.

## Diagonal structures (a bridge at an angle)

Rasterise in (u, v) along/across the axis. Keep a railing on a single cell line (`|v|·S > HW − 0.15`); a
1.45-cell band doubles it into a lumpy ridge. Cross-hatch panels between posts with two colours on two rows (a V
stair-stepped on the diagonal reads as noise), and keep the deck colour distinct from the rail.

## Coordinates

Write building coordinates relative to an origin (`X(u) = OX + u`, `Z(u) = OZ + u`) so the complex can be
shifted onto the island centre — and route *every* helper through them: a helper that passes relative values
straight to `box()` builds a 30 m wall floating off the island.
