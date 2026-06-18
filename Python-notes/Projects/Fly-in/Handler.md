## Overview

`handler.py` owns all file I/O for the project. It handles two jobs:

| Job                  | Method               | When                          |
| -------------------- | -------------------- | ----------------------------- |
| Read a map file      | `read_map_file()`    | Menu option 1 — Select Map    |
| Write a solution file| `write_solution()`   | Menu option 3 — after routing |

`FileHandler` is a Pydantic `BaseModel`, so the filename is validated the moment the object is created — before any file is opened.

---

## `FileHandler`

```python
class FileHandler(BaseModel):
    filename: str
    name: str = ""
    nb_drones: str = ""
    difficulty: str = ""
    selected_map: Map | None = None
    hub_list: list[Hub] = Field(default_factory=lambda: list())
    connections: list[Connection] = Field(default_factory=lambda: list())
```

### Fields

| Field          | Type                  | Description                                        |
| -------------- | --------------------- | -------------------------------------------------- |
| `filename`     | `str`                 | The `.txt` filename — validated on creation        |
| `name`         | `str`                 | Map display name, populated during parsing         |
| `nb_drones`    | `str`                 | Raw drone count string, converted to `int` by `Map`|
| `difficulty`   | `str`                 | Difficulty label, populated during parsing         |
| `selected_map` | `Map \| None`         | The fully built `Map` object after parsing         |
| `hub_list`     | `list[Hub]`           | Hubs accumulated during parsing                    |
| `connections`  | `list[Connection]`    | Connections accumulated during parsing             |

---

### Validator — `filename_validator`

```python
@field_validator('filename', mode='after')
def filename_validator(cls, filename: str) -> str:
    if filename not in listdir("src/maps/"):
        raise FileNotFoundError(...)
    return filename
```

Runs when `FileHandler(filename=...)` is called. Checks that the file exists in `src/maps/` before anything else runs. Raises `FileNotFoundError` (caught by `bootstrap.py`) if not found.

> `mode='after'` means it runs after Pydantic has already confirmed the value is a `str`.

---

### `read_map_file()`

```python
def read_map_file(self) -> Map:
```

Reads and parses the map file line by line, building up `self.hub_list` and `self.connections`, then assembles a `Map`.

**Parsing rules:**

| Line prefix   | Action                                                           |
| ------------- | ---------------------------------------------------------------- |
| `#`           | Comment — if line 1 and has `:`, splits into difficulty + name  |
| `nb_drones:`  | Sets `self.nb_drones`                                            |
| `start_hub:`  | Parses hub data, injects `hub_type=start` into metadata, appends to `hub_list` |
| `end_hub:`    | Parses hub data, injects `hub_type=end` into metadata, appends to `hub_list`   |
| `hub:`        | Parses hub data, appends to `hub_list`                           |
| `connection:` | Splits `from-to`, parses metadata, appends to `connections`      |

**Hub data format** (after splitting the line value):

```
name  x  y  [key=value] [key=value] ...
 [0] [1] [2]    [3:]
```

Returns the assembled `Map` object (also stored in `self.selected_map`).

---

### `metadata_parser(metadata)`

```python
@staticmethod
def metadata_parser(metadata: list[str]) -> dict[str, str]:
```

Converts a list of `[key=value]` strings into a plain `dict`.

```python
["[max_drones=5]", "[zone=priority]"]
→ {"max_drones": "5", "zone": "priority"}
```

Strips the square brackets before splitting on `=`.

---

### `write_solution(ticks, end_name)`

```python
def write_solution(self, ticks: list[list[str]], end_name: str) -> None:
```

Called after `Pathfinder` finishes. Converts the tick-by-tick drone positions into a human-readable move log and writes it to `solution/<map_name>.txt`.

**Parameters:**

| Parameter  | Type               | Description                                      |
| ---------- | ------------------ | ------------------------------------------------ |
| `ticks`    | `list[list[str]]`  | Each tick is a list of hub names, one per drone  |
| `end_name` | `str`              | Name of the end hub                              |

Uses `self.selected_map` for drone count, hub metadata, and connection IDs.

**Output format:**

Each line of the output file represents one tick where something moved:

```
D1-B D2-C
D1-Z D3-B
```

- `D1-B` — Drone 1 moved to hub B
- `D1-Z` — Drone 1 arrived at the end hub (recorded once, then removed from future ticks)
- In `restricted` zones, the label is the connection ID (`from-to`) rather than the hub name

> The file is written to `solution/<name>.txt` relative to the project root, creating the directory if it doesn't exist.
