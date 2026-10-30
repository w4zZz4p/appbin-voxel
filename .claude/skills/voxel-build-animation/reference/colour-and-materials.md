# Colour and materials

The scene reads as a photograph when its colours do. Toy-like scenes come from saturated, bright, evenly coloured
materials; photographic ones from muted, slightly varied colours with the light in the shading, not the paint.

## Sampler workflow (`palette-sampler.html`)

1. Copy the photo to `ref/<name>.webp` and open the sampler from the served folder:
   `/.claude/skills/voxel-build-animation/palette-sampler.html?img=../../../ref/<name>.webp&crops=<name>:x0,y0,x1,y1;…&sky=x0,x1[,yBottom]&k=5`
   (crops as fractions of width/height; `k` 5–6 clusters per crop, default 5).
2. Screenshot the sampler first and place crops from its drawn boxes; guessed fractions land on shaded walls and
   return a lit material too dark (white stucco comes back grey). Sample **lit faces** of each material; sample a
   shaded face separately only to learn how deep the photo's shade goes.
3. Read `JSON.stringify(window.__palette)`: per crop, clusters with `hex`, `share` and `hsl` ([h 0–360, s, l]); a
   cluster marked `!` is outside the photographic range. Cluster shares say how often each shade appears — use
   them as the array weights in `P`.
4. Sky strip (`sky=`) where nothing crosses it; drop the last stop if a roof or tree contaminates it. Put the stops
   into `SKY.day` with the horizon band lighter than the zenith. No sky in the photo → pick a plausible muted
   gradient for the time of day and say so.
5. Sampled colours already include the photo's light: render, run `__compare()` against the photo, and let
   lamp-lit areas come out right through baked lamps rather than by brightening the base colour.

## Saturation and lightness limits

- Every colour in `P` stays within **sRGB saturation ≤ 0.4** and **lightness 0.12–0.85**; the palette lint after
  the bake warns otherwise. Fix by resampling, not by silencing.
- Exempt (`ACCENT` keys, `GLOW`, lamp colours): glow voxels, flowers, signage, painted accents, fire. Keep them small
  in area — saturation reads as an accent only against a muted field.
- Pure black (`l < 0.12`) and pure white (`l > 0.85`) never appear on a surface; iron is `0x2c2d2e`, "black" caps
  `0x232323`, white trim ≈ `0xd6d0c2`. Openings read dark through AO and a dark room colour, not through black.
- Glass tint `0x6b96b4` (s ≈ 0.33): visibly blue against walls, not a toy's cyan.
- Overcast photos push every material toward the sky's grey-blue: let the sampled values carry that; don't
  "correct" them back to catalogue colours.

## Real-world albedos (what the sampled values should land near)

Daylight sRGB ranges for the lit face of each material; the shaded face is ~0.6× the lightness and a little
cooler. If a sampled cluster is far outside its range, the crop is on a highlight, a shadow or a different
material.

| Material | Albedo | sRGB lightness | Hue | Saturation | Shades in `P` |
|---|---|---|---|---|---|
| Grass, lawn | 0.15–0.25 | 0.30–0.42 | 70–95° (olive) | 0.25–0.38 | 3 + a dry/worn shade |
| Foliage (deciduous) | 0.10–0.20 | 0.25–0.40 | 80–120° | 0.20–0.35 | 3 + ~5% yellowed |
| Conifer | 0.08–0.15 | 0.18–0.30 | 90–130° | 0.20–0.35 | 3 |
| Concrete, render | 0.30–0.50 | 0.55–0.72 | 30–60° (warm grey) | 0.02–0.10 | 2–3 + a streak shade |
| Brick | 0.20–0.35 | 0.30–0.45 | 10–22° | 0.28–0.40 | 3 + mortar |
| Fieldstone, limestone | 0.25–0.45 | 0.45–0.65 | 30–50° | 0.08–0.20 | 3–4 + mortar |
| Asphalt | 0.05–0.12 | 0.18–0.30 | any | ≤ 0.05 | 2 + patch + line paint |
| Slate, dark tile | 0.10–0.15 | 0.20–0.30 | 200–260° (cool) | ≤ 0.10 | 2–3 + ridge/cap |
| Clay tile | 0.25–0.35 | 0.35–0.48 | 12–25° | 0.30–0.40 | 3 + replaced tiles |
| Weathered timber | 0.20–0.35 | 0.40–0.55 | 25–40° | 0.08–0.18 | 3 |
| Painted metal, iron | 0.05–0.10 | 0.15–0.22 | any | ≤ 0.08 | 1–2 |
| Water (overcast) | — | 0.30–0.45 | 200–220° | 0.15–0.30 | navy + shallow band |
| Bare soil | 0.10–0.20 | 0.20–0.32 | 25–40° | 0.25–0.38 | 2 |

## Variation per material

- 3–4 shades per material plus a distinct mortar/joint colour, chosen from clusters of the same crop, never by
  hand-brightening one hex. One shade reads as plastic.
- The bake adds blotches, speckle, a slight per-blotch hue drift (`WEATHER.hue`), grime at the foot of walls,
  streaks and moss; the per-normal tint cools sides and lights tops. Keep `P` plain and let the bake do the
  variation.
- Big flat areas (a lawn, a plain wall) need a region function (`surfaceC`, a `vnoise` patch) on top of the shade
  array, or the hash speckle reads as noise rather than wear.
