# Crane Operator

## Purpose

Crane Operator is a simple, child-friendly crane operation game for younger players. In a side view, the player moves a tower crane's trolley along the jib and raises or lowers its hook to pick up loads of different weights in a staging area and place them on marked building platforms before time runs out. The game introduces the idea that a crane's allowed lifting capacity decreases as the load moves farther from the tower. It has multiple levels with different site layouts, loads, and delivery targets.

## Inputs

The player does not type numeric values. Load weights, pickup points, delivery points, and the time limit are set by each game level.

| Input | Control | Behavior and limits | Default |
|---|---|---|---|
| Start or continue | Enter, or the on-screen button | Begins the current level or advances from its result screen | Game opens on the title/instructions screen |
| Move trolley toward tower | Left Arrow or on-screen left button | Moves the hook's horizontal radius inward at 7 ft/s; radius is clamped to 50–150 ft | Trolley starts at the level's pickup radius |
| Move trolley away from tower | Right Arrow or on-screen right button | Moves the hook's horizontal radius outward at 7 ft/s; radius is clamped to 50–150 ft | Trolley starts at the level's pickup radius |
| Raise hook | Up Arrow or on-screen up button | Raises the hook at 7 ft/s; hook height is clamped to 0–50 ft | Hook starts at the level's specified height |
| Lower hook | Down Arrow or on-screen down button | Lowers the hook at 7 ft/s; hook height is clamped to 0–50 ft | Hook starts at the level's specified height |
| Pick up or release load | Space or on-screen action button | Picks up a load at its marked pickup point or releases a carried load at its marked delivery point, when the hook is in the required position | No load is attached |
| Pause or resume | Escape, or the on-screen pause button | Pauses or resumes the level; the countdown stops while paused | Level is unpaused after starting |

Movement occurs only while a direction control is held. The game uses a 90-second countdown for each level. The load's weight and the chart capacity are displayed in pounds (lb); horizontal radius and hook height are displayed in feet (ft).

## Calculations

### Units and variables

- `r`: horizontal distance from the tower's centerline to the hook (radius), in ft.
- `h`: hook height above the ground, in ft.
- `v_r`: trolley speed, 7 ft/s.
- `v_h`: hoist speed, 7 ft/s.
- `dt`: elapsed simulation time for one update, in seconds.
- `W`: weight of the carried load, in lb.
- `C(r)`: crane's allowed load capacity at radius `r`, in lb.
- `U`: load utilization ratio, dimensionless; display it as `100 × U` percent.
- `t`: remaining level time, in seconds.
- `S`: cumulative game-session score, in points.

### Movement and limits

- While a horizontal key is held, update radius by `r_new = r_old + direction × v_r × dt`, where direction is `-1` toward the tower and `+1` away from it. Clamp `r_new` to 50–150 ft.
- While a vertical key is held, update hook height by `h_new = h_old + direction × v_h × dt`, where direction is `+1` to raise and `-1` to lower. Clamp `h_new` to 0–50 ft.
- While a level is active and unpaused, update the timer by `t_new = max(0, t_old - dt)`. Reaching zero ends the level as a time-out.
- A load can be picked up when the visible hook overlaps the crate. A delivery succeeds when the visible bottom of the carried crate is aligned with the marked platform. An unsuccessful delivery attempt keeps the load attached so the player can adjust it.
- The score starts at 0 for a new game session and increases by 100 points for each successfully delivered load. The score is cumulative across levels; there is no score penalty for a missed delivery, timeout, or tip-over.

### Environmental challenge levels

- Site 4 has strong wind. A simplified spring-and-gust model swings an attached load horizontally. The visible swung position is used for delivery alignment, so the player must compensate or wait for the load to pass over the platform.
- Site 5 has a second operating crane with a clearly marked red suspended load. Its load travels horizontally and vertically on a repeating path. If the player's carried load overlaps it, the current level immediately ends as a collision loss.
- Wind and second-crane motion stop while the game is paused. Both challenge paths are deterministic and leave safe windows for completing every delivery.

### Load chart and safety

This is an example chart for the game, not a real crane chart:

| Radius `r` (ft) | Capacity `C(r)` (lb) |
|---:|---:|
| 50 | 20,000 |
| 75 | 13,000 |
| 100 | 9,000 |
| 125 | 7,000 |
| 150 | 5,500 |

For radii between chart rows, calculate capacity by linear interpolation. Between chart points `(r1, C1)` and `(r2, C2)`:

`C(r) = C1 + ((r - r1) / (r2 - r1)) × (C2 - C1)`

At chart radii, use the listed capacity. Between rows, use linear interpolation as specified above. This is smoother and less conservative than using the next larger radius's capacity. The trolley cannot move outside the 50–150 ft range, so no extrapolation is needed. Hook height does not change chart capacity.

While a load is attached, calculate its utilization as `U = W / C(r)`.

- If `U >= 0.90` and `U <= 1.00`, show a prominent warning that the load is at least 90% of capacity. The warning includes the load, capacity, and utilization percentage.
- If `U > 1.00`, immediately play a dramatic arcade-style crane tip-over animation and fail the current level. Exactly 100% is not a tip-over, but the 90% warning is still shown.
- Evaluate the thresholds whenever the load's radius changes and immediately after pickup. Do not clamp trolley movement to a safe radius: crossing the 100% threshold must trigger the tip-over. Stop movement when the tip-over starts.
- An unattached load does not trigger a load-capacity warning or tip-over.

### Level result

- A level delivery counts when the load is released within both delivery tolerances and the crane has not tipped over.
- Each successful delivery advances the level's progress. The level is won when all assigned loads have been delivered before time reaches zero.
- Pressing the action control before the load is aligned with the platform gives corrective feedback and keeps the load attached.
- Time reaching zero before all deliveries are complete ends the level as a loss. A tip-over or a collision with the other crane also immediately ends the level as a loss.

## Outputs

The game displays:

- A side-view scene showing the tower crane and jib, trolley, hook, staging area, loads of different weights, building, and marked delivery platforms.
- The current load's weight (`W`, lb), current radius (`r`, ft), hook height (`h`, ft), chart capacity at the current radius (`C(r)`, lb), utilization (`100 × U`, %), remaining time (`t`, s), and score (`S`, points).
- The full load chart as a plot with radius (ft) on the horizontal axis and allowed load (lb) on the vertical axis. Mark the current radius with a vertical guide and mark the current load as a point at `(r, W)` so the player can compare it with the capacity curve. Also show the current numeric radius, load, and capacity. When no load is attached, show the current radius and capacity but no load point.
- Level progress (loads delivered / total loads), control hints, and clear states for ready, carrying, windy, crane traffic, collision, 90%-capacity warning, tip-over, paused, delivered, won, lost, or timed out.
- A prominent 90%-capacity warning and dramatic tip-over animation when utilization exceeds 100%, as defined above.

## Look and feel

- Use a bright, friendly, uncluttered cartoon style with large, recognizable crane and load shapes. Make the load, hook, staging area, and delivery target easy to distinguish.
- On first opening, show the game title, a short objective, control hints, and a prominent Start button. Do not start the timer until the player starts a level.
- Keep the play scene readable: crane and work area take most of the screen; status values and the load chart remain visible without covering the load or target.
- Use clear text and symbols as well as color for warnings and game states.
- On phones and tablets, use a responsive layout with large touch controls and an on-screen action button. Prefer landscape orientation for the play scene; keep controls reachable and prevent status panels from obscuring gameplay. Keyboard controls remain available on desktop.

## Validation

Check chart capacity and safety thresholds using linear interpolation. Values are calculated from the table; warning begins at `W >= 0.90 × C(r)`, and tip-over begins only at `W > C(r)`.

| Test radius | Hand calculation for allowed load `C(r)` | 90% warning threshold | Expected boundary behavior |
|---:|---|---:|---|
| 74.9 ft (just below the 75 ft table radius) | `20,000 + ((74.9 - 50) / (75 - 50)) × (13,000 - 20,000) = 13,028 lb` | `0.90 × 13,028 = 11,725.2 lb` | At `W = 11,725 lb`, no warning; at `W = 11,726 lb`, warn. At `W = 13,028 lb`, warn but do not tip; at `W = 13,029 lb`, tip over. |
| 75.1 ft (just above the 75 ft table radius) | `13,000 + ((75.1 - 75) / (100 - 75)) × (9,000 - 13,000) = 12,984 lb` | `0.90 × 12,984 = 11,685.6 lb` | At `W = 11,685 lb`, no warning; at `W = 11,686 lb`, warn. At `W = 12,984 lb`, warn but do not tip; at `W = 12,985 lb`, tip over. |
| 100 ft (exact table radius) | `C(100) = 9,000 lb` | `0.90 × 9,000 = 8,100 lb` | At `W = 8,099 lb`, no warning; at `W = 8,100 lb`, warn. At `W = 9,000 lb`, warn but do not tip; at `W = 9,001 lb`, tip over. |

Expected for all three tests: the chart cursor is at the tested radius, the load point is at `(r, W)`, and the numeric capacity and warning/tip state match the table. These tests establish that interpolation is used on both sides of a chart row, rather than snapping to a table value.

## Out of scope

- Real crane operation, engineering, certification, or safety advice; the example chart and game rules are fictional and must not be used to operate real equipment.
- Realistic crane physics, structural stress, wind simulation, rigging, sling-angle, ground-bearing, or counterweight calculations; the wind and collision challenges are arcade-style only.
- Multiplayer, online accounts, leaderboards, user-created levels, or editing the load chart.
- Free-form numeric input or user-configurable crane specifications.
