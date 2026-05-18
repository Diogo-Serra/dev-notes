# Exception Types & Multi-Catch

**Module:** Garden Guardian — Data Engineering for Smart Agriculture
**Exercise:** Ex2 — Different Types of Problems
**Topic:** Built-in exceptions, catching specific types, multi-catch, `open()`

---

## What is an Exception?

When Python encounters an error at runtime, it **raises** an exception — an object that describes what went wrong. If nothing catches it, the program crashes with a traceback.

```
ValueError: invalid literal for int() with base 10: 'abc'
^^^^^^^^^   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
type        message
```

Python has a rich set of built-in exception types. Using the right one makes error messages meaningful and lets callers handle specific failures precisely.

---

## Common Built-in Exception Types

### `ValueError`
Raised when a function receives an argument of the correct type but an **invalid value**.

```python
int("abc")       # ValueError: invalid literal for int() with base 10: 'abc'
int("12.5")      # ValueError: invalid literal for int() with base 10: '12.5'
```

### `TypeError`
Raised when an operation is applied to an object of an **incompatible type**.

```python
"hello" + 42     # TypeError: can only concatenate str (not "int") to str
len(123)         # TypeError: object of type 'int' has no len()
```

### `ZeroDivisionError`
Raised when dividing by zero.

```python
10 / 0           # ZeroDivisionError: division by zero
10 // 0          # ZeroDivisionError: integer division or modulo by zero
```

### `FileNotFoundError`
Raised when trying to open a file that does not exist.

```python
open("/non/existent/file")   # FileNotFoundError: [Errno 2] No such file...
```

### Other Common Types (for reference)

| Exception | Triggered by |
|---|---|
| `IndexError` | Accessing a list index out of range |
| `KeyError` | Accessing a dict key that does not exist |
| `AttributeError` | Accessing an attribute that does not exist on an object |
| `NameError` | Using a variable name that has not been defined |
| `ImportError` | Failing to import a module |

---

## Catching a Specific Exception

Use `except ExceptionType:` to handle only that kind of error:

```python
try:
    value = int("abc")
except ValueError:
    print("Bad input — could not convert to int")
```

Only `ValueError` is caught. Any other exception type would still propagate up and crash the program.

---

## Catching Multiple Types Separately

You can chain multiple `except` blocks — each handles a different exception type:

```python
try:
    result = 10 / int(user_input)
except ValueError:
    print("Input was not a valid number")
except ZeroDivisionError:
    print("Cannot divide by zero")
```

Python checks the `except` blocks in order and runs the **first** one that matches. Only one block executes per exception.

---

## Catching Multiple Types Together

To handle several exception types with the same code, group them in a tuple:

```python
try:
    result = 10 / int(user_input)
except (ValueError, ZeroDivisionError):
    print("Invalid input or division by zero")
```

This is equivalent to two separate blocks but avoids repetition when the response is identical.

---

## Exception Order Matters

More specific exceptions must come **before** more general ones. The base class `Exception` matches everything — if placed first, the specific blocks below it never run.

```python
# Wrong — Exception catches everything, specific blocks are unreachable
try:
    int("abc")
except Exception:
    print("Some error")
except ValueError:          # never reached
    print("Bad value")

# Correct — specific first, general last
try:
    int("abc")
except ValueError:
    print("Bad value")
except Exception:
    print("Unexpected error")
```

---

## The Exception Hierarchy

Python exceptions form an inheritance tree. Catching a parent type also catches all its children:

```
BaseException
└── Exception
    ├── ValueError
    ├── TypeError
    ├── ArithmeticError
    │   └── ZeroDivisionError
    ├── OSError
    │   └── FileNotFoundError
    ├── LookupError
    │   ├── IndexError
    │   └── KeyError
    └── AttributeError
```

Catching `OSError` catches `FileNotFoundError`. Catching `Exception` catches almost everything (except system exits and keyboard interrupts).

---

## Opening Files with `open()`

`open(path)` returns a file object. If the path does not exist, it raises `FileNotFoundError`:

```python
try:
    f = open("/data/sensors.csv")
    content = f.read()
    f.close()
except FileNotFoundError:
    print("Sensor data file not found")
```

Always close the file when done, or use a `with` block (covered later) which handles it automatically.

---

## Accessing Exception Details

Bind the exception to a name with `as` to read its message:

```python
try:
    int("abc")
except ValueError as e:
    print(f"Caught ValueError: {e}")
    # Output: Caught ValueError: invalid literal for int() with base 10: 'abc'
```

---

## Key Takeaways

- Python has a hierarchy of built-in exception types — use the most specific one available
- `except TypeError:` catches only that type; `except (TypeError, ValueError):` catches both
- List specific exceptions before general ones — order matters
- Catching `Exception` is a broad safety net; prefer specific types when you know what to expect
- Use `as e` to capture and display the exception message

---

## Common Mistakes

```python
# Catching too broadly — hides real bugs
try:
    do_something()
except Exception:
    pass    # silently swallows all errors, including ones you didn't expect

# Catching the wrong type — exception still propagates
try:
    open("missing.txt")
except ValueError:   # FileNotFoundError is NOT a ValueError — program still crashes
    print("Error")

# Unreachable specific handler
try:
    int("abc")
except Exception:    # catches everything
    print("Error")
except ValueError:   # dead code — never reached
    print("Bad value")
```

---

*Tags: #resource/python #python/exceptions #python/error-handling*

---

**Index:** [[Python Modules Index]]
**Previous:** [[Ex1 - Classes & Objects]] | **Next:** [[Ex3 - Python Collections]]
**See also:** [[Ex4 - Context Managers & with]] (exception-safe resource handling)
