# Prop inspection (subagents)

One pass over the whole scene misses prop faults; inspect **every prop individually**. The usual faults:
- vehicles at 1.5–1.7× or 0.7× real size, globe lamps as 40 cm cubes;
- a body type that doesn't match the photo (hatchback drawn with a sedan's deck);
- a fixture placed on a sight line (lamp post in front of a door in the hero view);
- glowing fixtures that light nothing, light pools with no fixture;
- colours brighter or more saturated than the photo's, or a prop cleaner than its surroundings.

## Registry (template)

- `prop(name, [w, h, l])` before a prop's code tags its voxels (`v.p`) and records its real size in metres
  (footprint order doesn't matter, `h` is height); `prop()` ends it. A voxel overwritten later takes the later
  name. Name props uniquely and by place (`car-drive`, `bench-terrace-w`, `lamp-door-l`); wrap fixture helper
  calls (`lantern`, `uplight`, `lampPost`) too.
- After the bake, `window.__props[name]` = `{ n, glow, glass, lamps, m: [x, h, z] metres, scale: [w, h, l]
  ratios, centre, issues }`, with console warnings for sizes outside 1.0–1.2× (± one cell) and glow with no lamp.
  `__seeProp(name, az, pad)` unticks Orbit and zooms onto the prop.
- `__seeProp(name, az)` orbits the island centre, so a prop far from it is seen off-axis (a sign on a rim block hid
  behind a tower at the expected angle); let the inspector tilt with a drag or pick the azimuth from the prop's `centre`.
  Blade signs on x-faces are edge-on from ±π/2: judge them from 0 / π.
- `__props` compares **bounding boxes**: a prop turned off the grid axes (a rotated animal, an open gate, a bale
  at 25°) reports a wider footprint — declare the turned box with a comment, or keep size-checked props near
  0°/90°. Whether the *shape* is right needs eyes.

## Running it

1. Build the scene, check it loads (console clean, small connectivity count, palette lint ok), read `__props`,
   and fix the console's size and glow warnings yourself first.
2. Split the props into **3–5 groups** (vehicles; garden furniture; lights and fixtures; trees and plants; yard
   clutter) — each agent reloads a 100k+ voxel page, so not one per bench. Launch them **in one message**
   (`general-purpose`, parallel).
3. Inspectors are **read-only**: they report, you edit. Parallel edits to one large file collide.
4. Apply fixes, reload, re-inspect only what failed (SendMessage to the same agent). Stop when every prop passes
   or what's left is a judgement call — list those in the report.

## Inspector brief (paste, filled in — it starts with no context)

> **Page:** `http://localhost:<port>/<file>.html?v=<N>` · file `<path>` · photo `<ref/….webp>` ·
> `M_PER_UNIT = <m>`, `S = <S>`.
> **Your props:** `<name>` — `<what it should be, e.g. "a red 5-door hatchback like the photo's">`; …
>
> **Browser:** open your own tab (`tabs_create`, then `navigate`) and pass its `tabId` to every call; never use or
> close another tab, don't resize the window. Wait ~5 s after load, press F
> (`dispatchEvent(new KeyboardEvent('keydown',{code:'KeyF'}))`), wait 2 s, call `__bake(true)` (the hidden-pane
> throttle leaves lamps half faded otherwise). For each prop call `__seeProp(name, az)` from the side it is seen
> from plus one other, and screenshot. Use `?sky=night` for a second look at anything with lights. Read each
> prop's code (search for `prop('<name>'`).
>
> **Check and report on:**
> - **Scale** — `__props[name].m` vs real size, and visually against a door (2.1 m) or storey at the same depth;
>   parts in proportion (wheel ≈ 0.65 m, seat 0.45 m, table 0.75 m, a car's hood vs cabin).
> - **Lighting** — every glowing part has a lamp (`lamps > 0`) with its pool *at* the fixture; no bright patch
>   without a fixture; brightness in line with similar fixtures; day and night both read; not black on its shadow
>   side or blown out.
> - **Grounding and contact** — on the surface, not sunk or hovering a cell up; wheels touching; face-to-face
>   contact with walls (brackets); contact shadow under it.
> - **Placement** — faces the right way; not clipping walls, paths, props or trunks; not blocking a door, path or
>   the hero view; inside its footprint; symmetric where the photo is.
> - **Shape and detail** — silhouette and body type match the photo or real object; enough detail for its size;
>   nothing missing (wheels both sides, table legs).
> - **Colour realism** — compare against the photo's prop, not a catalogue: no colour more saturated than its
>   photo counterpart (sRGB s ≤ 0.4 unless it is a painted accent), lightness in the material's daylight range,
>   3+ shades on any surface bigger than a few cells, wear where it is used; no debris specks around it.
>
> **Output** per prop: `PASS` or `FIX`; each issue with evidence (`__props` numbers, which view) and a concrete
> change (coordinates in design units / fine cells, which line or helper argument). End with one summary line per
> prop. Don't edit files.
