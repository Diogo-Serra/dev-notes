# Modules, Packages & Imports

**Module:** The Codex — Mastering Python's Import Mysteries
**Exercise:** Parts I–IV — Alembic, Distillation, Transmutation, Circular Deps
**Topic:** Modules, packages, `__init__.py`, import syntax, absolute vs relative, circular imports

---

## What is a Module?

A module is any `.py` file. When you import it, Python executes the file once and stores the result in `sys.modules`. Every name defined at the top level becomes an attribute of the module object.

```
elements.py          →   module named `elements`
alchemy/elements.py  →   module named `alchemy.elements`
```

---

## What is a Package?

A package is a **directory containing an `__init__.py` file**. The `__init__.py` transforms the folder into an importable namespace and runs when the package is first imported.

```
alchemy/
├── __init__.py       ← makes `alchemy` a package
├── elements.py       ← module `alchemy.elements`
└── transmutation/
    ├── __init__.py   ← makes `alchemy.transmutation` a sub-package
    └── recipes.py    ← module `alchemy.transmutation.recipes`
```

Without `__init__.py`, the directory is just a folder — Python will not treat it as a package (in Python 3, implicit namespace packages exist, but explicit is clearer and more portable).

---

## Import Syntax

### `import module`

Imports the module object. You access its contents via dot notation:

```python
import elements

result = elements.create_fire()
```

### `from module import name`

Imports a specific name directly into the current namespace:

```python
from elements import create_water

result = create_water()   # no prefix needed
```

### `import module as alias`

Gives the module a local name:

```python
import alchemy.elements as ae

ae.create_air()
```

### `from module import name as alias`

```python
from alchemy.potions import healing_potion as heal

heal()
```

---

## Accessing Sub-packages

Use the dotted path to address nested modules:

```python
import alchemy.elements              # access as alchemy.elements.create_earth()
from alchemy.elements import create_air   # access as create_air()
from alchemy.transmutation import recipes  # access as recipes.lead_to_gold()
```

Python resolves the path left-to-right: finds `alchemy/`, then `elements.py` inside it.

---

## `__init__.py` — Controlling the Package Interface

`__init__.py` runs when the package is imported. Use it to:

**Re-export selected names** so users don't need to know the internal structure:

```python
# alchemy/__init__.py
from alchemy.elements import create_air
from alchemy.potions import healing_potion as heal
```

Now users can write:

```python
import alchemy
alchemy.create_air()   # works — re-exported in __init__.py
alchemy.heal()         # works — aliased in __init__.py
alchemy.create_earth() # AttributeError — not re-exported
```

**`__all__`** explicitly declares what `from package import *` exposes:

```python
# alchemy/__init__.py
__all__ = ["create_air", "heal"]
```

---

## How Python Finds Modules — `sys.path`

When you write `import something`, Python searches these locations in order:

1. The directory of the script being run
2. Directories in the `PYTHONPATH` environment variable
3. Standard library paths
4. Site-packages (installed third-party packages)

```python
import sys
print(sys.path)   # inspect the search path
```

This is why `elements.py` in the project root is importable directly — the script's directory is always first on `sys.path`.

---

## Absolute Imports

An absolute import uses the **full dotted path** from the project root:

```python
# inside alchemy/transmutation/recipes.py
from alchemy.elements import create_air          # absolute
from alchemy.potions import strength_potion      # absolute
```

Absolute imports are unambiguous — they always mean the same path regardless of where the importing file lives. **Preferred in most cases.**

---

## Relative Imports

A relative import uses dots to express position relative to the current package:

| Syntax | Meaning |
|---|---|
| `from . import module` | Same package |
| `from .module import name` | Same package |
| `from .. import module` | Parent package |
| `from ..module import name` | Parent package |

```python
# inside alchemy/transmutation/recipes.py
from . import recipes          # same package (alchemy/transmutation/)
from .. import potions         # parent package (alchemy/)
from ..elements import create_air   # parent package, specific name
```

Relative imports only work **inside a package** — you cannot use them in a top-level script (the one you run directly with `python3`).

---

## `__name__` and the Import System

When Python **runs** a file directly, `__name__` is set to `"__main__"`. When it is **imported**, `__name__` is the module's dotted name.

```python
# elements.py
print(__name__)   # prints "elements" when imported, "__main__" when run directly
```

This is why the `if __name__ == "__main__":` guard prevents test code from running on import:

```python
def create_fire() -> str:
    return "Fire element created"

if __name__ == "__main__":
    print(create_fire())   # only runs when this file is executed directly
```

---

## Circular Imports

A circular import occurs when module A imports module B, and module B imports module A (directly or through a chain). Python partially initialises the first module before the second finishes loading, so names may not yet exist when the second module tries to use them.

```
dark_spellbook.py  →  imports dark_validator.py
dark_validator.py  →  imports dark_spell_allowed_ingredients from dark_spellbook.py
                       ↑ not yet fully loaded → ImportError
```

**Error message:**
```
ImportError: cannot import name 'dark_spell_allowed_ingredients' from partially
initialized module 'alchemy.grimoire.dark_spellbook'
```

### Breaking the Cycle

Three common solutions:

**1. Move the shared dependency to a third module**

Extract what both modules need into a separate file neither imports from the other:

```
spellbook.py     →  imports validator.py
validator.py     →  imports constants.py   ← no cycle
spellbook.py     →  imports constants.py   ← no cycle
```

**2. Import inside the function (lazy import)**

Defer the import until it is actually needed — by then, the other module is fully loaded:

```python
def light_spell_record(spell_name: str, ingredients: str) -> str:
    from .light_validator import validate_ingredients   # imported lazily
    result = validate_ingredients(ingredients)
    ...
```

**3. Import the module, not the name**

Import the module object rather than a specific name. The name is looked up at call time, not import time:

```python
from . import light_validator   # ok — doesn't need the name resolved yet

def light_spell_record(spell_name: str, ingredients: str) -> str:
    result = light_validator.validate_ingredients(ingredients)
```

---

## Import Checklist

| Situation | Use |
|---|---|
| Importing from the same package | Relative import (`.`) |
| Importing from a parent or sibling package | Relative (`..`) or absolute |
| Importing from project root or stdlib | Absolute |
| Re-exporting names from a sub-module | `__init__.py` |
| Limiting `from pkg import *` | `__all__` |
| Test code that should not run on import | `if __name__ == "__main__":` |

---

## Key Takeaways

- A module is any `.py` file; a package is a directory with `__init__.py`
- `import x` gives you the module object; `from x import y` brings a name directly into scope
- `__init__.py` controls what the package exposes — re-export selectively to simplify the public API
- Python searches `sys.path` to find modules; the running script's directory is always first
- Absolute imports are unambiguous and preferred; relative imports use `.` and `..` and only work inside packages
- Circular imports fail because Python partially initialises modules — break cycles by extracting shared code, importing lazily, or importing the module instead of the name

---

## Common Mistakes

```python
# Relative import in a top-level script
from .elements import create_fire   # ImportError: attempted relative import with no known parent

# Circular import — importing a name that doesn't exist yet
# a.py: from b import foo
# b.py: from a import bar   ← bar may not be defined yet when b.py loads

# Forgetting __init__.py
# alchemy/ without __init__.py → `import alchemy` may fail or give a namespace package

# Shadowing a stdlib module with a local file
# If you name your file `random.py`, `import random` will find yours, not the stdlib
```

---

*Tags: #resource/python #python/imports #python/packages #python/modules*

---

**Index:** [[Python Modules Index]]
**Previous:** [[Ex5 - Abstract Classes & Polymorphism]] | **Next:** [[Ex7 - Design Patterns]]
**See also:** [[Ex8 - Virtual Environments & Packages]] (packages and dependency management)
