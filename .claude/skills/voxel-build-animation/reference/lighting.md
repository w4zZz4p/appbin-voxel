# Lighting, camera and effects

## Light defaults (template `LIGHT`, `HAZE`, exposure)

- `LIGHT = { hemi: 1.0, sun: 1.2, sky: 0xcfdcea, ground: 0x7a7262, sunColor: 0xffe3c4 }`,
  `toneMappingExposure = 0.95`: a cool sky fill a little weaker than a warm sun. Daylight stays within
  hemisphere 0.8–1.2 and sun 0.8–1.5; a sun above ~1.5× the hemisphere blows out tops and blackens sides, which
  reads as a render, not a photo. Overcast photos: hemi 1.1, sun 0.5. Dusk: sun 0.6 low and warm from behind, hemi
  0.9 cool.
- **Per-normal tint** (lit shader): top faces take ~22% of the hemisphere sky colour, sides are ×0.93–0.985 and
  slightly cool, undersides ×0.82–0.9. It gives every cube three readable faces without raising contrast; keep
  `LIGHT.sky` the sampled sky colour so the tint matches the background.
- **Depth fade** `HAZE = { strength: 0.32, near: -12, far: 55 }`: the final colour blends toward the sky's
  horizon band (`SKY.day[2]`, `SKY.night[2]` at night) by depth along the camera's view axis, from `near` world
  units in front of the target depth to full `strength` at `far` behind it. Works for both cameras (depth is
  `vViewPosition.z`). Big scenes (`M_PER_UNIT ≥ 1`) use `far` ≈ the island radius; strength above 0.45 turns the
  back of the island into fog.
- `setSky` scales `LIGHT.hemi` / `LIGHT.sun` for night; a scene that changes the lights must keep that scaling.

## Baked lighting (AO + lamps)

Real-time point lights cost per pixel, slow down past ~15–20 and cast no contact shadows, so the template bakes
light into the voxels in a single pass after connectivity.

- **Ambient occlusion:** from every exposed face, 9 short rays (normal ± tangents, reach `BAKE.reach` design
  units) through a `Uint8Array` occupancy grid; near hits darken by up to `BAKE.ao` (0.6). Glass doesn't occlude.
  Use 0.7–0.8 for overcast scenes, 0.4–0.5 at night where lamps dominate.
- **Only light sources emit light.** Every lamp is registered by `lamp(src, fx, fy, fz, opts)` in **fine cells**,
  where `src` is a glowing voxel of its fixture; a light at a bare coordinate drifts from its fixture and gives a
  pool of light at a random spot. Lamps whose `src` isn't glow after connectivity are dropped with a warning naming
  the prop that now owns the cell (e.g. a tree crown over a lit window, or `A0 + 3` outside a 3-cell window — use
  the pane centre). The fixture helpers call it for you: `lantern(axis, l, dir, a, y, opts)`,
  `uplight(axis, l, dir, a, opts)` (by a wall), `uplightAt(fx, fz, opts)` (free-standing), `wallLamp(axis, l, dir,
  a, y, opts)` (one cell), `lampPost(x, z, opts)`, `win()` / `rWin()` for lit rooms and doors. For a custom fixture,
  build the glow voxels, then `lamp()` at them. `window.__lamps` lists what was baked.
- Lamp fields: `{ color, power, r (design units), dir?, cone?, shadow?, wrap?, hero? }`. Contribution =
  `power × (wrap + (1 − wrap)·facing) × (1 − d/r)²` (wrap 0.35) with a shadow ray to the lamp; glow voxels don't
  block it, and the ray stops 2.5 cells short so a lantern's own caps don't cast a stripe. `dir` + `cone`
  (cosine of the half angle) make a spot.
- **Baked light is emission, not albedo**: it goes into the `lamp` instance attribute and the lit material adds
  `diffuseColor × lamp × lampK` (`onBeforeCompile`), so it reads the same on every side, day or night (as albedo
  it vanishes on walls facing away from the sun). `BAKE.gain` scales it; `setLights(k)` fades it in with the glow
  material. It doesn't reach glass or glow voxels.

| Fixture | power | r | notes |
|---|---|---|---|
| Lantern | 1.6 | 3.5 | a tight pool; at r 6 two door lanterns light the whole facade evenly |
| Lamp post | 1.2 | 8–10 | |
| Lit window | 1 | 9 | weak spot out through the glass |
| Rooms, tents | 1–1.5 | | |
| Uplight, dusk/night | ~5 | ≈ 1.5× wall height | `dir: [0,1,0], cone: 0.15` so the wash reaches the eaves |
| Uplight, day | ~2.4 | ≈ wall height | `cone: 0.72` — the night setting floods sunny walls orange (it's emission) |
| LED panel, billboard | 1.2–1.6 | ≈ 1.1 × width | no `dir`/`cone`, `wrap` 0.35: the halo lands on the sign's own wall. A cone facing out from a wall high on a tower lights nothing (`lamps reaching almost nothing`) |
| Blade sign (perpendicular to the wall) | 1.6 | 11 | lamp in the one-cell gap between wall and blade, not inside the blade: at r 7 inside it no halo reached the wall |
| Fixture lens colour | | | glow cells take the lamp's colour (`o.color ?? GLOW`): amber lenses casting cyan (uplights, park posts) read wrong |
| Street lamp (7 m, head under the arm) | 2.2 | 13 | `dir: [0,-1,0], cone: 0.15`: a pool on the road; 1.5 / 11 left the street black |

Dozens of lamps are fine; the bake logs its time and flags lamps that reach almost nothing.

**Avoid real `PointLight`s** (`hero: true`): with no shadows, even at intensity 10 / distance 10 two door lanterns
wash the entire facade and read as light from nowhere. Use one only for what baked light can't reach (glass,
moving parts) and check with `__hero(false)` that it spreads no light where there's no fixture.

## Sun shadows (baked per face; the shadow map only while voxels move)

- **Settled voxels carry baked sun shadows:** from every exposed face that faces the sun, `bake()` walks 3 slightly
  jittered rays toward the sun cell by cell (3D DDA, reach `BAKE.sunReach` = 30 design units; glass and glow don't
  occlude) and stores the shadowed fraction per axis in the `sunSh` instance attribute (the face's own value).
- Rays walk the grid cell by cell, never by fixed steps: a one-cell tuft that a stepped ray only clips is missed and
  its shadow comes out faint, while the exact walk gives a full shadow on the neighbour at its foot.
- The shader scales the sun's **direct** term by `mix(map sample, 1 − baked, sunShK)`: the baked shadow dims with
  `sun.intensity` at night like the map, leaves sky fill and lamp emission alone, and a shadowed spot stays
  shadowed through the cross-fade. `BAKE.sunShadow` (1) is its strength; below 1 a little sun leaks in as fake bounce.
- `setLights(k, settled, dt)` fades the bake in over ~1 s once every voxel has landed, then sets `sun.castShadow =
  false` (no shadow pass per frame, no aliasing); any motion (replay, demolish, pile) brings the map back within a
  frame so flying voxels keep casting. `__shadow('map' | 'bake')` forces a source, `__shadow()` reports and resets.
- **The shadow map is fitted, not fixed:** `fitShadowCamera()` (called by `bake()`) projects the voxel bounds (+8
  design units up, for dropping voxels) into light space and wraps them with a 2-unit margin. `bias -0.00005` of the
  fitted depth range plus `normalBias 0.03` keep the offset under 0.02 world units, so contact shadows meet their
  casters; `-0.0006` over a 300-unit range detaches every shadow by a third of a voxel. `?shadow=hi` uses an 8192 map.
- The map's edges on a wall the sun grazes alias into a sawtooth with a period of about one voxel, and a texel
  covers ~0.02 world units even when fitted; the bake follows the grid exactly, which is why the map is for
  motion only.

## Sun, sky and camera

- `HemisphereLight` for sky fill + one `DirectionalLight` sun (shadows: see **Sun shadows** above) + ACES tone
  mapping. Match the photo: overcast = strong hemisphere, weak sun; dusk = warm low sun from behind, cool front
  fill, warm lamps; night = blue ambient + emissive glass + optional bloom.
- Hemisphere ground colour a mid earth tone (`0x7a7262`); a dark one turns the island underside into a black hole.
- **Sky texture is the default:** `useSkyTexture(SKY.day, SKY.night, SKY.at)` draws the sampled gradient as
  `scene.background` with a lighter band at the horizon (`at: [0, 0.5, 0.72, 1]`, band = index 2) and lerps it to
  night. Fill `SKY.day` from the sampler's `sky` strip, lightening the horizon stop if the strip is even. `?bg=css`
  falls back to the body gradient (keep it matching).
- **Cameras.** Default orthographic (`VIEW` half-height 40). `?cam=persp` is the hero view: `PerspectiveCamera`,
  `FOV` 28°, elevation `ORBIT_EL` 0.36 rad (~21°, keep 15–25°), distance `VIEW / tan(FOV/2)` so it frames the
  same height at the target. Use it for the hero screenshot and the photo comparison; the ortho view stays for
  side checks and inspection (`__props` centres are world points and work in both).
- Start near the photo's angle; with imagined sides, orbit 360° slowly (`camAz += dt * 0.12`) and hold the
  finished scene ~30 s before looping.
- **Voxel outlines:** a 32×32 canvas texture with a near-invisible border (`#f4f4f4`) as `map`, so cubes read as
  surfaces; `?grid=1` draws the full grid (`#dadada`) for checking geometry.

## Bloom (night scenes, fire)

- `EffectComposer` + `UnrealBloomPass` + `OutputPass`, threshold ≈ 0.78 so glow voxels bloom and lit surfaces
  don't; ramp strength with the lamps.
- The sky texture is already the background, so bloom's opaque canvas changes nothing; with `?bg=css` the
  `#night` overlay can't show through an opaque canvas — keep the texture.
- Skip `composer.render()` while `innerWidth` is 0 (hidden pane), or WebGL floods the console with zero-size
  framebuffer warnings.
- With the composer the lit material writes linear colour and `OutputPass` converts to sRGB, so the depth-haze
  colours must be linear (`raw = h => new THREE.Color(h)`), not raw sRGB components, or the haze band shifts lighter.
- Keep the glow gain at 1.0 and the threshold ≈ 0.6: at gain 1.25 cyan neon bloomed to white; pale tints (0xbafdfc,
  0xfbdaeb) on a panel bloom white too — mix a saturated pair (0x62d5dc + 0x14f1f4, 0xe032ad + 0xf07ad0) instead. Night city that matched
  the photo's `__compare`: `LIGHT { hemi 1.5, sun 0.55, sky 0x5a7a8e, ground 0x3a4048, sunColor 0x9fb8d8 }`, bloom
  (0.6, 0.4, 0.6) ramped with the lamps, `HAZE.strength 0.48`, sky stops lifted to the photo's upper-third means.
- Sampled night materials fall under the palette lint (l < 0.12): lift base walls to l 0.12–0.16 (navy 0x172330,
  steel 0x1c2b38) and put neon / lit-room keys in `ACCENT`; the darkness comes from the low hemisphere, not the albedo.

## Fire, smoke and steam

- Voxel `TAG`s: 0 prop, 1 smoke, 2 rocket/clean metal, 3 steel, 4 terrain, 5 flame, 6 leaves, 7 animal, 8 solid
  prop mass, 9 scruff grass (tufts, weeds: no grime, streaks or side AO). The template hollows every TAG ≥ 6; smoke and clean metal get no moss/grime (moss turns clouds
  green), steel gets rust.
- Smoke: noisy ellipsoid puffs (two `vnoise` octaves on the radius) that overwrite only terrain and smoke. Keep a
  2-voxel skin and delete the interior. A tall steam column is a stack of overlapping puffs drifting sideways.
- Flames are glow voxels placed **before** the smoke and **straddling its surface** — buried flames are
  invisible. One ragged tongue per jet of densely overlapping puffs (step ≈ 1/3 of the gap), white-yellow near
  the engines, orange at the tail.
- Light smoke and vehicle with shadowless baked lamps (they sit inside the steam), each tied to the nearest flame
  voxel as `src` — `fireLamp(x, y, z, opts)` (design point) finds the nearest TAG-5 glow voxel and warns if none; a vertical row of weak lamps gives a tall body an even wash.
  Shadowless lamps skip voxels below the ground surface, or they paint warm streaks on the underside.
