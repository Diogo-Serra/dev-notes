---
tags: [projects, fly-in, index]
---

# Fly-in - Index

A Python drone routing system. Given a map of hubs and connections, it routes a fleet of drones from a start hub to an end hub using **Dinic's max-flow algorithm**, then animates the result with Pygame.

---

## Execution Flow

```
fly-in.py
     ↓
bootstrap.initialize(argv)
     ↓
app.run()
     ↓
  ┌──────────────────────────────────────┐
  │  Menu loop                           │
  │  1. Select Map → FileHandler         │
  │                       ↓             │
  │                  handler.read_map_file() → Map
  │  3. Navigate  → Pathfinder(map)      │
  │                       ↓             │
  │               handler.write_solution()   │
  │                       ↓             │
  │               Renderer(map, pf).run()│
  └──────────────────────────────────────┘
```

---

## Notes

| Note                                          | Description                                                            |
| --------------------------------------------- | ---------------------------------------------------------------------- |
| [[Projects/Fly-in/App\|App]]                  | `bootstrap.py` + `app.py` - startup, menu loop, execution routing     |
| [[Projects/Fly-in/Maps\|Maps]]                | `maps.py` - `Drone`, `Hub`, `Connection`, `Map` Pydantic models        |
| [[Projects/Fly-in/Handler\|Handler]]          | `handler.py` - map file parsing and solution file writing              |
| [[Projects/Fly-in/Pathfinder\|Pathfinder]]   | `pathfinder.py` - Dinic max-flow, path extraction, tick simulation     |
| [[Projects/Fly-in/Renderer\|Renderer]]        | `renderer.py` - Pygame window, drawing loop, keyboard controls         |

---

## Project Structure

```
fly-in.py          ← entry point
src/
  bootstrap.py     ← validates argv, launches app, handles errors
  app.py           ← interactive menu loop
  classes/
    maps.py        ← Pydantic data models
    handler.py     ← file I/O (read map, write solution)
    pathfinder.py  ← routing algorithm + simulation
    renderer.py    ← Pygame visualisation
  maps/            ← .txt map files
solution/          ← output .txt solution files (auto-generated)
```

---

## Map File Format

```
# Easy : Linear Path
nb_drones: 3
start_hub: A 0 0 [max_drones=10]
end_hub:   Z 5 0 [max_drones=10]
hub:       B 2 0 [max_drones=2]
connection: A-B [max_link_capacity=2]
connection: B-Z [max_link_capacity=2]
```

| Line prefix   | Meaning                                      |
| ------------- | -------------------------------------------- |
| `# diff:name` | Map difficulty and display name (line 1)     |
| `nb_drones:`  | Number of drones to route                    |
| `start_hub:`  | Source hub - `name x y [metadata...]`        |
| `end_hub:`    | Sink hub - `name x y [metadata...]`          |
| `hub:`        | Intermediate hub                             |
| `connection:` | Bidirectional link - `from-to [metadata...]` |

### Hub metadata keys

| Key          | Effect                                           |
| ------------ | ------------------------------------------------ |
| `max_drones` | Max drones that can occupy this hub at once      |
| `zone`       | `blocked` skips hub, `priority` routes first, `restricted` uses stricter capacity check |

### Connection metadata keys

| Key                 | Effect                              |
| ------------------- | ----------------------------------- |
| `max_link_capacity` | Max drones that can use this link per tick |
