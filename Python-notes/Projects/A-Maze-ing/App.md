## Overview

`app.py` is the **application controller**. It owns the interactive menu loop and is responsible for handling all user input after the program has started. It sits between the user and the two core modules — `parser` (to reload settings) and `mazegen` (to generate and render the maze).

It is called from `parser.py` once the initial startup flow (`init`) has succeeded.

```
a-maze-ing.py
     ↓
  main.py  →  parser.load_settings("init")
                    ↓
               settings_parser()  →  Settings
                    ↓
               MazeGenerator.generate()
                    ↓
               app.run()   ← you are here
```

---

## Functions

### `clear_screen()`

```python
def clear_screen() -> None:
    system('cls' if name == 'nt' else 'clear')
```

Clears the terminal. Uses `os.system` with `cls` on Windows and `clear` on Unix/Linux. Called at startup and after each menu choice to keep the output clean.

> Uses `os.name` — `'nt'` means Windows, anything else is treated as Unix.

---

### `run(settings, maze)`

```python
def run(settings: Settings, maze: MazeGenerator) -> None:
```

The main interactive loop. Receives the already-validated `Settings` object and a ready `MazeGenerator` instance (which has already run `generate()` once before the menu appears).

**Parameters:**

| Parameter  | Type           | Description                              |
| ---------- | -------------- | ---------------------------------------- |
| `settings` | `Settings`     | Validated config from `parser.py`        |
| `maze`     | `MazeGenerator`| Initialised generator with a ready grid  |

---

## The Menu Loop

```python
while (True):
    choice = input(MENU + "\n    Select option: ").strip()
    ...
    match choice:
        case "1": ...
        case "2": ...
        case "3": ...
        case "4": ...
        case _:  print("Invalid option")
```

Uses Python's `match/case` (structural pattern matching, introduced in Python 3.10) as a clean alternative to `if/elif` chains.

The loop only exits when the user enters `"0"`.

---

## Menu Options

### `1` — Show settings

```python
settings.show_settings()
```

Calls the `show_settings()` method on the `Settings` object. Prints all current config values (width, height, entry, exit, output file, perfect, seed) to the terminal.

---

### `2` — Read new settings

```python
old_settings: Settings = settings
new_settings = load_settings("", "config.txt", "run")
```

Reloads `config.txt` at runtime without restarting the program. Uses the `"run"` flag so `load_settings` skips the script name validation.

Three possible outcomes:

| Condition | Result |
|---|---|
| Config file has errors | `None` returned — settings unchanged, error printed |
| Config is valid and **different** | New maze generated and rendered, settings updated |
| Config is valid but **same** | Old maze re-rendered, "No changes" printed |

The comparison `old_settings != new_settings` works because `Settings` is a Pydantic `BaseModel`, which supports equality checks out of the box.

---

### `3` — Generate a maze

```python
maze.generate()
maze.render_maze(color)
```

Re-runs the DFS algorithm with the current settings and renders the result. If `SEED` is set, this always produces the same maze. If `SEED` is `None`, each run produces a different one.

---

### `4` — Toggle color

```python
color = "blue" if color == "white" else "white"
```

Toggles the terminal render color between `white` and `blue`. If no maze has been generated yet, calls `generate()` first before rendering.

> Colors are applied via ANSI escape codes inside `render_maze()` in `mazegen.py`.

---

### `0` — Exit

Breaks the loop and exits the program cleanly with a printed message.

---

## State Management

`app.py` holds two mutable local variables across the loop:

| Variable | Default | Role |
|---|---|---|
| `settings` | passed in from `parser` | current active config |
| `color` | `"white"` | current render color for the visualizer |

The `maze` variable can also be reassigned inside the loop (case `"2"`) when new settings are loaded.
