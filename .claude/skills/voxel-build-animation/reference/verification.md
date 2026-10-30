# Verification (in the Claude browser pane)

## Serving the page

- `file://` pages render as static snapshots in the pane — serve the project folder. `.claude/launch.json` has
  `static-py3` (port 8783) and `static-py3-b` (8791), both `python3 -m http.server` with `"autoPort": false`
  (this Mac has no `python`). `preview_start` one, then navigate to `http://localhost:<port>/<file>.html`.
- If another chat already serves one port, start the other config: with a reassigned port the server still binds
  the hard-coded one and navigation fails. If a borrowed server goes away, start your own.
- Both ports taken by other chats → add a third `python3 -m http.server` config (e.g. `static-py3-c`, 8797) to
  launch.json and start that; a borrowed server works for a sampler page but the hook warns the pane may not reach it.
- Add `?v=N` to bust the cache after edits; `http.server` lets the browser reuse an old copy of the same URL, so a
  reload without a new `v` can show the previous build.

## Timing gotchas

- Generating 100k+ voxels takes a few seconds; wait ~5 s after load.
- While the pane is hidden, frames are throttled: a screenshot right after load can show an empty sky (check the
  `#bar i` width), and lamps stay half faded after `F` — call `__bake(true)` before judging lighting.
- The pane can stop animation frames entirely (fps readout blank, `#bar i` frozen); a screenshot wakes it, so
  interleave screenshots in a `browser_batch` while waiting on the build. The readout shows `idle` when nothing
  redraws, which is expected for a settled scene with Orbit off.
- `?sky=night` in a hidden pane may never reach night (frames throttled): call `__night(1)` to force it, `__night()`
  for automatic.
- A drag unticks Orbit; ticking it again orbits on from the dragged view (keeps zoom and tilt). Reload or
  `__seeAz()` before judging the default framing.
- A screenshot straight after `__seeAz` / `__zoom` captures the previous frame, so two views look identical:
  `await new Promise(r => setTimeout(r, 500))` before each capture, in a `browser_batch` of view + screenshot pairs.
- In a hidden pane an awaited `__compare()` can outlive the tool's 45 s limit: start it without awaiting
  (`__compare(src, 0).then(r => window.__cmp = r)`) and read `__cmp` in a later call.

## Test hooks

`F` skip to end · `__seeAz(angle)` frame a side · `__zoom(z, x, y, z)` zoom the camera onto a world point
(the screenshot `zoom` action only crops pixels) · `__bake(true|false)` force baked lamps on/off, `__bake()` back
to automatic · `__hero(false)` hide hero PointLights · `__lamps` baked lamps · `__props` / `__seeProp(name, az)`
props · `__lost` voxels removed by connectivity, bucketed by 4-unit cells · `__compare(src, opacity)` photo
overlay and per-region colour table (below) · `__shadow('map' | 'bake')` force the sun's shadow source (shadow
map vs baked faces), `__shadow()` back to automatic, returning `{ k, castShadow }`. URL flags: `?grid=1` full cube
grid, `?cam=persp` perspective hero view, `?bg=css` CSS sky instead of the texture, `?shadow=lo|hi` 1024 / 4096 shadow map (default 2048), `?dpr=N` pins the pixel ratio.

## Checks

1. Console: no errors, a small connectivity count, `bake: N ms` (up to ~2 s is fine), `palette lint: ok`, voxel
   count in range (60k–260k is normal at `S = 2`). No `prop X: no voxels left` or lamp-dropped warnings.
2. Connectivity count in the hundreds or more → read `__lost`. Typical culprits: an arm meeting its carriage only
   at a corner, a window room with no reveal joining it to the wall, a dish not joined to its mast, a hose one
   cell off a tank, a sprite floated by `q(0) = 1`.
3. Views: `F`, wait 3 s, untick Orbit, `__seeAz(0.3)` (front), `__seeAz(Math.PI + 0.3)` (back), `±Math.PI/2`
   (sides); screenshot each, then `__zoom(3, …)` onto windows, arches, glass and every prop group from the side
   it is seen from. Look for occluders in front of the subject along each view and dark specks on grass.
4. Photo check: frame the photo's angle, screenshot, and compare silhouette, proportions, main material colours
   and where the light falls. `__bake(false)` / `__bake(true)` shows what the lamps contribute.
5. Look check: screenshot also with `?cam=persp` — haze visible toward the back of the island, no cube grid
   readable at full zoom, no colour more saturated than the photo's.

## Photo comparison (`__compare`)

`__compare('ref/<name>.webp', 0.5)` lays the photo over the render (`object-fit: cover`, pointer-events off) at
that opacity and prints a `console.table` of the mean colour per 3×3 screen region for render and photo: hex,
sRGB saturation / lightness, and the differences `ds`, `dl`. It returns the rows. `__compare()` removes the overlay.

1. Frame the photo's angle (`__seeAz`, or drag, then `__zoom` so the subject fills the same part of the screen as
   in the photo); press `F` and `__bake(true)` first.
2. Screenshot at opacity 0.5 for silhouette and placement, then at 0 and 1 for colour.
3. Read the table region by region. Regions that are mostly one material (lawn bottom row, wall centre, sky top
   row) are the useful ones; `|ds| > 0.1` or `|dl| > 0.12` there means resample that material, not tweak the
   light. Lamp-lit regions are judged with `__bake(false)` as well.
4. The sky regions compare the texture against the photo's sky: fix `SKY.day`, not the haze.
