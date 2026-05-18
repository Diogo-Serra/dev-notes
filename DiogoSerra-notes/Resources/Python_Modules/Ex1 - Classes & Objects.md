# Classes & Objects

**Module:** Code Cultivation — Object-Oriented Garden Systems
**Exercise:** Ex1 — Garden Data Organizer
**Topic:** Defining classes, creating instances, attributes, methods, `self`

---

## What is a Class?

A class is a **blueprint** for creating objects. It defines what data an object holds (attributes) and what it can do (methods). Once defined, you can create as many instances (objects) from it as you need.

```python
class Plant:
    pass   # empty class — valid Python, does nothing yet
```

---

## Defining Attributes

Attributes are variables that belong to a specific instance. You assign them directly on the object after creation:

```python
class Plant:
    pass

rose = Plant()       # create an instance
rose.name = "Rose"   # assign attributes to that instance
rose.height = 25
rose.age = 30
```

Each instance is independent — changing `rose.height` does not affect another `Plant` instance.

---

## What is `self`?

`self` is a reference to the **current instance** of the class. It is the first parameter of every method and is passed automatically when you call a method on an object. You never pass it yourself.

```python
class Plant:
    def show(self):
        print(self.name + ": " + str(self.height) + "cm, " + str(self.age) + " days old")
```

When you write `rose.show()`, Python internally calls `Plant.show(rose)` — `self` becomes `rose`.

---

## Defining Methods

A method is a function defined inside a class. It always takes `self` as its first parameter.

```python
class Plant:
    def show(self):
        print(self.name + ": " + str(self.height) + "cm, " + str(self.age) + " days old")
```

Calling it:

```python
rose = Plant()
rose.name = "Rose"
rose.height = 25
rose.age = 30

rose.show()
# Output: Rose: 25cm, 30 days old
```

---

## Creating Multiple Instances

Each instance is a separate object with its own data:

```python
rose = Plant()
rose.name = "Rose"
rose.height = 25
rose.age = 30

sunflower = Plant()
sunflower.name = "Sunflower"
sunflower.height = 80
sunflower.age = 45

cactus = Plant()
cactus.name = "Cactus"
cactus.height = 15
cactus.age = 120

rose.show()       # Rose: 25cm, 30 days old
sunflower.show()  # Sunflower: 80cm, 45 days old
cactus.show()     # Cactus: 15cm, 120 days old
```

---

## `class` vs Instance

| Concept | What it is | Example |
|---|---|---|
| Class | The blueprint | `Plant` |
| Instance | An object made from the blueprint | `rose`, `sunflower` |
| Attribute | Data stored on an instance | `rose.height` |
| Method | Function defined in a class, acting on `self` | `rose.show()` |

---

## Type Hints in Classes

Since this module requires type hints, annotate your attributes and method signatures:

```python
class Plant:
    name: str
    height: float
    age: int

    def show(self) -> None:
        print(f"{self.name}: {self.height}cm, {self.age} days old")
```

The `-> None` return type means the method does not return a value.

---

## f-strings: Cleaner Output Formatting

Instead of concatenating strings with `+` and `str()`, use f-strings (formatted string literals). Prefix the string with `f` and embed expressions in `{}`:

```python
# Without f-string
print(self.name + ": " + str(self.height) + "cm, " + str(self.age) + " days old")

# With f-string
print(f"{self.name}: {self.height}cm, {self.age} days old")
```

Both produce the same output. f-strings are cleaner, more readable, and less error-prone.

---

## Naming Conventions

| Element | Convention | Example |
|---|---|---|
| Class | `PascalCase` | `Plant`, `GardenData` |
| Method | `snake_case` | `show()`, `get_height()` |
| Instance | `snake_case` | `rose`, `my_plant` |
| Attribute | `snake_case` | `self.name`, `self.height` |

---

## Key Takeaways

- `class Name:` defines a blueprint; instances are created with `Name()`
- Attributes are assigned directly on instances: `obj.attr = value`
- Every method receives `self` as its first parameter — it refers to the calling instance
- Multiple instances from the same class are fully independent
- Use `-> None` when a method prints but returns nothing
- f-strings (`f"{var}"`) are the preferred way to format output

---

## Common Mistakes

```python
# Forgetting self in method signature
class Plant:
    def show():          # TypeError when called — missing self
        print(self.name)

# Accessing attribute before assigning it
rose = Plant()
rose.show()              # AttributeError: 'Plant' object has no attribute 'name'
                         # You must set rose.name before calling show()

# Confusing the class and an instance
Plant.show()             # TypeError — no instance to bind self to
```

---

*Tags: #resource/python #python/oop #python/classes*

---

**Index:** [[Python Modules Index]]
**Previous:** [[Ex0 - Defining Functions & print]] | **Next:** [[Ex2 - Exception Types]]
**See also:** [[Ex5 - Abstract Classes & Polymorphism]], [[Ex7 - Design Patterns]], [[Ex9 - Pydantic Models & Validation]]
