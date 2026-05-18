# Python Collections — Lists, Tuples, Sets & Dictionaries

**Module:** Data Quest — Mastering Python Collections
**Exercises:** Ex0–Ex6 — Command Quest, Score Cruncher, Position Tracker, Achievement Hunter, Inventory Master, Stream Wizard, Data Alchemist
**Topic:** Lists, tuples, sets, dictionaries, generators, comprehensions, `sys.argv`

---

## Collection Overview

| Type | Ordered | Indexed | Mutable | Unique | Key-Value |
|---|---|---|---|---|---|
| `list` | Yes | `int` | Yes | No | No |
| `tuple` | Yes | `int` | No | No | No |
| `set` | No | — | Yes | Yes | No |
| `dict` | Yes (3.7+) | `key` | Yes | Keys only | Yes |

---

## Lists

A list is an **ordered, mutable, indexed** sequence. It allows duplicates and can hold elements of any type.

```python
scores: list[int] = [1500, 2300, 1800, 2100, 1950]
```

### `sys.argv` — lists from the command line

The `sys` module exposes `sys.argv` — a list of strings containing command-line arguments. `sys.argv[0]` is always the script name; the user arguments start at index 1:

```python
import sys

# python3 script.py hello world 42
print(sys.argv)       # ['script.py', 'hello', 'world', '42']
print(sys.argv[0])    # 'script.py'
print(sys.argv[1:])   # ['hello', 'world', '42']  — arguments only
print(len(sys.argv))  # 4  — includes script name
```

The slice `sys.argv[1:]` is the idiomatic way to get just the user-supplied arguments.

### Creating and accessing

```python
items: list[str] = ["sword", "potion", "shield"]

items[0]    # "sword"            — positive index (from front)
items[-1]   # "shield"           — negative index (from back)
items[1:]   # ["potion", "shield"]  — slice from index 1 onward
items[:2]   # ["sword", "potion"]   — slice up to (not including) index 2
```

### Mutating a list

```python
items.append("helmet")           # add one element at the end
items.extend(["bow", "arrow"])   # add all elements from another iterable
items.insert(1, "armor")         # insert at a specific index
items.remove("potion")           # remove first occurrence by value — ValueError if missing
items.pop()                      # remove and return the last element
items.pop(0)                     # remove and return element at index
items.sort()                     # sort in-place (ascending by default)
items.reverse()                  # reverse in-place
```

### Useful built-ins

```python
scores = [1500, 2300, 1800, 2100, 1950]

len(scores)     # 5
sum(scores)     # 9650
max(scores)     # 2300
min(scores)     # 1500
sorted(scores)  # new sorted list — original unchanged
```

---

## Tuples

A tuple is an **ordered, immutable, indexed** sequence. Once created, it cannot be modified. Because they are immutable, tuples are hashable (usable as dictionary keys or set elements) when their contents are also hashable.

```python
position: tuple[float, float, float] = (1.0, 2.5, 3.0)
```

### Creating tuples

```python
point  = (1.0, 2.5, 3.0)    # standard
single = (42,)               # single-element — trailing comma is required
coords = 1.0, 2.5, 3.0      # parentheses are optional (packing)
empty  = ()
```

### Accessing and unpacking

```python
x = point[0]          # index access — same as a list
x, y, z = point       # unpacking — assigns each value to a variable
first, *rest = point  # starred unpacking: first=1.0, rest=[2.5, 3.0]
```

### Why immutability matters

- **Semantics**: a 3D coordinate should not change after creation
- **Safety**: tuples can be used as dictionary keys or stored in sets
- **Function returns**: returning multiple values from a function naturally uses a tuple

```python
def get_range(values: list[int]) -> tuple[int, int]:
    return min(values), max(values)

low, high = get_range([3, 1, 4, 1, 5])
```

### List vs Tuple

| | `list` | `tuple` |
|---|---|---|
| Syntax | `[1, 2, 3]` | `(1, 2, 3)` |
| Mutable | Yes | No |
| Hashable | No | Yes (if contents hashable) |
| Use case | Sequence that grows/changes | Fixed record, coordinate, multi-value return |

---

## Sets

A set is an **unordered collection of unique elements**. Duplicates are silently discarded. Optimised for membership testing (O(1) average) and set mathematics.

```python
achievements: set[str] = {"Boss Slayer", "Speed Runner", "Untouchable"}
```

### Creating a set

```python
skills: set[str] = {"run", "jump", "climb"}

# From a list — deduplicates automatically
raw = ["run", "jump", "run", "climb"]
skills = set(raw)    # {'run', 'jump', 'climb'}

# Empty set — MUST use set(), not {}
empty: set[str] = set()   # {} creates an empty dict!
```

### Adding and removing

```python
achievements.add("Collector")
achievements.discard("Speed Runner")  # safe — no error if missing
achievements.remove("Boss Slayer")    # raises KeyError if missing
```

### Set operations

```python
alice: set[str] = {"Boss Slayer", "Speed Runner", "Untouchable"}
bob:   set[str] = {"Speed Runner", "Collector",   "Untouchable"}

alice.union(bob)           # all elements from both       — alice | bob
alice.intersection(bob)    # elements present in both     — alice & bob
alice.difference(bob)      # in alice but NOT in bob      — alice - bob
bob.difference(alice)      # in bob but NOT in alice      — bob - alice
```

```
alice  = { A, B, C }
bob    = {    B, C, D }

union        = { A, B, C, D }   — everything
intersection = {    B, C    }   — overlap
alice - bob  = { A          }   — exclusive to alice
bob - alice  = {          D }   — exclusive to bob
```

### Multi-set operations

For more than two sets, use `*` unpacking to pass the rest as arguments:

```python
players = [alice, bob, charlie, dylan]

all_achievements = players[0].union(*players[1:])
common           = players[0].intersection(*players[1:])
```

### When to use a set

Use a `set` when you need fast `in` checks, deduplication, or union/intersection/difference operations. Use a `list` when order or duplicates matter.

---

## Dictionaries

A dictionary stores **key-value pairs**. Keys must be unique and hashable; values can be anything. Python 3.7+ preserves insertion order.

```python
inventory: dict[str, int] = {"sword": 1, "potion": 5, "shield": 2}
```

### Creating and accessing

```python
d: dict[str, int] = {"hp": 100, "mp": 50}

d["hp"]             # 100 — KeyError if key is missing
d.get("mp")         # 50
d.get("xp", 0)      # 0 — default value returned instead of KeyError
d["xp"] = 200       # add a new key or update an existing one
```

### Iterating

```python
for key in d:                  # iterate keys
for key in d.keys():           # same, explicit
for value in d.values():       # iterate values
for key, value in d.items():   # iterate (key, value) pairs
```

### Useful methods

```python
d.keys()               # dict_keys view
d.values()             # dict_values view
d.items()              # dict_items view of (key, value) tuples
d.update({"mp": 75})   # merge — overwrites existing keys
d.pop("xp")            # remove and return value — KeyError if missing
d.pop("xp", None)      # safe remove with a default
```

### Parsing structured command-line input

A common pattern from the exercises — parsing `name:quantity` arguments:

```python
inventory: dict[str, int] = {}

for param in sys.argv[1:]:
    if ":" not in param:
        print(f"Error - invalid parameter '{param}'")
        continue
    name, qty_str = param.split(":", 1)
    if name in inventory:
        print(f"Redundant item '{name}' - discarding")
        continue
    try:
        inventory[name] = int(qty_str)
    except ValueError as e:
        print(f"Quantity error for '{name}': {e}")
```

---

## Generators

A generator is a function that **yields values one at a time** instead of building the entire sequence in memory. Python suspends execution at each `yield` and resumes from that point on the next `next()` call.

```python
from typing import Generator
import random

def gen_event() -> Generator[tuple[str, str], None, None]:
    players = ["alice", "bob", "charlie", "dylan"]
    actions = ["run", "jump", "grab", "climb"]
    while True:
        yield (random.choice(players), random.choice(actions))
```

### Using a generator

```python
gen = gen_event()
next(gen)   # ('alice', 'run')
next(gen)   # ('bob', 'climb')
```

Each `next()` call runs the function until the next `yield`, then pauses again.

### Finite generator with `for .. in`

A generator that eventually exhausts works naturally in a `for` loop:

```python
def consume_event(events: list) -> Generator[tuple, None, None]:
    while events:
        idx = random.randint(0, len(events) - 1)
        yield events.pop(idx)

for event in consume_event(my_list):
    print(event)   # prints each event, removes it from the list
```

### Generator type hint

```python
from typing import Generator

# Generator[YieldType, SendType, ReturnType]
# For simple one-way generators: Generator[str, None, None]
```

### List vs Generator

| | `list` | Generator |
|---|---|---|
| Memory | All items stored at once | One item at a time |
| Reusable | Yes | No — exhausted after one pass |
| Random access | Yes | No |
| Use case | Small datasets, index access needed | Large/infinite streams, one-pass processing |

---

## Comprehensions

Comprehensions are a concise, single-line way to build collections from iterables.

### List comprehension

```python
names = ["Alice", "bob", "Charlie", "dylan"]

capitalised  = [name.capitalize() for name in names]
# ['Alice', 'Bob', 'Charlie', 'Dylan']

already_caps = [name for name in names if name[0].isupper()]
# ['Alice', 'Charlie']
```

### Dict comprehension

```python
players = ["Alice", "Bob", "Charlie"]
scores  = {name: random.randint(100, 1000) for name in players}
# {'Alice': 263, 'Bob': 666, 'Charlie': 907}

avg         = sum(scores.values()) / len(scores)
high_scores = {name: s for name, s in scores.items() if s > avg}
```

### Set comprehension

```python
words = ["hello", "world", "hello", "python"]
unique_upper: set[str] = {word.upper() for word in words}
# {'HELLO', 'WORLD', 'PYTHON'}
```

Use a comprehension when the logic fits on one line and you are building a new collection. Use a regular loop when there are side effects or the logic is complex.

---

## Full Comparison Table

| Operation | `list` | `tuple` | `set` | `dict` |
|---|---|---|---|---|
| Create | `[1, 2]` | `(1, 2)` | `{1, 2}` | `{k: v}` |
| Empty literal | `[]` | `()` | `set()` | `{}` |
| Access by position | `a[0]` | `a[0]` | — | `d[key]` |
| Membership test | `x in a` O(n) | `x in a` O(n) | `x in s` O(1) | `k in d` O(1) |
| Add element | `append()` | — | `add()` | `d[k] = v` |
| Remove element | `remove()` / `pop()` | — | `discard()` / `remove()` | `pop()` |
| Mutable | Yes | No | Yes | Yes |
| Allows duplicates | Yes | Yes | No | No (keys) |
| Ordered | Yes | Yes | No | Yes (3.7+) |

---

## Key Takeaways

- **List**: default sequence type — ordered, mutable, indexed by integer, allows duplicates
- **Tuple**: use when data must not change — coordinates, multi-value returns, dict keys
- **Set**: use for membership testing and deduplication — `in` is O(1) vs O(n) for lists
- **Dict**: use for named lookup — keys are unique, values can be anything
- **Generator**: produces values on demand — ideal for large or infinite sequences; exhausted after one pass
- **Comprehensions**: concise collection construction — list, dict, and set forms all exist
- `{}` is always an empty **dict** — use `set()` for an empty set
- `sys.argv[0]` is the script name; `sys.argv[1:]` are the user-supplied arguments

---

## Common Mistakes

```python
# Empty set — {} creates a dict, not a set
empty = {}      # dict
empty = set()   # correct

# Single-element tuple requires a trailing comma
single = (42)    # int — parentheses are just grouping
single = (42,)   # tuple

# Mutating a tuple
point = (1.0, 2.5)
point[0] = 3.0   # TypeError: 'tuple' object does not support item assignment

# KeyError on dict access — use .get() when the key might be absent
d = {"hp": 100}
d["mp"]          # KeyError
d.get("mp", 0)   # 0 — safe

# Indexing a set
s = {1, 2, 3}
s[0]   # TypeError: 'set' object is not subscriptable

# Forgetting that generators are single-use
gen = (x for x in range(5))
list(gen)   # [0, 1, 2, 3, 4]
list(gen)   # []  — already exhausted
```

---

*Tags: #resource/python #python/collections #python/lists #python/tuples #python/sets #python/dicts #python/generators*

---

**Index:** [[Python Modules Index]]
**Previous:** [[Ex2 - Exception Types]] | **Next:** [[Ex4 - Context Managers & with]]
**See also:** [[Ex9 - Pydantic Models & Validation]] (structured typed data)
