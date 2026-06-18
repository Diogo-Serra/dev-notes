## Overview

`renderer.py` handles the Pygame visualisation. It takes the already-computed `Pathfinder` result and animates it — drawing hubs, connections, and drone positions tick by tick on a fullscreen window.

It has no routing logic of its own. It only reads `pathfinder.ticks` to know where drones are at each step.

---

## Pygame Basics

Pygame is a 2D graphics library built on SDL. The rendering loop follows a fixed pattern:

```
pygame.init()
screen = pygame.display.set_mode(...)

while running:
    # 1. handle input events
    for event in pygame.event.get(): ...

    # 2. clear the screen
    screen.fill(BG_COLOR)

    # 3. draw everything
    draw_connections()
    draw_hubs()
    draw_drones()
    draw_panel()

    # 4. push the frame to the display
    pygame.display.flip()

    # 5. cap the frame rate
    clock.tick(60)

pygame.quit()
```

> `pygame.display.flip()` swaps the back buffer to the screen. Without it, nothing you drew would be visible.  
> `clock.tick(60)` limits the loop to 60 frames per second to avoid burning CPU.

---

## Class Constants

```python
HUB_RADIUS    = 38       # circle radius for each hub in pixels
DRONE_BADGE_R = 13       # radius of the drone count badge
BG_COLOR      = (18, 18, 38)   # dark navy background
_MAX_SCALE    = 220      # max pixels-per-map-unit (prevents over-stretching)
```

---

## `__init__(nav_map, pathfinder)`

Sets up the window and pre-computes layout values.

**State initialised:**

| Attribute         | Description                                              |
| ----------------- | -------------------------------------------------------- |
| `_tick`           | Current tick index being displayed (starts at 0)         |
| `_playing`        | Whether auto-play is active                              |
| `_tick_delay_ms`  | Milliseconds between auto-advance ticks (default 600)    |
| `_last_tick_ms`   | Timestamp of the last tick advance (for timing)          |
| `_south_hubs`     | Set of hub names whose labels should render above instead of below |
| `_scale`          | Pixels per map unit, computed from screen size           |
| `_show_labels`    | Whether hub/link labels are drawn (toggled with `L`)     |

### `_south_hubs` detection

```python
for conn in nav_map.connections:
    if fx == tx:           # vertical connection
        if ty < fy:
            self._south_hubs.add(conn.from_hub)  # from is higher on screen
        if fy < ty:
            self._south_hubs.add(conn.to_hub)
```

Hubs that sit above a vertical connection have their name label pushed above the circle (using `midbottom`) instead of below (`midtop`) to avoid overlapping the connection line.

---

## Coordinate System — `_to_screen(x, y)`

```python
def _to_screen(self, x: int, y: int) -> tuple[int, int]:
```

Converts map coordinates (arbitrary integers from the map file) into screen pixel positions. Two things happen:

1. **Scale** — multiply by `self._scale` to stretch the map to fit the window.
2. **Y-flip** — map Y increases upward (like a standard graph), screen Y increases downward. The formula inverts Y:

```python
sy = oy + map_h - (y - self._min_y) * self._scale
```

The map is also centred within the available area using an offset `ox, oy`.

---

## `_compute_scale()`

Fits the map into the drawable area while respecting `_MAX_SCALE`:

```python
scale = min(avail_w / span_x, avail_h / span_y)
return min(scale, _MAX_SCALE)
```

Takes the smaller of the horizontal and vertical fits so the map is never clipped. Caps at `_MAX_SCALE` to prevent hubs becoming enormous on small maps.

---

## `run()` — The Main Loop

```python
def run(self) -> None:
```

Handles input events and drives rendering until the user quits.

### Keyboard controls

| Key     | Action                                      |
| ------- | ------------------------------------------- |
| `ESC`   | Quit                                        |
| `P`     | Toggle play / pause                         |
| `SPACE` | Advance one tick (only when paused)         |
| `R`     | Reset to tick 0                             |
| `L`     | Toggle hub/link labels on and off           |

### Auto-advance timing

```python
if self._playing:
    now = pygame.time.get_ticks()
    if now - self._last_tick_ms >= self._tick_delay_ms:
        self._advance_tick()
        self._last_tick_ms = now
```

Checks elapsed time each frame. Advances the tick only when `_tick_delay_ms` milliseconds have passed. This is time-based, not frame-based, so it's unaffected by frame rate.

> `pygame.time.get_ticks()` returns milliseconds since `pygame.init()`.

---

## Drawing Methods

### `_draw_connections()`

Draws a grey line between each pair of connected hubs. If labels are on, also renders the `max_link_capacity` value offset perpendicular to the line (so it doesn't sit on top of it).

```python
# perpendicular offset to avoid overlapping the line
px, py = -dy / length, dx / length   # rotate 90°
lx = midpoint_x + int(px * offset)
```

### `_draw_hubs()`

Draws a filled circle for each hub, then a white border ring. If labels are on, renders the hub name outside the circle and the `(x,y)` / `max_drones` values inside.

Hub colour comes from `hub.metadata.get('color', 'white')` and is resolved through `_resolve_color()` which maps colour name strings to RGB tuples.

### `_draw_drones()`

Reads `self._pf.ticks[self._tick]` — a list of hub names, one per drone — and counts how many drones are at each hub using a simple `dict`.

For each hub that has drones, draws an orange badge circle offset to the top-right of the hub circle, with the drone count number inside.

```python
bx = sx + int(HUB_RADIUS * 0.72)
by = sy - int(HUB_RADIUS * 0.72)
```

### `_draw_panel()`

The top bar. Renders:
- Map name, difficulty, and drone count (centred, large font)
- Current tick, total ticks, flow reached vs drones requested (bottom-left, colour-coded green when playing, amber when paused)
- Keyboard shortcut reference (centred, small font)

### `_draw_map_info()`

A floating info box in the top-right corner of the map area. Shows name, difficulty, drone count, hub count, and link count. Uses `pygame.SRCALPHA` for a semi-transparent dark background.

```python
bg = pygame.Surface((box_w, box_h), pygame.SRCALPHA)
bg.fill((10, 10, 28, 210))   # RGBA — last value is alpha (0=transparent, 255=opaque)
```

> `pygame.SRCALPHA` tells Pygame the surface has an alpha channel. Without this flag, the alpha value in `fill()` would be ignored.

---

## `_blit_centered(surf, cx, cy)`

```python
rect = surf.get_rect(center=(cx, cy))
self._screen.blit(surf, rect)
```

A small helper that blits (pastes) a surface onto the screen centred on a given pixel. `blit` without this would anchor to the top-left corner instead.
