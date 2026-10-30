# Animation modes

`ANIM = { intro, order, motion, outro, sky }` at the top of the template sets the scene's defaults; `?intro=`,
`?order=`, `?motion=`, `?outro=`, `?sky=` override them. The UI dropdowns switch live: order and motion re-run
`plan()` (recompute `t0`, re-sort, new `DUR`/`BUILD_END`) and replay, and the URL is updated with
`history.replaceState` so a combination can be shared. Picking *3D print* also switches the motion to *Grow*.
Default is `layers/drop/reset`.

| Key | Value | What it does |
|---|---|---|
| order | `layers` | bottom-up, hash jitter + a little centre-out spread |
| | `scan` | 3D printer: strict layers (`SCAN` s each), rows rastered along z in alternating directions, a translucent orange plate rides on the current layer |
| | `stages` | by `STAGE` tag: each stage builds bottom-up from its own lowest voxel; the next starts at ~80% of it |
| motion | `drop` | falls `6*S` cells with an ease-out-back scale pop (0.55 s) |
| | `grow` | extrudes upward from the voxel's floor with slight overshoot (0.45 s) |
| | `swarm` | from a cloud up to `32*S` cells out and above, swirls in on a curve, scales up (1.6 s) |
| | `tornado` | rides a funnel round the island centre, spiralling down and in (1.8 s) |
| | `slide` | slides `9*S` cells in along its open side facing away from the centre; voxels with no open side drop (0.7 s) |
| | `pop` | elastic scale-up in place (0.6 s) |
| | `tumble` | falls from higher spinning about a random axis, lands square (0.9 s; `setR`) |
| | `squash` | falls stretched, flattens on impact, springs back, volume kept (0.75 s) |
| | `flash` | drops in hot white and cools to its colour (0.9 s; writes instance colours) |
| intro | `none` | build starts immediately |
| | `blueprint` | blue outline of every exposed non-terrain voxel fades in for `BLUEPRINT` (2.5 s), the build fills it, it fades out |
| outro | `reset` | hold 30 s, restart |
| | `demolish` | hold 30 s, come apart in reverse build order (thrown outward, gravity, shrink), rebuild. `D` any time |
| | `pile` | everything above the terrain arcs onto a heap (last built at the bottom), holds, flies back in build order. `P` any time |
| sky | `day` | stays as the photo |
| | `night` | 2 s after the build dusk fades in: the sky texture and haze colour lerp to `SKY.night`, hemisphere/sun dim and tint blue, glow and hero lights ×2.2 (baked lamps need no boost), UI switches to light text (`body.dark`); with `?bg=css` the `#night` overlay fades in instead |

## Engine notes

- **Stages:** set `STAGE = n` before each part (0 ground, 1 walls, 2 windows/doors/lanterns, 3 roof, 4
  trees/props; `TAG = 4` around the terrain loop). A voxel overwritten later takes the later stage, so windows
  punched into a wall appear as stage 2. Unset scenes fall back to one stage.
- **Adding a motion:** a `POSE.name(v, p)` that ends at `(v.x, v.y, v.z)` with scale 1 as `p → 1`, its duration
  in `DURS`, an `<option>` in the Motion dropdown. `setM` takes separate x/y/z scales and clears rotation, so a
  rotated voxel (`setR`) lands square. A motion that changes colour needs `POSE.name.done(v)` to restore it;
  switching motion calls `restoreColours()`.
- Anything indexed by position in `vox` (demolish arrays `REL`/`VX`/…) must be dropped when `plan()` re-sorts.
- **Demolish follows `t0`, not height:** keyed to height, lawn and house release together and read as one
  explosion.
- Effects that move "the building" skip terrain with `v.y > topAt(v.x, v.z)`, not `v.y >= TOP`: lawn hummocks
  are above `TOP`, and moving them exposes the culled hollow island as dark holes.
- `slide` picks the open side facing **away from the island centre**; the first open side found is often the
  inside of a room.
- **Blueprint ghost writes depth**, draws last and is scaled 0.96 so landed voxels hide it; with
  `depthWrite: false` overlapping faces stack into a milky blob.
- `scan` shows a hollow bowl while printing the culled underside — it reads as infill; leave it.
- **Morph to another scene** isn't built: `pile` already pairs every voxel with a slot, so a morph would use
  another scene's voxels as targets (paired by colour). That needs scenes exported as data.
