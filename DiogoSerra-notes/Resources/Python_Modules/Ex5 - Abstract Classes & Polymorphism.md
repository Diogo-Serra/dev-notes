# Abstract Classes & Polymorphism

**Module:** Code Nexus — Polymorphic Data Streams in the Digital Matrix
**Exercise:** Ex0–Ex2 — Data Processor, Data Stream, Data Pipeline
**Topic:** ABC, abstract methods, method overriding, polymorphism, `Protocol`

---

## The Problem Abstract Classes Solve

When multiple classes share a common interface, you want to enforce that every subclass implements the required methods. Without enforcement, a missing method only fails at runtime — often at the worst possible time.

```python
class DataProcessor:
    def ingest(self, data):
        pass   # subclass can forget to override — no error until called
```

Abstract Base Classes (ABCs) turn that runtime surprise into an immediate error at instantiation.

---

## Abstract Base Class with `abc`

Import `ABC` and `abstractmethod` from the `abc` module:

```python
from abc import ABC, abstractmethod
from typing import Any

class DataProcessor(ABC):

    @abstractmethod
    def validate(self, data: Any) -> bool:
        ...

    @abstractmethod
    def ingest(self, data: Any) -> None:
        ...

    def output(self) -> tuple[int, str]:
        # concrete method — shared by all subclasses
        ...
```

Key rules:
- A class becomes abstract by inheriting from `ABC`
- Methods decorated with `@abstractmethod` **must** be overridden in every concrete subclass
- You **cannot instantiate an abstract class directly** — Python raises `TypeError`
- Concrete methods (without `@abstractmethod`) are inherited normally

```python
dp = DataProcessor()   # TypeError: Can't instantiate abstract class
```

---

## Implementing an Abstract Class

A concrete subclass must override every abstract method, or it remains abstract itself:

```python
class NumericProcessor(DataProcessor):

    def __init__(self) -> None:
        self._store: list[str] = []

    def validate(self, data: Any) -> bool:
        if isinstance(data, (int, float)):
            return True
        if isinstance(data, list):
            return all(isinstance(item, (int, float)) for item in data)
        return False

    def ingest(self, data: int | float | list[int | float]) -> None:
        if not self.validate(data):
            raise ValueError("Improper numeric data")
        items = data if isinstance(data, list) else [data]
        for item in items:
            self._store.append(str(item))
```

If `validate` or `ingest` were missing, `NumericProcessor()` would raise `TypeError`.

---

## Method Overriding

Overriding means providing a new implementation of an inherited method in a subclass. The method name must match exactly.

```python
class TextProcessor(DataProcessor):

    def validate(self, data: Any) -> bool:      # overrides DataProcessor.validate
        if isinstance(data, str):
            return True
        if isinstance(data, list):
            return all(isinstance(item, str) for item in data)
        return False

    def ingest(self, data: str | list[str]) -> None:   # overrides DataProcessor.ingest
        if not self.validate(data):
            raise ValueError("Improper text data")
        ...
```

The overriding method can have a **more specific signature** for the parameter types, as long as the abstract method's contract is fulfilled.

---

## Polymorphism

Polymorphism means writing code that works with objects of **different types** through a **common interface**. You do not need to know the specific type — just that it implements the interface.

```python
processors: list[DataProcessor] = [
    NumericProcessor(),
    TextProcessor(),
    LogProcessor(),
]

data = 42

for proc in processors:
    if proc.validate(data):        # same call on every type
        proc.ingest(data)          # same call on every type
        break
```

The loop does not care whether it is talking to a `NumericProcessor` or a `TextProcessor`. Each object responds to `validate()` and `ingest()` according to its own implementation.

---

## `isinstance()` for Type Dispatch

When routing data to the right processor, `isinstance()` checks whether an object is an instance of a given type (or a tuple of types):

```python
isinstance(42, int)              # True
isinstance(42, (int, float))     # True — checks against multiple types
isinstance([1, 2], list)         # True
isinstance("hello", int)         # False
```

Used inside `validate()` to check input without crashing:

```python
def validate(self, data: Any) -> bool:
    return isinstance(data, (int, float))
```

---

## `typing.Any`

`Any` is a special type hint that disables type checking for that value. Use it when a method must accept genuinely unknown types (like `validate`, which tests anything):

```python
from typing import Any

def validate(self, data: Any) -> bool:   # accepts literally anything
    return isinstance(data, int)
```

Do not use `Any` to avoid writing proper types — reserve it for genuinely dynamic inputs.

---

## Duck Typing with `Protocol`

ABC enforces a contract through **inheritance** — subclasses must explicitly inherit from the ABC. `Protocol` enforces a contract through **structure** — any class that has the right methods qualifies, even without inheriting.

```python
from typing import Protocol

class ExportPlugin(Protocol):
    def process_output(self, data: list[tuple[int, str]]) -> None:
        ...
```

Any class that implements `process_output` with this signature is automatically compatible — no explicit inheritance needed. This is called **structural subtyping** (or duck typing):

```python
class CSVExporter:                        # does NOT inherit ExportPlugin
    def process_output(self, data: list[tuple[int, str]]) -> None:
        for rank, value in data:
            print(f"CSV: {value}")

class JSONExporter:                       # also does NOT inherit ExportPlugin
    def process_output(self, data: list[tuple[int, str]]) -> None:
        ...
```

Both satisfy the `ExportPlugin` protocol. `mypy` verifies the match statically.

---

## ABC vs Protocol

| | `ABC` | `Protocol` |
|---|---|---|
| Enforcement | Explicit inheritance required | Structural — any matching class qualifies |
| Instantiation | Cannot instantiate abstract class | Protocol itself should not be instantiated |
| Use case | Shared behaviour + enforced interface | Interface only, no shared code |
| Fails at | Instantiation time | `mypy` static check |

Use `ABC` when subclasses share concrete behaviour. Use `Protocol` when you only care that an object has certain methods, regardless of its class hierarchy.

---

## Abstract vs Concrete Methods in an ABC

An ABC can mix both:

```python
class DataProcessor(ABC):

    @abstractmethod
    def validate(self, data: Any) -> bool:   # must override
        ...

    @abstractmethod
    def ingest(self, data: Any) -> None:     # must override
        ...

    def output(self) -> tuple[int, str]:     # shared — no override needed
        rank = self._rank
        value = self._store.pop(0)
        self._rank += 1
        return (rank, value)
```

Subclasses inherit `output()` for free and only implement what is unique to them.

---

## Key Takeaways

- `ABC` + `@abstractmethod` force subclasses to implement required methods — missing ones cause `TypeError` at instantiation
- Concrete methods in an ABC are inherited and shared — avoid duplicating code across subclasses
- Polymorphism lets you call `validate()` and `ingest()` on any `DataProcessor` without knowing its specific type
- `isinstance(data, (int, float))` is the idiomatic way to check multiple types at once
- `typing.Any` disables type checking for a value — use only when the type is genuinely unknown
- `Protocol` provides structural duck typing — no inheritance required, just matching method signatures

---

## Common Mistakes

```python
# Forgetting to override all abstract methods
class BrokenProcessor(DataProcessor):
    def validate(self, data: Any) -> bool:
        return True
    # ingest not implemented — TypeError on instantiation

# Instantiating the abstract class directly
dp = DataProcessor()   # TypeError

# Using Any everywhere to avoid writing types
def ingest(self, data: Any) -> None:   # fine here — genuinely unknown input
    ...
def calculate(x: Any, y: Any) -> Any:  # bad — you know these are numbers
    return x + y
```

---

*Tags: #resource/python #python/oop #python/abc #python/polymorphism*

---

**Index:** [[Python Modules Index]]
**Previous:** [[Ex4 - Context Managers & with]] | **Next:** [[Ex6 - Modules, Packages & Imports]]
**See also:** [[Ex1 - Classes & Objects]], [[Ex7 - Design Patterns]] (ABC used in Abstract Factory)
