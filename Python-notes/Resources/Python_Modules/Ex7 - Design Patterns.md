# Design Patterns — Factory, Mixin & Strategy

**Module:** DataDeck — Abstract Card Architecture
**Exercise:** Ex0–Ex2 — Creature Factory, Capabilities, Abstract Strategy
**Topic:** Abstract Factory, multiple inheritance / mixins, Strategy pattern, MRO

---

## What is a Design Pattern?

A design pattern is a named, reusable solution to a recurring software design problem. Patterns are not code — they are blueprints. Knowing the name lets you communicate the intent instantly with other developers.

Three patterns appear in this module:

| Pattern | Problem it solves |
|---|---|
| Abstract Factory | Creating families of related objects without specifying concrete classes |
| Mixin | Adding capabilities to a class without deep inheritance chains |
| Strategy | Swapping algorithms at runtime without changing the object that uses them |

---

## Pattern 1: Abstract Factory

### The Problem

You need to create related objects that belong to the same **family**, and you want to swap the entire family without changing the code that uses them.

```
FlameFamily  →  Flameling (base) + Pyrodon (evolved)
AquaFamily   →  Aquabub   (base) + Torragon (evolved)
```

Client code should work with any family without knowing which one it has.

### The Structure

```python
from abc import ABC, abstractmethod

# Abstract product
class Creature(ABC):
    def __init__(self, name: str, type_: str) -> None:
        self.name = name
        self.type_ = type_

    @abstractmethod
    def attack(self) -> str: ...

    def describe(self) -> str:
        return f"{self.name} is a {self.type_} type Creature"

# Concrete products — one family
class Flameling(Creature):
    def attack(self) -> str:
        return f"{self.name} uses Ember!"

class Pyrodon(Creature):
    def attack(self) -> str:
        return f"{self.name} uses Flamethrower!"

# Abstract factory
class CreatureFactory(ABC):
    @abstractmethod
    def create_base(self) -> Creature: ...

    @abstractmethod
    def create_evolved(self) -> Creature: ...

# Concrete factory — one per family
class FlameFactory(CreatureFactory):
    def create_base(self) -> Creature:
        return Flameling("Flameling", "Fire")

    def create_evolved(self) -> Creature:
        return Pyrodon("Pyrodon", "Fire/Flying")
```

### Using It

The function below works with **any** factory — it never names a concrete class:

```python
def test_factory(factory: CreatureFactory) -> None:
    base = factory.create_base()
    evolved = factory.create_evolved()
    print(base.describe())
    print(base.attack())
    print(evolved.describe())
    print(evolved.attack())

test_factory(FlameFactory())   # works
test_factory(AquaFactory())    # works — same function, different family
```

### Key Insight

The factory is the only place that names concrete classes. Everything else depends on the abstract types. Swapping an entire family means swapping one factory object.

---

## Pattern 2: Mixin / Multiple Inheritance

### The Problem

Some creatures can heal. Some can transform. These **capabilities** are independent of the creature family and may apply to many different classes. Putting everything in `Creature` bloats the base class; deep inheritance chains become brittle.

### Mixins — Adding Capabilities Horizontally

A **mixin** is a class designed to be inherited alongside another class. It adds a specific set of methods without being a meaningful standalone object.

```python
# Capability mixin — not a Creature, not a standalone class
class HealCapability(ABC):
    @abstractmethod
    def heal(self) -> str: ...

class TransformCapability(ABC):
    _transformed: bool = False

    @abstractmethod
    def transform(self) -> str: ...

    @abstractmethod
    def revert(self) -> str: ...
```

A creature with healing ability inherits from **both**:

```python
class Sproutling(Creature, HealCapability):
    def attack(self) -> str:
        return f"{self.name} uses Vine Whip!"

    def heal(self) -> str:
        return f"{self.name} heals itself for a small amount"
```

`Sproutling` is a `Creature` **and** has `HealCapability`. Both contracts must be satisfied.

### Method Resolution Order (MRO)

When a class has multiple bases, Python uses the **C3 linearization algorithm** to determine the order in which base classes are searched for a method. You can inspect it:

```python
print(Sproutling.__mro__)
# (<class 'Sproutling'>, <class 'Creature'>, <class 'HealCapability'>, <class 'ABC'>, <class 'object'>)
```

Python searches left to right. If `Sproutling` defines a method, it wins. If not, Python checks `Creature`, then `HealCapability`, then `ABC`, then `object`.

### `isinstance()` with Mixins

Because `Sproutling` inherits from both, it passes `isinstance` checks for both:

```python
s = Sproutling(...)
isinstance(s, Creature)         # True
isinstance(s, HealCapability)   # True
```

This is how the Strategy pattern later decides which capabilities a creature has.

### `super()` in Multiple Inheritance

`super()` follows the MRO — it does not always call the direct parent. This matters when both bases define `__init__`:

```python
class Sproutling(Creature, HealCapability):
    def __init__(self, name: str, type_: str) -> None:
        super().__init__(name, type_)   # calls Creature.__init__ via MRO
```

For mixins that do not define `__init__`, this is usually not an issue — but be aware the MRO determines who `super()` calls.

---

## Pattern 3: Strategy

### The Problem

Different creatures fight differently: some attack and heal, some transform first, some just attack. If you write `if isinstance(creature, HealCapability)` everywhere, the tournament code becomes fragile and hard to extend.

### The Structure

Encapsulate each algorithm in its own class with a common interface:

```python
class BattleStrategy(ABC):
    @abstractmethod
    def is_valid(self, creature: Creature) -> bool: ...

    @abstractmethod
    def act(self, creature: Creature) -> None: ...

class NormalStrategy(BattleStrategy):
    def is_valid(self, creature: Creature) -> bool:
        return True   # works for any creature

    def act(self, creature: Creature) -> None:
        print(creature.attack())

class DefensiveStrategy(BattleStrategy):
    def is_valid(self, creature: Creature) -> bool:
        return isinstance(creature, HealCapability)

    def act(self, creature: Creature) -> None:
        if not self.is_valid(creature):
            raise ValueError(f"Invalid Creature '{creature.name}' for defensive strategy")
        print(creature.attack())
        print(creature.heal())          # type: ignore — guarded by is_valid
```

### Using It

The battle function receives a strategy and delegates to it — it never checks types itself:

```python
def battle(
    opponents: list[tuple[CreatureFactory, BattleStrategy]]
) -> None:
    for i, (fac_a, strat_a) in enumerate(opponents):
        for fac_b, strat_b in opponents[i + 1:]:
            c_a = fac_a.create_base()
            c_b = fac_b.create_base()
            print(c_a.describe())
            print("vs.")
            print(c_b.describe())
            strat_a.act(c_a)
            strat_b.act(c_b)
```

Swapping a strategy means passing a different object — no conditional logic inside `battle`.

### Key Insight

The Strategy pattern separates **what** is done (the creature) from **how** it is done (the strategy). Adding a new strategy never requires touching existing creatures or the tournament code.

---

## How the Three Patterns Work Together

```
CreatureFactory  →  creates Creature objects (Abstract Factory)
Creature         →  mixed with HealCapability / TransformCapability (Mixin)
BattleStrategy   →  decides how a Creature acts in a fight (Strategy)
```

- The **factory** handles object creation — client code never calls `Flameling()` directly
- The **mixin** adds capabilities — `isinstance(c, HealCapability)` is the only type check needed
- The **strategy** handles behaviour — the tournament loop is capability-agnostic

---

## Key Takeaways

- **Abstract Factory** — group related object creation behind a common interface; swap families by swapping the factory
- **Mixin** — add capabilities via multiple inheritance; keep them independent of the core class hierarchy
- **MRO** — Python resolves methods left to right through the inheritance chain; use `ClassName.__mro__` to inspect it
- **Strategy** — encapsulate algorithms in classes with a shared interface; swap behaviour at runtime without if/else chains
- Use `isinstance(obj, CapabilityClass)` to check for mixin capabilities — it is the idiomatic bridge between mixins and strategies

---

## Common Mistakes

```python
# Mixin that defines __init__ without calling super() — breaks the MRO chain
class HealCapability:
    def __init__(self) -> None:
        pass   # does NOT call super() — Creature.__init__ is never reached

# Strategy that checks types internally instead of delegating to is_valid
def act(self, creature: Creature) -> None:
    if isinstance(creature, HealCapability):   # strategy doing type checks = bad smell
        creature.heal()

# Forgetting is_valid before act — raises confusing AttributeError instead of clear message
strategy.act(wrong_creature)   # AttributeError: 'Flameling' object has no attribute 'heal'
# Better: always guard with is_valid first or raise a clear ValueError inside act
```

---

*Tags: #resource/python #python/design-patterns #python/oop #python/multiple-inheritance*

---

**Index:** [[Python Modules Index]]
**Previous:** [[Ex6 - Modules, Packages & Imports]] | **Next:** [[Ex8 - Virtual Environments & Packages]]
**See also:** [[Ex1 - Classes & Objects]], [[Ex5 - Abstract Classes & Polymorphism]]
