# Pydantic — Models & Validation

**Module:** Cosmic Data — Discover Pydantic Models & Validation
**Exercise:** Ex0–Ex2 — Space Station Data, Alien Contact Logs, Space Crew Management
**Topic:** `BaseModel`, `Field`, type coercion, enums, `@model_validator`, nested models, `ValidationError`

---

## What is Pydantic?

Pydantic is a Python library for **data validation using type annotations**. You declare the shape and constraints of your data as a class, and Pydantic enforces them automatically when you instantiate it.

```bash
pip install pydantic
```

Three core guarantees:
1. **Type coercion** — `"42"` becomes `42` for an `int` field where possible
2. **Validation** — values that violate constraints raise a `ValidationError` with clear messages
3. **Serialisation** — models convert cleanly to/from dicts and JSON

---

## `BaseModel` — The Foundation

Every Pydantic model inherits from `BaseModel`. Fields are declared as class-level type-annotated attributes:

```python
from pydantic import BaseModel
from datetime import datetime

class SpaceStation(BaseModel):
    station_id: str
    name: str
    crew_size: int
    power_level: float
    is_operational: bool = True          # default value
    last_maintenance: datetime
```

Instantiate by passing keyword arguments:

```python
station = SpaceStation(
    station_id="ISS001",
    name="International Space Station",
    crew_size=6,
    power_level=85.5,
    last_maintenance="2024-01-15T08:30:00",   # string → datetime automatically
)
```

Accessing fields is the same as any class attribute:

```python
print(station.crew_size)        # 6
print(station.is_operational)   # True
```

---

## Type Coercion

Pydantic automatically converts compatible types:

| Input | Field type | Result |
|---|---|---|
| `"42"` | `int` | `42` |
| `42` | `float` | `42.0` |
| `"2024-01-15T08:30:00"` | `datetime` | `datetime(2024, 1, 15, 8, 30)` |
| `1` | `bool` | `True` |
| `"true"` | `bool` | `True` |

Incompatible types raise a `ValidationError`:

```python
SpaceStation(crew_size="many", ...)   # ValidationError: not a valid integer
```

---

## `Field` — Constraints and Metadata

Use `Field(...)` to add validation constraints beyond the type:

```python
from pydantic import BaseModel, Field
from typing import Optional

class SpaceStation(BaseModel):
    station_id: str       = Field(..., min_length=3, max_length=10)
    name: str             = Field(..., min_length=1, max_length=50)
    crew_size: int        = Field(..., ge=1, le=20)        # 1 ≤ crew ≤ 20
    power_level: float    = Field(..., ge=0.0, le=100.0)
    notes: Optional[str]  = Field(None, max_length=200)    # optional with default None
```

`...` (Ellipsis) means the field is **required** — no default.

### Common `Field` constraints

| Constraint | Applies to | Meaning |
|---|---|---|
| `min_length` / `max_length` | `str`, `list` | Length bounds |
| `ge` / `le` | `int`, `float` | ≥ / ≤ |
| `gt` / `lt` | `int`, `float` | > / < |
| `pattern` | `str` | Regex the value must match |
| `default` | Any | Default value |
| `description` | Any | Documentation string |

---

## Optional Fields

`Optional[T]` (or `T | None`) marks a field that may be absent. Combine with a `None` default:

```python
from typing import Optional

class AlienContact(BaseModel):
    message_received: Optional[str] = Field(None, max_length=500)
```

If not provided, the value is `None` — no `ValidationError`.

---

## Enums

Use Python's standard `Enum` class to restrict a field to a fixed set of values:

```python
from enum import Enum
from pydantic import BaseModel

class ContactType(str, Enum):
    radio = "radio"
    visual = "visual"
    physical = "physical"
    telepathic = "telepathic"

class AlienContact(BaseModel):
    contact_type: ContactType
```

Inheriting from `str` (a `str` enum) allows Pydantic to accept the string value directly:

```python
contact = AlienContact(contact_type="radio")   # coerces to ContactType.radio
```

Passing an invalid string raises a `ValidationError`.

---

## `@model_validator` — Custom Cross-field Validation

`Field` constraints only validate a single field in isolation. For rules that depend on multiple fields, use `@model_validator(mode='after')`. It runs after all individual fields have been validated and must return `self`.

```python
from pydantic import BaseModel, model_validator

class AlienContact(BaseModel):
    contact_id: str
    contact_type: ContactType
    signal_strength: float
    witness_count: int
    message_received: Optional[str] = None

    @model_validator(mode='after')
    def validate_business_rules(self) -> 'AlienContact':
        # Rule 1 — contact ID must start with "AC"
        if not self.contact_id.startswith("AC"):
            raise ValueError("Contact ID must start with 'AC'")

        # Rule 2 — physical contact must be verified
        if self.contact_type == ContactType.physical and not self.is_verified:
            raise ValueError("Physical contact reports must be verified")

        # Rule 3 — strong signals must include a message
        if self.signal_strength > 7.0 and not self.message_received:
            raise ValueError("Strong signals (> 7.0) should include received messages")

        return self   # always return self
```

---

## Handling `ValidationError`

When validation fails, Pydantic raises `pydantic.ValidationError`. Catch it to handle the error gracefully:

```python
from pydantic import ValidationError

try:
    station = SpaceStation(crew_size=50, ...)
except ValidationError as e:
    print(e)          # human-readable summary of all errors
    print(e.errors()) # list of error dicts — field, type, message
```

`e.errors()` returns a list of dicts — useful for API responses:

```python
[
    {
        'type': 'less_than_equal',
        'loc': ('crew_size',),
        'msg': 'Input should be less than or equal to 20',
        'input': 50,
    }
]
```

---

## Nested Models

A model field can be another Pydantic model. Validation is recursive — the nested model is validated first:

```python
class CrewMember(BaseModel):
    member_id: str  = Field(..., min_length=3, max_length=10)
    name: str       = Field(..., min_length=2, max_length=50)
    rank: Rank
    age: int        = Field(..., ge=18, le=80)
    is_active: bool = True

class SpaceMission(BaseModel):
    mission_id: str         = Field(..., min_length=5, max_length=15)
    crew: list[CrewMember]  = Field(..., min_length=1, max_length=12)
    duration_days: int      = Field(..., ge=1, le=3650)
```

You can pass crew members as dicts — Pydantic coerces them automatically:

```python
mission = SpaceMission(
    mission_id="M2024_MARS",
    crew=[
        {"member_id": "CM001", "name": "Sarah Connor", "rank": "commander", "age": 40},
    ],
    duration_days=900,
    ...
)
```

If any `CrewMember` fails validation, the error is nested under the `crew` field in the `ValidationError`.

---

## Serialisation — `model_dump()` and `model_dump_json()`

Convert a model instance to a dict or JSON string:

```python
station = SpaceStation(...)

d = station.model_dump()           # dict
j = station.model_dump_json()      # JSON string

# Round-trip: reconstruct from dict
station2 = SpaceStation.model_validate(d)
```

---

## Pydantic v1 vs v2 — What Changed

This module uses **Pydantic v2**. Key differences from v1:

| v1 | v2 | Notes |
|---|---|---|
| `@validator` | `@model_validator(mode='after')` | v1 decorator is deprecated |
| `.dict()` | `.model_dump()` | Renamed |
| `.json()` | `.model_dump_json()` | Renamed |
| `@root_validator` | `@model_validator(mode='before'/'after')` | Unified |

Always use v2 syntax — `@validator` still exists but raises deprecation warnings.

---

## Key Takeaways

- Inherit from `BaseModel` and annotate fields — Pydantic validates on instantiation
- `Field(...)` adds constraints: `ge`, `le`, `min_length`, `max_length`, `pattern`, etc.
- Pydantic coerces compatible types automatically (string `→` `datetime`, `"1"` `→` `int`)
- `Optional[str] = None` marks a field as not required
- `Enum` (preferably `str, Enum`) restricts a field to a fixed set of values
- `@model_validator(mode='after')` validates cross-field rules — always `return self`
- Catch `ValidationError` for readable error messages; use `.errors()` for structured output
- Nested models are validated recursively — invalid nested data surfaces in the parent's error

---

## Common Mistakes

```python
# Using v1 @validator — deprecated in v2
@validator('contact_id')           # DeprecationWarning / broken in strict v2
def check_id(cls, v):
    ...

# Forgetting return self in model_validator
@model_validator(mode='after')
def validate_rules(self) -> 'AlienContact':
    if ...:
        raise ValueError("...")
    # missing return self → Pydantic receives None → validation breaks

# Confusing Optional with a required field
message: Optional[str]             # still required — must pass None explicitly
message: Optional[str] = None     # truly optional — omitting it gives None

# Mixing ge/le with min_length/max_length
crew_size: int = Field(..., min_length=1)   # wrong — min_length is for str/list
crew_size: int = Field(..., ge=1)           # correct for numbers
```

---

*Tags: #resource/python #python/pydantic #python/validation #python/data-engineering*

---

**Index:** [[Python Modules Index]]
**Previous:** [[Ex8 - Virtual Environments & Packages]] | **Next:** [[Ex10 - Functional Programming]]
**See also:** [[Ex1 - Classes & Objects]], [[Ex3 - Python Collections]]
