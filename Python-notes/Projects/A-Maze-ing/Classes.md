## Overview

`mazegen.py` is the **core module** of the project. It defines two classes:

| Class | Role |
|---|---|
| `Settings` | Pydantic model — validates and stores the maze configuration |
| `MazeGenerator` | Generates, renders and saves the maze using DFS |

Both are exported from the package's `__init__.py` so they can be used standalone:

```python
from mazegen import Settings, MazeGenerator
```

---

## `Settings`

```python
class Settings(BaseModel):
    WIDTH: int = Field(ge=3, le=40)
    HEIGHT: int = Field(ge=3, le=40)
    ENTRY: tuple[int, int]
    EXIT: tuple[int, int]
    OUTPUT_FILE: str
    PERFECT: bool
    SEED: int | None = None
```

A [Pydantic](https://docs.pydantic.dev/) `BaseModel` that validates all input before anything runs. Pydantic enforces types and runs validators automatically when the object is created.

### Fields

| Field | Type | Constraints |
|---|---|---|
| `WIDTH` | `int` | 3 – 40 |
| `HEIGHT` | `int` | 3 – 40 |
| `ENTRY` | `tuple[int, int]` | must be in-bounds, not on `42` pattern |
| `EXIT` | `tuple[int, int]` | must be in-bounds, not on `42` pattern, ≠ ENTRY |
| `OUTPUT_FILE` | `str` | must start with a letter, only `a-z 0-9 _ -`, ends with `.txt` |
| `PERFECT` | `bool` | `True` = no loops, `False` = extra passages added |
| `SEED` | `int \| None` | optional — `None` = random each run |

---

### Validators

Pydantic validators run automatically when `Settings(...)` is called. There are three:

#### `parse_tuple` — `@field_validator('ENTRY', 'EXIT', mode="before")`

```python
@classmethod
def parse_tuple(cls, value: str) -> tuple[int, int]:
    clean = value.strip(ascii_letters + '() ')
    x, y = clean.split(",")
    return (int(x.strip()), int(y.strip()))
```

Runs **before** type conversion. Parses the raw string from `config.txt` (e.g. `"(0, 0)"`) into a proper `tuple[int, int]`. Strips surrounding letters, parentheses and spaces, then splits on the comma.

> `mode="before"` means it intercepts the raw string value before Pydantic tries to coerce it into a tuple.

---

#### `validator_output_file` — `@field_validator('OUTPUT_FILE', mode="after")`

```python
@classmethod
def validator_output_file(cls, name: str) -> str:
```

Runs **after** Pydantic has accepted the string. Checks:
1. Name is not empty after removing `.txt`
2. First character is a letter
3. Only alphanumeric + `_` and `-` characters
4. Must end with `.txt`

Returns the validated name unchanged if all checks pass, otherwise raises `ValueError`.

---

#### `validator_maze` — `@model_validator(mode='after')`

```python
@model_validator(mode='after')
def validator_maze(self) -> Settings:
```

Runs **after all fields are validated**, so it can access the full object. Checks:
- `ENTRY` and `EXIT` x-coordinates are within `WIDTH`
- `ENTRY` and `EXIT` y-coordinates are within `HEIGHT`
- No coordinate is negative
- `ENTRY ≠ EXIT`
- Neither `ENTRY` nor `EXIT` overlaps with a cell in the `42` pattern

> `@model_validator` vs `@field_validator` — field validators check one field at a time, model validators check the whole object at once.

---

### `show_settings()`

```python
def show_settings(self) -> None:
```

Pretty-prints all current settings to stdout. Called from `app.py` menu option `1`.

---

## `MazeGenerator`

```python
class MazeGenerator:
    def __init__(self, settings: Settings) -> None:
```

The maze engine. Takes a validated `Settings` object and uses it to drive DFS generation, terminal rendering and file output.

### Attributes

| Attribute | Type | Description |
|---|---|---|
| `settings` | `Settings` | full config object |
| `WIDTH / HEIGHT` | `int` | grid dimensions |
| `ENTRY / EXIT` | `tuple[int, int]` | start and end coordinates |
| `OUTPUT_FILE` | `str` | filename for saved maze |
| `PERFECT` | `bool` | whether to add extra passages |
| `SEED` | `int \| None` | RNG seed |
| `grid` | `list[list[int]] \| None` | the maze — `None` until `generate()` runs |
| `fixed` | `set[tuple[int, int]]` | cells blocked by the `42` pattern |

---

### `_compute_fixed_cells(width, height)` — `@staticmethod`

```python
@staticmethod
def _compute_fixed_cells(width: int, height: int) -> set[tuple[int, int]]:
```

Returns the set of `(x, y)` positions that form the `42` digit pattern, centered in the grid. Returns an empty set if the grid is smaller than `9×7` (the minimum to fit the pattern with a 1-cell border).

The digits are stored as 5×3 bitmaps:

```python
four = [
    [1, 0, 1],
    [1, 0, 1],
    [1, 1, 1],
    [0, 0, 1],
    [0, 0, 1],
]
two = [
    [1, 1, 1],
    [0, 0, 1],
    [1, 1, 1],
    [1, 0, 0],
    [1, 1, 1],
]
```

`1` = cell is part of the pattern (added to `fixed`). DFS pre-marks these as visited so it never carves through them — leaving them as solid walled islands.

---

### `generate()`

```python
def generate(self) -> None:
```

The main DFS algorithm. See [[DFS - mazegen]] for a full step-by-step walkthrough.

In brief:
1. Fills the grid with `0xF` (all walls set)
2. Pre-marks fixed cells as visited
3. Runs iterative DFS with backtracking to carve passages
4. If `PERFECT=False`, randomly removes extra walls after DFS
5. Calls `save_maze()` at the end

---

### `save_maze()`

```python
def save_maze(self) -> None:
```

Writes the maze to `Output_<OUTPUT_FILE>`. Format:

```
FFFFFFF...   ← each row, one hex char per cell (uppercase)
FFFFFFF...

(0, 0)       ← ENTRY coordinates
(14, 9)      ← EXIT coordinates
Pathfinder   ← placeholder for future pathfinding output
```

Each hex character is a single digit `0`–`F` representing the bitmask of walls remaining in that cell.

---

### `show_maze()`

```python
def show_maze(self) -> None:
```

Prints the raw hex grid to stdout. Useful for debugging. Raises `ValueError` if called before `generate()`.

---

### `render_maze(color)`

```python
def render_maze(self, color: str) -> None:
```

Renders the maze as ASCII art in the terminal using ANSI escape codes.

**How it works:**
- Iterates row by row, printing a **top wall row** and a **cell row** for each
- Wall characters: `+` for corners, `---` for horizontal walls, `|` for vertical walls
- Empty space `   ` where a wall has been carved
- Special cells:
  - `fixed` cells → yellow background (`\033[43m`)
  - `ENTRY` → ` S `
  - `EXIT` → ` E `

**Colors:**

| Value | Effect |
|---|---|
| `"white"` | No extra color codes — uses terminal default |
| `"blue"` | Applies `\033[94m` (bright blue) to walls |

> `RESET = "\033[0m"` is appended after every colored segment to avoid color bleed.
