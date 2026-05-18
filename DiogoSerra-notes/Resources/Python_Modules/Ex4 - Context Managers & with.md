# Context Managers & the `with` Statement

**Module:** Data Archivist — Digital Preservation in the Cyber Archives
**Exercise:** Ex3 — Vault Security
**Topic:** `with` statement, context managers, safe file handling, file modes

---

## The Resource Management Problem

When you open a file, the OS allocates a file descriptor. If an exception occurs before you call `close()`, that descriptor leaks — the file stays locked and resources are wasted.

```python
# Unsafe — if read() throws, close() never runs
f = open("data.txt")
content = f.read()   # what if this fails?
f.close()            # may never be reached
```

The `with` statement solves this by guaranteeing cleanup regardless of what happens.

---

## The `with` Statement

`with` wraps a block of code and automatically calls the cleanup logic when the block exits — whether it exits normally or because of an exception.

```python
with open("data.txt") as f:
    content = f.read()
# f.close() is called automatically here, always
```

The file is closed the moment execution leaves the `with` block — even if an exception propagates out.

---

## How It Works: Context Managers

Any object that implements `__enter__` and `__exit__` can be used with `with`. These are called **context managers**.

| Method | Called | Purpose |
|---|---|---|
| `__enter__` | On entering `with` | Set up the resource, return it |
| `__exit__` | On leaving `with` | Tear down the resource, handle exceptions |

`open()` returns a file object that is a context manager. You rarely need to implement these methods yourself, but understanding them explains why `with` works.

---

## File Modes

The second argument to `open()` is the mode:

| Mode | Meaning |
|---|---|
| `"r"` | Read (default). Raises `FileNotFoundError` if file missing |
| `"w"` | Write. Creates the file if missing; **truncates** (overwrites) if it exists |
| `"a"` | Append. Creates if missing; adds to the end if it exists |
| `"x"` | Create. Raises `FileExistsError` if the file already exists |
| `"b"` | Binary mode — combine with others: `"rb"`, `"wb"` |
| `"+"` | Read + write — combine with others: `"r+"`, `"w+"` |

```python
with open("log.txt", "w") as f:    # create or overwrite
    f.write("Starting log\n")

with open("log.txt", "a") as f:    # append to existing
    f.write("Another entry\n")
```

---

## Reading Files

```python
with open("data.txt", "r") as f:
    content = f.read()          # entire file as one string

with open("data.txt", "r") as f:
    line = f.readline()         # one line at a time (includes \n)

with open("data.txt", "r") as f:
    lines = f.readlines()       # list of all lines (each includes \n)

# Most Pythonic — iterate line by line without loading everything
with open("data.txt", "r") as f:
    for line in f:
        print(line.strip())     # strip() removes trailing \n
```

---

## Writing Files

`write()` takes a string and returns the number of characters written. It does **not** add a newline automatically.

```python
with open("archive.txt", "w") as f:
    f.write("Line one\n")
    f.write("Line two\n")
```

To write multiple lines from a list:

```python
lines = ["Line one\n", "Line two\n", "Line three\n"]
with open("archive.txt", "w") as f:
    for line in lines:
        f.write(line)
```

---

## Type Hints with `typing.IO`

```python
import typing

def read_file(path: str) -> tuple[bool, str]:
    try:
        with open(path, "r") as f:
            return (True, f.read())
    except OSError as e:
        return (False, str(e))
```

For type-annotating the file object itself:

```python
import typing

def process(f: typing.IO[str]) -> str:
    return f.read()
```

`typing.IO[str]` means a text-mode file. `typing.IO[bytes]` for binary.

---

## Multiple Context Managers

You can open several files in one `with` statement:

```python
with open("input.txt", "r") as src, open("output.txt", "w") as dst:
    for line in src:
        dst.write(line.upper())
```

Both are guaranteed to close when the block exits.

---

## Returning Results from File Operations

A clean pattern is to return a `(success: bool, content: str)` tuple:

```python
def secure_read(path: str) -> tuple[bool, str]:
    try:
        with open(path, "r") as f:
            return (True, f.read())
    except OSError as e:
        return (False, str(e))

def secure_write(path: str, content: str) -> tuple[bool, str]:
    try:
        with open(path, "w") as f:
            f.write(content)
        return (True, "Content successfully written to file")
    except OSError as e:
        return (False, str(e))
```

The caller decides what to do with the result — the function never crashes.

---

## `with` vs Manual `open` / `close`

```python
# Manual — fragile
f = open("data.txt")
try:
    content = f.read()
finally:
    f.close()          # you have to write the finally yourself

# With statement — equivalent, cleaner
with open("data.txt") as f:
    content = f.read()
```

`with` is the idiomatic Python approach. Prefer it in all production code.

---

## Key Takeaways

- `with open(...) as f:` guarantees `f.close()` is called, even on exceptions
- The object returned by `open()` is a context manager — it implements `__enter__` and `__exit__`
- `"r"` reads, `"w"` overwrites, `"a"` appends, `"x"` creates (fails if exists)
- `read()` → full string; `readline()` → one line; `readlines()` → list; `for line in f` → most efficient
- `write()` does not add newlines — you must include `\n` manually
- Multiple files can be opened in one `with` block

---

## Common Mistakes

```python
# Reading after the with block — file is already closed
with open("data.txt") as f:
    pass
content = f.read()   # ValueError: I/O operation on closed file

# Forgetting newlines in write
with open("out.txt", "w") as f:
    f.write("Line 1")
    f.write("Line 2")
# File contains: Line 1Line 2  — no separation

# Using "w" when you meant "a" — silently destroys existing content
with open("log.txt", "w") as f:   # truncates the file!
    f.write("New entry\n")
```

---

*Tags: #resource/python #python/files #python/context-managers*

---

**Index:** [[Python Modules Index]]
**Previous:** [[Ex3 - Python Collections]] | **Next:** [[Ex5 - Abstract Classes & Polymorphism]]
**See also:** [[Ex2 - Exception Types]] (exceptions raised inside `with` blocks)
