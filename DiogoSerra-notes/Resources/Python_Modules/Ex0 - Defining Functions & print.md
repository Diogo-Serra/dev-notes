# Defining Functions & `print()`

**Module:** Growing Code — Python Fundamentals
**Exercise:** Ex0 — Hello Garden
**Topic:** Writing functions, basic output

---

## What is a Function?

A function is a named, reusable block of code. You define it once and call it whenever you need it. In Python, functions are defined with the `def` keyword.

```python
def greet():
    print("Hello!")
```

Breaking it down:

| Part | Meaning |
|---|---|
| `def` | keyword that starts a function definition |
| `greet` | the name you give the function |
| `()` | parameter list — empty here, no inputs needed |
| `:` | colon marks the start of the function body |
| indented block | everything indented belongs to the function |

---

## Indentation

Python uses **indentation** (4 spaces) to define blocks. There are no `{}` braces. The indentation is not optional — it is the syntax.

```python
def greet():
    print("Hello!")   # inside the function
    print("World!")   # still inside the function

print("Outside")      # outside the function
```

---

## The `print()` Function

`print()` is a built-in function that writes output to the terminal, followed by a newline.

```python
print("Hello, Garden Community!")
# Output: Hello, Garden Community!
```

**String literals** — the text you pass to `print()` — can use single or double quotes interchangeably:

```python
print('Single quotes work')
print("Double quotes work")
```

Both produce the same result. Pick one style and be consistent.

---

## Calling a Function

Defining a function does not run it. You must **call** it by using its name followed by `()`.

```python
def ft_hello_garden():
    print("Hello, Garden Community!")

ft_hello_garden()   # this line actually runs the function
```

```
Hello, Garden Community!
```

---

## Function Naming Conventions

Python follows the **snake_case** convention for function names:

```python
# Good
def ft_hello_garden():
    ...

# Avoid
def FtHelloGarden():   # PascalCase — used for classes, not functions
def fthellgarden():    # hard to read
```

The `ft_` prefix in this project is a naming requirement — in the real world, you would just write `hello_garden()`.

---

## What Happens When Python Reads Your File

Python reads your file top to bottom. When it reaches a `def` block, it stores the function in memory but does not run it yet. The function only runs when it is explicitly called.

```python
# Python reads this and stores it — nothing is printed yet
def ft_hello_garden():
    print("Hello, Garden Community!")

# Python sees this call and executes the function
ft_hello_garden()
```

This is why, for this project, you write **only the function** and let `main.py` call it for you.

---

## `print()` with Multiple Values

You can pass multiple arguments to `print()`, separated by commas. It joins them with a space by default.

```python
print("Hello,", "Garden!")
# Output: Hello, Garden!
```

You can change the separator with the `sep` parameter:

```python
print("2026", "05", "18", sep="-")
# Output: 2026-05-18
```

---

## Key Takeaways

- `def name():` defines a function; the indented block is its body
- Indentation is mandatory — 4 spaces is the standard
- `print("text")` outputs a line to the terminal
- A function does nothing until it is called with `name()`
- String literals use single or double quotes — be consistent

---

## Common Mistakes

```python
# Missing colon
def ft_hello_garden()       # SyntaxError
    print("Hello!")

# Wrong indentation
def ft_hello_garden():
print("Hello!")             # IndentationError

# Defining but never calling
def ft_hello_garden():
    print("Hello!")
# (nothing happens — you forgot to call it)
```

---

*Tags: #resource/python #python/functions #python/print*

---

**Index:** [[Python Modules Index]]
**Next:** [[Ex1 - Classes & Objects]]
**See also:** [[Ex10 - Functional Programming]] (lambdas and HOF build on functions)
