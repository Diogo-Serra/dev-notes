## Part 1 - Why BFS for Solving?

DFS generates the maze. BFS **solves** it.

BFS explores cells layer by layer, always visiting the closest unvisited cells first. This guarantees the **shortest path** from entry to exit - no other algorithm is needed.

```
Start → all cells 1 step away → all cells 2 steps away → ... → Exit
```

> **Key insight:** BFS finds the shortest path because it never reaches a cell via a longer route first. The moment it reaches the exit, the path is already optimal.

---

## Part 2 - The Queue and Parent Map

Two structures drive the whole algorithm:

```python
queue: list[tuple[int, int]] = [(sx, sy)]   # cells to visit
parent: dict[...] = {(sx, sy): None}        # how we got to each cell
```

`parent` does two things at once:
1. Tracks which cells have already been visited (if a cell is a key, it's been seen)
2. Records the path backwards - every cell knows which cell it came from

```python
head = 0  # index into queue — avoids expensive pop(0) on a list
```

Instead of `queue.pop(0)` (O(n)), a `head` pointer advances through the list. Same behavior, no cost.

---

## Part 3 - The BFS Loop

```python
while head < len(queue):
    x, y = queue[head]
    head += 1

    if (x, y) == (gx, gy):
        break                          # reached exit, stop early

    cell = self.grid[y][x]
    for wall_bit, dx, dy in DIRECTIONS:
        if (cell & wall_bit) == 0:     # wall is open (passage exists)
            nx, ny = x + dx, y + dy
            if ok(nx, ny) and (nx, ny) not in parent:
                parent[(nx, ny)] = (x, y)   # record where we came from
                queue.append((nx, ny))
```

A wall is open when its bit is **not set** - `(cell & wall_bit) == 0`. If the neighbor is in-bounds, not blocked, and not yet visited, it gets added to the queue with its parent recorded.

---

## Part 4 - Reconstructing the Path

BFS finds the exit but records the journey backwards. To get the actual path, walk the `parent` map from exit back to entry, then reverse:

```python
path: list[tuple[int, int]] = []
cur = (gx, gy)
while cur is not None:
    path.append(cur)
    cur = parent[cur]   # step backwards
path.reverse()          # entry → exit order
```

If `(gx, gy)` is never added to `parent`, the exit was unreachable - `solve()` returns `None`.

---

## Part 5 - Converting the Path to Directions

`path_to_dirs()` converts the list of coordinates into a `NESW` string for the output file:

```python
for (x1, y1), (x2, y2) in zip(self.solution, self.solution[1:]):
    dx, dy = x2 - x1, y2 - y1
    # dy == -1 → moved up   → North
    # dx ==  1 → moved right → East
    # dy ==  1 → moved down  → South
    # dx == -1 → moved left  → West
```

Each consecutive pair of cells produces one letter. A 5-cell path produces a 4-character string.
