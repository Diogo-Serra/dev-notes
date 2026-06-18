## Overview

`bootstrap.py` and `app.py` together handle everything that is not the algorithm — startup validation, the interactive menu, and execution routing.

```
fly-in.py
     ↓
bootstrap.initialize(argv)
     ↓
app.run()   ← menu loop lives here
```

---

## `bootstrap.py`

The entry point module. Its only job is to validate `argv`, call `app.run()`, and catch any errors that bubble up from anywhere in the program.

### `initialize(argv)`

```python
def initialize(argv: list[str]) -> None:
```

Called from `fly-in.py` with `sys.argv`.

**What it does:**

1. Strips whitespace from each argument.
2. Checks `argv[0]` is `"fly-in.py"` — if not, prints usage and returns.
3. Calls `app.run(script)` to start the menu loop.
4. Catches errors with specific messages for each type.

### Error handling

| Exception          | Message printed                                   |
| ------------------ | ------------------------------------------------- |
| `ValidationError`  | Per-field location + message + format hint        |
| `FileNotFoundError`| OS error message + usage hint                    |
| `ValueError`       | Error message directly                            |
| `Exception`        | "Unexpected error: ..."                           |

> `BaseException` is intentionally **not** caught — that would swallow `KeyboardInterrupt` and `SystemExit`, which should be allowed to propagate.

> The module-level `from pydantic import ValidationError` is wrapped in a `try/except ImportError` with `exit(1)`. This is one of the valid uses of `exit()` inside an except — without it, the rest of the module would load and then crash with a `NameError` when `ValidationError` is referenced.

---

## `app.py`

The interactive menu loop. Runs after bootstrap validation succeeds and stays alive until the user exits.

### State

```python
_map: Map | None = None
handler: FileHandler | None = None
```

Both are `None` until the user selects a map. Persisted across the loop so all menu options can access them.

### The Menu Loop

```python
while True:
    choice = input(MENU + "Select option: ").strip()
    match choice:
        case "1": ...
        case "2": ...
        case "3": ...
        case "0": break
        case _:   print("Invalid option")
```

Uses Python's `match/case` (structural pattern matching, Python 3.10+) as a clean alternative to `if/elif` chains.

---

### Menu Options

#### `1` — Select Map

```python
handler = FileHandler(filename=map_file)
_map = handler.read_map_file()
_map.map_info()
```

Opens the map selection sub-menu (`map_menu()`), creates a `FileHandler`, parses the file into a `Map`, and prints map info. Both `handler` and `_map` are stored for use in other options.

#### `2` — See Map Info

```python
_map.map_info()
```

Prints the current map's name, difficulty, drone count, and hub list. Guards against no map being selected.

#### `3` — Navigation System

```python
pf = Pathfinder(_map)
handler.write_solution(pf.ticks, pf.end_name)
Renderer(_map, pf).run()
```

Three steps in sequence:
1. `Pathfinder` runs the full algorithm — Dinic flow, path extraction, tick simulation.
2. `handler.write_solution()` writes the result to `solution/<name>.txt`.
3. `Renderer` opens the Pygame window for animated playback.

> `handler` is needed here (not just `_map`) because `write_solution` is a method that uses `handler.selected_map`.

---

### `map_menu(path_maps)`

```python
def map_menu(path_maps: str) -> str | None:
```

Reads all files from `src/maps/`, lists them numbered, and returns the chosen filename. Returns `None` if the user enters `0` (back).
