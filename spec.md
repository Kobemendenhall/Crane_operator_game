# Crane Operator

## Purpose

Crane Operator is a simple, child-friendly crane operation game for younger players. The player moves a tower crane's trolley and hook to pick up loads in a staging area and place them on marked building platforms before time runs out. The game introduces the idea that a crane's safe lifting capacity decreases as the load moves farther from the tower.

## Inputs

The player does not type numeric values. Load weights, pickup points, delivery points, and the time limit are set by each game level.

| Input | Control | Behavior and limits | Default |
|---|---|---|---|
| Start or continue | Enter, or the on-screen button | Begins the current level or advances from its result screen | Game opens on the title/instructions screen |
| Move trolley toward tower | Left Arrow or on-screen left button | Moves the hook's horizontal radius inward at 5 ft/s; radius is clamped to 50–150 ft | Trolley starts at the level's pickup radius |
| Move trolley away from tower | Right Arrow or on-screen right button | Moves the hook's horizontal radius outward at 5 ft/s; radius is clamped to 50–150 ft | Trolley starts at the level's pickup radius |
| Raise hook | Up Arrow or on-screen up button | Raises the hook at 5 ft/s; hook height is clamped to 0–50 ft | Hook starts at the level's specified height |
| Lower hook | Down Arrow or on-screen down button | Lowers the hook at 5 ft/s; hook height is clamped to 0–50 ft | Hook starts at the level's specified height |
| Pick up or release load | Space or on-screen action button | Picks up a load at its marked pickup point or releases a carried load at its marked delivery point, when the hook is in the required position | No load is attached |
| Pause or resume | Escape, or the on-screen pause button | Pauses or resumes the level; the countdown stops while paused | Level is unpaused after starting |

Movement occurs only while a direction control is held. The game uses a 120-second countdown for each level. The load's weight and the chart capacity are displayed in pounds (lb); horizontal radius and hook height are displayed in feet (ft).

## Calculations

### Units and variables

- `r`: horizontal distance from the tower's centerline to the hook (radius), in ft.
- `h`: hook height above the ground, in ft.
- `v_r`: trolley speed, 5 ft/s.
- `v_h`: hoist speed, 5 ft/s.
- `dt`: elapsed simulation time for one update, in seconds.
- `W`: weight of the carried load, in lb.
- `C(r)`: crane's allowed load capacity at radius `r`, in lb.
- `t`: remaining level time, in seconds.

### Movement and limits

- While a horizontal key is held, update radius by `r_new = r_old + direction × v_r × dt`, where direction is `-1` toward the tower and `+1` away from it. Clamp `r_new` to 50–150 ft.
- While a vertical key is held, update hook height by `h_new = h_old + direction × v_h × dt`, where direction is `+1` to raise and `-1` to lower. Clamp `h_new` to 0–50 ft.
- While a level is active and unpaused, update the timer by `t_new = max(0, t_old - dt)`. Reaching zero ends the level as a time-out.
- A load can be picked up or released only when the hook is within 2 ft horizontally of the marked point and within 1 ft vertically of its marked pickup or delivery height.

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

At exactly 50–150 ft, use the corresponding chart capacity. The trolley cannot move outside that range, so no extrapolation is needed. A load is safe to carry when `W <= C(r)`. Equality is allowed. If moving the trolley would make `W > C(r)`, stop the trolley at the last safe position and show a clear overload warning; do not allow the crane to tip. Hook height does not change chart capacity.

### Level result

- A level is delivered when the load is released within both delivery tolerances and the load is safe at that radius.
- Each successful delivery advances the level's progress. The level is won when all assigned loads have been delivered before time reaches zero.
- Releasing outside the marked delivery tolerances leaves the load where it lands and counts as a missed delivery; the player can continue if time remains.
- Time reaching zero before all deliveries are complete ends the level as a loss.

## Outputs

The game displays:

- A side-view scene showing the tower crane, trolley, hook, staging area, loads, building, and marked delivery platforms.
- The current load's weight (`W`, lb), current radius (`r`, ft), hook height (`h`, ft), chart capacity at the current radius (`C(r)`, lb), and remaining time (`t`, s).
- A load-chart table or plot with the current radius and capacity highlighted.
- Level progress (loads delivered / total loads), control hints, and clear states for ready, carrying, overload-prevented, paused, delivered, won, lost, or timed out.
- A visible warning when the current load would exceed the capacity at a requested radius. The trolley remains at the last safe radius.

## Look and feel

- Use a bright, friendly, uncluttered cartoon style with large, recognizable crane and load shapes. Make the load, hook, staging area, and delivery target easy to distinguish.
- On first opening, show the game title, a short objective, control hints, and a prominent Start button. Do not start the timer until the player starts a level.
- Keep the play scene readable: crane and work area take most of the screen; status values and the load chart remain visible without covering the load or target.
- Use clear text and symbols as well as color for warnings and game states.
- On phones and tablets, use a responsive layout with large touch controls and an on-screen action button. Prefer landscape orientation for the play scene; keep controls reachable and prevent status panels from obscuring gameplay. Keyboard controls remain available on desktop.

## Validation

Hand-check one safe delivery and the chart calculation:

1. Initial state: level timer is 120 s; a load weighing `W = 8,000 lb` is attached; current radius is `r = 50 ft`; hook height and marked delivery height are both `h = 20 ft`; the delivery point is at `r = 75 ft`.
2. The player holds Right Arrow for 5 seconds. With `v_r = 5 ft/s`, the radius change is `5 ft/s × 5 s = 25 ft`, so the new radius is `50 ft + 25 ft = 75 ft`.
3. At 75 ft, the chart gives `C(75) = 13,000 lb`. Since `8,000 lb <= 13,000 lb`, the load is safe and no overload warning appears.
4. The player presses Space at the delivery point. Expected result: the load is delivered, level progress increases by one, the displayed radius, load weight, and capacity are 75 ft, 8,000 lb, and 13,000 lb respectively, and the timer is 115 s.

Interpolation check: at `r = 62.5 ft`, halfway between 50 ft and 75 ft, `C(62.5) = 20,000 + ((62.5 - 50) / (75 - 50)) × (13,000 - 20,000) = 16,500 lb`.

## Out of scope

- Real crane operation, engineering, certification, or safety advice; the example chart and game rules are fictional and must not be used to operate real equipment.
- Realistic crane physics, structural stress, wind, rigging, sling-angle, ground-bearing, or counterweight calculations.
- Multiplayer, online accounts, leaderboards, user-created levels, or editing the load chart.
- Free-form numeric input or user-configurable crane specifications.
