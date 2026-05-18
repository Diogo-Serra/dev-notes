---
tags: [resource/python, index]
---

# Python Modules — Index

Reference notes from school Python modules, ordered by curriculum sequence.

---

## Notes

| # | Note | Key Topics |
|---|---|---|
| 0 | [[Ex0 - Defining Functions & print]] | `def`, `print()`, indentation, naming |
| 1 | [[Ex1 - Classes & Objects]] | `class`, instances, `self`, attributes, methods |
| 2 | [[Ex2 - Exception Types]] | `try/except`, `ValueError`, `TypeError`, `FileNotFoundError` |
| 3 | [[Ex3 - Python Collections]] | lists, tuples, sets, dicts, generators, comprehensions |
| 4 | [[Ex4 - Context Managers & with]] | `with`, `__enter__`/`__exit__`, file modes |
| 5 | [[Ex5 - Abstract Classes & Polymorphism]] | `ABC`, `@abstractmethod`, polymorphism, `Protocol` |
| 6 | [[Ex6 - Modules, Packages & Imports]] | `import`, packages, `__init__.py`, `sys.path`, circular imports |
| 7 | [[Ex7 - Design Patterns]] | Abstract Factory, Mixin, MRO, Strategy pattern |
| 8 | [[Ex8 - Virtual Environments & Packages]] | `venv`, `pip`, Poetry, `.env`, `python-dotenv` |
| 9 | [[Ex9 - Pydantic Models & Validation]] | `BaseModel`, `Field`, validators, `ValidationError` |
| 10 | [[Ex10 - Functional Programming]] | lambdas, closures, `functools`, decorators |

---

## Concept Map

```
Ex0 (functions)
  └── Ex1 (classes)
        └── Ex5 (abstract classes / polymorphism)
              └── Ex7 (design patterns — ABC + Mixin + Strategy)

Ex1 (classes)
  └── Ex9 (Pydantic — typed models)

Ex2 (exceptions)
  └── Ex4 (context managers — exception-safe resource handling)

Ex3 (collections — lists, dicts)
  └── Ex9 (Pydantic — structured data)

Ex0 (functions)
  └── Ex10 (functional programming — lambdas, HOF, decorators)

Ex6 (imports / packages)
  └── Ex8 (venv + dependency management)
```
