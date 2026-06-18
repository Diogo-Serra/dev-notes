## Overview

`pathfinder.py` is the algorithmic core of Fly-in. Given a `Map`, it figures out the fastest way to move all drones from start to end.

It runs in three stages:

```
_solve()
   ↓
1. Build a flow graph + run Dinic max-flow
   → finds how much flow (drones) can pass through each edge
   ↓
2. _extract_paths()
   → converts raw flow amounts into explicit drone routes
   ↓
3. _simulate()
   → plays the routes out tick by tick, respecting all constraints
```

After `__init__` completes, the `Pathfinder` exposes:

| Attribute      | Type               | Description                                       |
| -------------- | ------------------ | ------------------------------------------------- |
| `ticks`        | `list[list[str]]`  | Per-tick hub position of every drone              |
| `total_ticks`  | `int`              | Total number of ticks                             |
| `flow_reached` | `int`              | Number of drones that can reach the end           |
| `end_name`     | `str`              | Name of the end hub                               |

---

## Stage 1 — Dinic Max-Flow

### Why max-flow?

The map is a **capacity-constrained graph**. Each hub can hold a limited number of drones, and each connection can carry a limited number per tick. Max-flow finds the maximum number of drones that can simultaneously move from start to end.

![[dinic_algo_explainer.svg|697]]

### Node splitting

To enforce per-hub capacity limits (not just per-edge), each hub `i` is split into two nodes:

```
hub i (in-node)  →  hub i+N (out-node)   capacity = hub.max_drones
```

Connection edges then go from the **out-node** of one hub to the **in-node** of the next:

```
hub_a (out)  →  hub_b (in)   capacity = connection.max_link_capacity
```

This makes the hub itself a bottleneck, not just the connections.

> `N = num_hubs`. So node `i` is the in-node, node `N+i` is the out-node.

### `_add_edge(from_node, to_node, capacity)`

```python
def _add_edge(self, from_node, to_node, capacity):
    self._graph[from_node].append([to_node, capacity, len(self._graph[to_node])])
    self._graph[to_node].append([from_node, 0, len(self._graph[from_node]) - 1])
```

Adds a directed edge and its **reverse edge** (with capacity 0). The reverse edge is essential to Dinic — it allows the algorithm to "undo" flow decisions.

Each edge is stored as `[neighbor, remaining_capacity, reverse_edge_index]`.

### `_bfs(source, sink)`

```python
def _bfs(self, source, sink) -> bool:
```

Builds a **level graph** — assigns each node a distance (level) from the source using BFS. Only edges with remaining capacity are followed.

Returns `True` if the sink is reachable (i.e. more flow can still be pushed). The levels are stored in `self._level`.

### `_dfs(node, sink, flow_limit)`

```python
def _dfs(self, node, sink, flow_limit) -> int:
```

Pushes as much flow as possible from `node` to `sink` along the level graph. Only follows edges where `level[neighbor] == level[node] + 1` (forward-only in the level graph).

Uses `self._next_edge` to avoid re-scanning edges that have already been exhausted (the "blocking flow" optimisation that makes Dinic efficient).

Returns the amount of flow actually sent. Updates both the forward edge capacity and the reverse edge capacity.

### `_dinic(source, sink, limit)`

```python
def _dinic(self, source, sink, limit) -> int:
```

The outer Dinic loop:

```
while flow < limit and BFS finds a path:
    reset next_edge pointers
    while DFS pushes flow:
        total_flow += flow_sent
```

Each BFS call builds a new level graph. Each DFS call pushes one unit of blocking flow along it. Repeats until no more paths exist or the limit (nb_drones) is reached.

---

## Stage 2 — `_extract_paths(used_flow, source_idx, sink_idx, hub_names)`

After Dinic runs, the residual graph encodes **how much flow passed through each edge**. This stage reads that back and reconstructs individual drone paths.

```python
# How to read used flow from the residual graph:
# For each out-node edge to an in-node:
#   reverse_cap = how much flow was sent forward
```

Then a recursive DFS traces one path at a time from source to sink, decrementing `remaining_flow` as it goes. Priority hubs are visited first (sorted by `self._priority_indices`).

Returns `list[list[str]]` — one list of hub names per path found.

> Uses backtracking — if a dead end is hit, flow is restored and another branch is tried.

---

## Stage 3 — `_simulate(paths, start_name, end_name)`

Takes the abstract paths and plays them out tick by tick, respecting real-time constraints that the flow graph doesn't model.

### Setup

```python
drone_paths = [paths[i % len(paths)] for i in range(num_drones)]
```

Distributes paths across drones using modulo — if there are more drones than paths, paths are reused.

### Per-tick loop

Each tick:

1. **Resolve in-transit drones** — any drone flagged `in_transit` advances one step and joins `just_arrived`.
2. **Count hub occupancy** — builds a snapshot of how many drones are at each hub.
3. **Try to move each drone** — sorted by how far they are from the end (furthest first, to avoid blocking).

### Two movement modes

| Zone         | Capacity check                              | Move type     |
| ------------ | ------------------------------------------- | ------------- |
| Normal       | `hub_occupancy + 1 <= hub_cap`              | Instant move  |
| `restricted` | `hub_occupancy + reserved < hub_cap`        | In-transit (takes 2 ticks) |

> `restricted` hubs use a **reservation system** — the drone leaves its current hub and enters a "in-transit" state for one tick before arriving. This prevents multiple drones committing to the same restricted hub simultaneously.

### Safety limit

```python
MAX_TICKS = (max(len(p) for p in drone_paths) + num_drones) * (num_drones + 4)
```

A generous upper bound to prevent infinite loops in edge cases.

### Output

Sets `self.ticks` and `self.total_ticks` once the loop ends.
