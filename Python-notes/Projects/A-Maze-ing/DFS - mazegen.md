## Part 1 - The Grid and Wall Encoding

The first thing to understand is how a maze is represented in memory. Each cell in the grid stores a **bit mask** - a single number where individual bits represent whether a wall exists in each direction.

Here's a visual breakdown of the bit mask system:

![[bitwise_walls.png]]

> **Key insight:** `0xF` (binary `1111`) means all four walls are present. To carve a passage, you clear a bit using `&= ~direction`. When you carve north from cell A, you must also carve south from the neighbor B - that's what `OPPOSITE` is for.

---

## Part 2 - The DFS Algorithm (Depth-First Search)

This is the heart of the generator. DFS produces mazes with long, winding corridors and few dead-ends - it feels like a proper labyrinth.

![[dfs_maze_stepper.html]]

---

## Part 3 - Fixed Cells and the `_compute_fixed_cells` Pattern

Before DFS runs, your code computes **"fixed cells"** - cells that DFS will never carve through (they're pre-added to `visited`). These form a decorative `42` pattern in the middle of large mazes. The logic is:

1. Compute where the `42` digit shapes would appear in a grid of at least `9×7`.
2. Add those `(x, y)` positions to `visited` before DFS starts.
3. DFS naturally routes around them, leaving them as solid walled-off islands.

> **Clever trick:** the algorithm doesn't need to know about the pattern at all. It just sees "already visited" cells and avoids them.

---

## Part 4 - Writing Hexadecimal to an Output File

This is where the maze gets saved. Each cell's integer value is written as a hex digit:

```python
def save(self) -> None:
    with open(self.OUTPUT_PATH, 'w') as f:
        # First line: dimensions
        f.write(f"{self.WIDTH} {self.HEIGHT}\n")

        # Each row: space-separated hex values
        for row in self.grid:
            hex_values = [format(cell, 'X') for cell in row]
            # format(cell, 'X') → uppercase hex, e.g. 0xD → "D"
            f.write(' '.join(hex_values) + '\n')
```

A 4×4 maze output might look like this:

```
4 4
D 9 9 3
6 8 3 6
5 A 9 5
3 6 6 C
```

Each character is a single hex digit (`0`-`F`) representing the wall state of that cell. A renderer reads this back and draws walls wherever the bits are set.

---

## Part 5 - Reading the Hex File Back (for Rendering)

```python
def load(path: str):
    with open(path) as f:
        width, height = map(int, f.readline().split())
        grid = []
        for line in f:
            row = [int(h, 16) for h in line.split()]
            # int('D', 16) → 13 → 0b1101 → W+S+N walls present
            grid.append(row)
    return width, height, grid
```

`int('D', 16)` converts the hex string `"D"` back to the integer `13`, which is `0b1101` — walls on NORTH, SOUTH, and WEST, but not EAST.

---

## Full Mental Model

```
Grid initialisation:   every cell = 0xF  (all 4 walls set)
                              ↓
Fixed cells computed:  positions of '42' pattern → pre-visited
                              ↓
DFS carving loop:      pick random unvisited neighbour
                       → clear direction bit from current cell
                       → clear opposite bit from neighbour
                       → push neighbour, mark visited
                       → if stuck, pop (backtrack)
                              ↓
Output:                write WIDTH HEIGHT, then grid rows as hex digits
```


## DFS Maze Generator

Each cell is an int where bits 0-3 represent walls:
    N=bit0,
    E=bit1,
    S=bit2,
    W=bit3.

All walls start up (0xF). The algorithm carves paths by clearing shared
walls between adjacent cells, using a stack to backtrack when no unvisited
neighbours remain.

(bit, dx, dy, mirror_bit)
DIRECTIONS = [
    (0,  0, -1, 2),  # North
    (1, +1,  0, 3),  # East
    (2,  0, +1, 0),  # South
    (3, -1,  0, 1),  # West
]

```python
def dfs_generator(grid: list[list[int]]) -> list[list[int]]:
    import random

    width = len(grid)
    height = len(grid[0])
    visited: set[tuple[int, int]] = {(0, 0)}
    stack: list[tuple[int, int]] = [(0, 0)]

    while stack:
        x, y = stack[-1]

        neighbors = []
        for bit, dx, dy, mirror_bit in DIRECTIONS:
            nx, ny = x + dx, y + dy
            if (0 <= nx < width
                    and 0 <= ny < height
                    and (nx, ny) not in visited):
                neighbors.append((nx, ny, bit, mirror_bit))

        if neighbors:
            nx, ny, bit, mirror_bit = random.choice(neighbors)
            grid[x][y] &= ~(1 << bit)
            grid[nx][ny] &= ~(1 << mirror_bit)
            visited.add((nx, ny))
            stack.append((nx, ny))
        else:
            stack.pop()

    return grid
```
