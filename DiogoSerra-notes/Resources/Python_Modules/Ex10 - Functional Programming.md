# Functional Programming — Lambdas, Closures & Decorators

**Module:** FuncMage — Master the Ancient Arts of Functional Programming
**Exercise:** Ex0–Ex4 — Lambda Sanctum, Higher Realm, Memory Depths, Ancient Library, Master's Tower
**Topic:** Lambdas, first-class functions, closures, `functools`, decorators, `@staticmethod`

---

## First-Class Functions

In Python, functions are **first-class objects** — they can be assigned to variables, stored in data structures, passed as arguments, and returned from other functions. This is the foundation of functional programming.

```python
def greet(name: str) -> str:
    return f"Hello, {name}"

say_hello = greet          # assign to a variable
print(say_hello("Mage"))   # Hello, Mage

funcs: list = [greet, len, str.upper]   # store in a list
```

---

## Lambda Expressions

A lambda is an **anonymous, single-expression function**. Useful for short, one-time transformations where a full `def` would be verbose.

```python
# Syntax: lambda parameters: expression
double = lambda x: x * 2
add    = lambda x, y: x + y

double(5)   # 10
add(3, 4)   # 7
```

### With `map`, `filter`, `sorted`

```python
numbers = [3, 1, 4, 1, 5, 9, 2]

# map — apply a function to every element
doubled = list(map(lambda x: x * 2, numbers))
# [6, 2, 8, 2, 10, 18, 4]

# filter — keep elements where the function returns True
evens = list(filter(lambda x: x % 2 == 0, numbers))
# [4, 2]

# sorted — custom sort key
artifacts = [{"name": "Staff", "power": 92}, {"name": "Orb", "power": 85}]
sorted_artifacts = sorted(artifacts, key=lambda a: a["power"], reverse=True)
```

### When NOT to use lambda

Use a regular `def` when:
- The logic spans more than one expression
- You need to give it a meaningful name for readability
- It will be reused in multiple places

---

## Higher-Order Functions

A **higher-order function** takes a function as an argument or returns a function.

### Taking a function as an argument

```python
from collections.abc import Callable

def apply(func: Callable[[int], int], value: int) -> int:
    return func(value)

apply(lambda x: x ** 2, 5)   # 25
```

### Returning a function

```python
def power_amplifier(base_spell: Callable, multiplier: int) -> Callable:
    def amplified(target: str, power: int) -> str:
        return base_spell(target, power * multiplier)
    return amplified

mega_fireball = power_amplifier(fireball, 3)
mega_fireball("Dragon", 10)   # fireball called with power=30
```

The inner function `amplified` **captures** `base_spell` and `multiplier` from the enclosing scope — this is a closure (see below).

### `Callable` type hint

Import from `collections.abc` (not `typing` — the `typing.Callable` alias is kept for compatibility but `collections.abc.Callable` is the canonical source in 3.9+):

```python
from collections.abc import Callable

def spell_combiner(s1: Callable, s2: Callable) -> Callable:
    ...
```

`callable(obj)` — built-in that returns `True` if the object can be called:

```python
callable(len)        # True
callable(42)         # False
callable(lambda: 0)  # True
```

---

## Closures & Lexical Scoping

A **closure** is a function that remembers variables from the scope where it was defined, even after that scope has exited.

```python
def mage_counter() -> Callable[[], int]:
    count = 0                  # captured variable

    def increment() -> int:
        nonlocal count         # allows writing to the outer variable
        count += 1
        return count

    return increment

counter_a = mage_counter()
counter_b = mage_counter()

counter_a()   # 1
counter_a()   # 2
counter_b()   # 1  — independent state
```

Each call to `mage_counter()` creates a **new closure** with its own `count`. The closures are independent.

### `nonlocal` vs `global`

| Keyword | Scope affected | Use case |
|---|---|---|
| `nonlocal` | Enclosing function scope | Mutable state in a closure |
| `global` | Module-level scope | Rarely — makes code hard to test and reason about |

`nonlocal` is allowed because it stays within a well-defined function boundary. `global` is forbidden in this module because it creates hidden mutable state accessible from anywhere.

### Closure as private storage

```python
def memory_vault() -> dict[str, Callable]:
    _storage: dict = {}          # private — not accessible from outside

    def store(key: str, value: object) -> None:
        _storage[key] = value

    def recall(key: str) -> object:
        return _storage.get(key, "Memory not found")

    return {"store": store, "recall": recall}

vault = memory_vault()
vault["store"]("secret", 42)
vault["recall"]("secret")   # 42
```

---

## `functools` Module

### `functools.reduce`

Applies a binary function cumulatively to a sequence, reducing it to a single value:

```python
from functools import reduce
from operator import add, mul

numbers = [1, 2, 3, 4, 5]

total   = reduce(add, numbers)       # ((((1+2)+3)+4)+5) = 15
product = reduce(mul, numbers)       # 120
```

The `operator` module provides function equivalents of Python's operators: `operator.add`, `operator.mul`, `operator.lt`, etc. Use them instead of lambdas when possible — they are faster and more readable.

### `functools.partial`

Creates a new callable with some arguments pre-filled:

```python
from functools import partial

def enchant(power: int, element: str, target: str) -> str:
    return f"{element} enchantment ({power} power) on {target}"

fire_enchant  = partial(enchant, 50, "Fire")
ice_enchant   = partial(enchant, 50, "Ice")

fire_enchant("Sword")   # "Fire enchantment (50 power) on Sword"
```

`partial` returns a new function — the original is unchanged.

### `functools.lru_cache`

Memoises a function — caches return values keyed by arguments. Subsequent calls with the same arguments skip computation entirely:

```python
from functools import lru_cache

@lru_cache(maxsize=None)   # unlimited cache
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

fibonacci(35)               # fast
fibonacci.cache_info()      # CacheInfo(hits=..., misses=..., maxsize=None, currsize=...)
fibonacci.cache_clear()     # reset the cache
```

Without caching, recursive fibonacci is O(2ⁿ). With `lru_cache`, it is O(n).

### `functools.singledispatch`

Dispatches to different implementations based on the **type** of the first argument:

```python
from functools import singledispatch

@singledispatch
def cast_spell(arg: object) -> str:
    return "Unknown spell type"

@cast_spell.register(int)
def _(arg: int) -> str:
    return f"Damage spell: {arg} damage"

@cast_spell.register(str)
def _(arg: str) -> str:
    return f"Enchantment: {arg}"

@cast_spell.register(list)
def _(arg: list) -> str:
    return f"Multi-cast: {len(arg)} spells"

cast_spell(42)            # "Damage spell: 42 damage"
cast_spell("fireball")    # "Enchantment: fireball"
cast_spell([1, 2, 3])     # "Multi-cast: 3 spells"
```

---

## Decorators

A decorator is a function that **wraps another function**, adding behaviour before or after it runs — without modifying the original.

### Basic decorator

```python
from collections.abc import Callable
from functools import wraps

def spell_timer(func: Callable) -> Callable:
    @wraps(func)               # preserves __name__, __doc__ of the original
    def wrapper(*args, **kwargs):
        import time
        print(f"Casting {func.__name__}...")
        start = time.time()
        result = func(*args, **kwargs)
        print(f"Spell completed in {time.time() - start:.3f} seconds")
        return result
    return wrapper

@spell_timer
def fireball(target: str) -> str:
    return f"Fireball hits {target}!"
```

`@spell_timer` above `fireball` is syntactic sugar for `fireball = spell_timer(fireball)`.

### Why `@functools.wraps`?

Without `wraps`, the wrapper function replaces the original's metadata:

```python
print(fireball.__name__)   # "wrapper" without @wraps
print(fireball.__name__)   # "fireball" with @wraps
```

Always use `@wraps(func)` inside your decorator to preserve the original function's name, docstring, and signature.

### Parameterised decorator (decorator factory)

A decorator that takes arguments needs an extra layer — a factory that returns the actual decorator:

```python
def power_validator(min_power: int) -> Callable:
    def decorator(func: Callable) -> Callable:
        @wraps(func)
        def wrapper(power: int, *args, **kwargs):
            if power < min_power:
                return "Insufficient power for this spell"
            return func(power, *args, **kwargs)
        return wrapper
    return decorator

@power_validator(min_power=10)
def lightning(power: int, target: str) -> str:
    return f"Lightning hits {target} for {power}!"
```

Three levels: factory → decorator → wrapper.

### Retry decorator

```python
def retry_spell(max_attempts: int) -> Callable:
    def decorator(func: Callable) -> Callable:
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except Exception:
                    print(f"Spell failed, retrying... (attempt {attempt}/{max_attempts})")
            return f"Spell casting failed after {max_attempts} attempts"
        return wrapper
    return decorator
```

---

## `@staticmethod`

A static method belongs to the class namespace but receives neither `self` nor `cls`. It is essentially a plain function that lives inside a class for organisational reasons:

```python
class MageGuild:
    @staticmethod
    def validate_mage_name(name: str) -> bool:
        return len(name) >= 3 and all(c.isalpha() or c == " " for c in name)

    def cast_spell(self, spell_name: str, power: int) -> str:
        return f"Successfully cast {spell_name} with {power} power"

MageGuild.validate_mage_name("Gandalf")   # True — no instance needed
```

| Method type | First parameter | Receives |
|---|---|---|
| Instance method | `self` | The instance |
| Class method (`@classmethod`) | `cls` | The class itself |
| Static method (`@staticmethod`) | — | Nothing extra |

---

## Key Takeaways

- Functions are first-class — assign, pass, and return them like any value
- Lambdas are anonymous single-expression functions — use them with `map`, `filter`, `sorted`; use `def` for anything more complex
- Higher-order functions take or return functions — the basis of composition and reuse
- Closures capture variables from their enclosing scope; `nonlocal` allows mutation of them
- `functools.reduce` folds a sequence; `partial` pre-fills arguments; `lru_cache` memoises; `singledispatch` dispatches by type
- A decorator is `original = decorator(original)` in disguise — always use `@wraps(func)` inside
- Parameterised decorators add one more layer: factory → decorator → wrapper

---

## Common Mistakes

```python
# Lambda with side effects — use def instead
process = lambda x: (print(x), x * 2)   # awkward tuple hack

# Forgetting @wraps — loses function metadata
def timer(func):
    def wrapper(*args, **kwargs):   # missing @wraps(func)
        return func(*args, **kwargs)
    return wrapper

# nonlocal missing — closes over variable but can't rebind it
def counter():
    count = 0
    def inc():
        count += 1   # UnboundLocalError — Python sees assignment, treats as local
        return count
    return inc
# Fix: add `nonlocal count` inside inc

# Calling a decorator factory without arguments
@power_validator        # wrong — this passes the function as min_power
def my_spell(): ...

@power_validator(10)    # correct — factory called first, returns decorator
def my_spell(): ...
```

---

*Tags: #resource/python #python/functional #python/closures #python/decorators #python/functools*

---

**Index:** [[Python Modules Index]]
**Previous:** [[Ex9 - Pydantic Models & Validation]]
**See also:** [[Ex0 - Defining Functions & print]] (functions are the foundation)
