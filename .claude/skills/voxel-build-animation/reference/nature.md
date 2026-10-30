# Nature, surroundings and imperfection

## Surroundings

- Recreate what is in the photo (trees by type, shrubs, topiary cones, flower beds, chairs, tents, cars, rocks).
- Imagine the unseen sides: back terrace with balustrade and furniture, formal garden/fountain, pergola,
  conservatory, side door and drive, woodland ring. Keep tall trees off the main sight lines and out of the band
  just behind an imagined courtyard, or the back view shows only treetops: a woodland ring keeps spruces ≥ r + 5
  units off the walls, a ~16-unit clearing behind the back wall and the hero view's near side open.
- **Keep busy, same-coloured props out from behind the hero's silhouette** in the hero view: a prop of the
  subject's colour directly behind its head or edge swallows that edge. Check what lies behind the subject along
  the view, too.
- **Match the photo's vegetation density.** 17 separate trees in open grass read as a park next to a photo of
  continuous woodland; ~30 trees plus ~110 shrubs (40% clustered round the trees) read as woodland.
- Keep shrubs a radius clear of paths and props: a later blob, root flare or reeds overwrite them (console:
  `prop X: no voxels left`).

## Trees

- `tree()` (template): leaning trunk with a kink, 3–5 uneven crown clumps with gaps, root flare, ~5% yellowed
  leaves. Give each tree its own slight hue/lightness shift. Real heights: [scale-and-props.md](scale-and-props.md).
- Foliage colours are desaturated (s 0.2–0.35, l 0.25–0.4; see [colour-and-materials.md](colour-and-materials.md));
  the per-normal tint and AO give the crown its light and shade, so a saturated leaf colour reads as plastic.
- Canopies: run a branch from the trunk into each leaf blob, or the blob floats.
- **Holm oaks** need a low dome crown (`tree(…, { low: true })`: skirt down to ~⅓ of the
  height, 5–7 clumps, crown radius ≈ 0.4·h); the default gives a ball on a long bare trunk ("lollipop").
- **Spruce/fir** (`spruce()`): per-layer `disc()`s, radius `r·(1−t)` minus a 0.8-unit whorl
  sawtooth every 5 layers, trunk to 75% of the height. Step each layer in by under a cell (widest at the whorl's
  foot): whorls stepping in by 30% leave every tier floating after hollowing (40k voxels lost). Each whorl's foot
  grows out over the tier below by ≤ 0.8 fine cells per layer and steps in by ≤ 1.5 (`spruce()` clamps both, whorls
  every ~1 m): a wider step hollows away the joining cells and the crown comes loose. Trunk **after** leaves.
  Identical cones read as a plantation: each spruce draws its own taper exponent, whorl spacing/depth, jutting tiers,
  drift off plumb and tint from `hash` (not `rnd`, so the layout doesn't shift). Keep the skirt low (3–15% of h): at
  up to a third the spruces read as lollipops. A drifting trunk must stay face-joined where it steps sideways, or the
  whole crown comes loose (~7k voxels per tree).
- `blob()` edges wobble by smooth design-unit noise plus under a cell of speckle: per-cell noise of ±`rough` units
  punched holes deeper than the 2-cell skin at S = 3 and lost ~56k crown voxels.
- **Weeping willow:** low dome plus 4–5 clumps on top (one big blob reads as a flat-topped cylinder), and hanging
  shoots every ~4.5 cells round two rings with lengths spread by `hash²`; at every 2.2 cells the curtains form
  a solid green wall.

## Hollowing solid masses

Hollow every solid mass, not just leaves: bales, logs, crops, animals (`TAG ≥ 6`; the template deletes each
voxel whose neighbours one *and* two cells away on all six sides exist, keeping a face-connected 2-voxel skin).
Hollowing cuts a ~400k animal-and-meadow scene by 160k and keeps ~20 large trees near 250k at `S = 2`. The skin
stays connected only if neighbouring layers differ by ≤ ~2 cells. It leaves a few thousand interior pockets the
connectivity pass removes — ~2000 there is expected; check `__lost` is all inside masses.

## Meadows, flowers, crops

- **Wildflowers in drifts:** `vnoise` picks the species per area (15% strays); density `0.004 + 0.03·vnoise`,
  ~1.3% of lawn columns overall. At 2.4% plus tall-grass patches the lawn becomes a stubble field; even spacing
  reads as a pincushion of matchsticks.
- Stalk + bloom cell; daisy petals round a yellow eye at `S ≥ 3`; blooms 2–3 cells at `S = 3`; stalks short and
  close to the lawn colour. Grow flowers last and only where the column is clear to the bloom. Bloom colours are
  the palette's accents (`P.flower`, exempt from the lint) and the only saturated thing in a meadow.
- Keep a clear disc of short grass round a small feature (a fairy ring vanishes among same-red poppies).
- **Crops (wheat):** smooth noise alone gives flat-topped walls that read as hay bricks. Fill ~78% of columns (50%
  within 2 cells of the edge, 3 cells lower there), sink every 3rd row as drill rows, drop the top cell of ~40%
  of columns, and make the ears *darker* golden-bronze over paler straw — pale ears vanish.

## Imperfection: nothing is clean or perfect

Generated scenes tend to look too "clean" and "perfect". Aim for "lived-in", not ruined.

Automatic in the template (strengths in `WEATHER` and `SCRUFF`, 0 = off):
- **Weathering** in the bake: colour blotches with a slight hue drift each (`WEATHER.hue` 0.06, warm or cool),
  off-shade voxels, grime up the foot of walls (~1.8 units), rain streaks, moss on upward faces (`moss` 0.2; above
  0.35 every roof and wall top goes green). Moss skips props (`STAGE 4` except leaves — it tints iron lantern caps
  olive), smoke, clean metal and animals; steel gets rust instead.
- **Scruff** before connectivity: weeds and the odd flower at the foot of anything standing in the lawn; leaf
  litter under every `tree()` crown. Not on `hardAt()` cells: weed cubes around wheels, bins and steps on paving
  read as debris. Scruff cells and grass tufts are `TAG 9`: the bake gives them blotch and hue drift only — wall-foot
  grime (which starts at h = 0.5, exactly one cell up) and side-face AO turn a free-standing cell near-black.

By hand in every scene:
- **Break every repeat:** windows dressed differently, shrubs of different sizes, chairs not aligned — rotate
  one, push one back.
- **Wear where things are used:** a worn path to the door, dirt by the gate, scuffed step edges, darker patches
  under benches and the car.
- **Clutter of life,** two or three per side: hose, watering can, bucket, bike, stacked logs, ladder, toys,
  wheelbarrow, bins by the side door, a doormat.
- **Organic edges:** uneven hedge tops, plants spilling over path edges, gravel scattered onto the lawn.
