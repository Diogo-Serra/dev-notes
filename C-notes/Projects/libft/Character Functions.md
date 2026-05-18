---
tags: [projects, libft]
---

# Character Functions

## Overview

| Function | Returns 1 if… | Returns 0 if… |
|---|---|---|
| `ft_isalpha(c)` | `A-Z` or `a-z` | anything else |
| `ft_isdigit(c)` | `0-9` | anything else |
| `ft_isalnum(c)` | alpha or digit | anything else |
| `ft_isascii(c)` | `0–127` | `c > 127` |
| `ft_isprint(c)` | `32–126` (printable) | anything else |
| `ft_toupper(c)` | returns `c - 32` if lowercase | returns `c` unchanged |
| `ft_tolower(c)` | returns `c + 32` if uppercase | returns `c` unchanged |

---

## Key Pattern

All check functions take `int c` (not `char`) — this matches the C standard.
Cast `unsigned char` when passing arbitrary bytes to avoid sign-extension bugs.

```c
int	ft_isalpha(int c)
{
	return ((c >= 'A' && c <= 'Z') || (c >= 'a' && c <= 'z'));
}
```

---

## Things to Remember

- `ft_toupper` / `ft_tolower` check the range before offsetting — they don't touch non-letter characters.
- `ft_isprint` includes space (`' '` = 32) but **not** DEL (`127`).
- These functions mirror `<ctype.h>` — no `#include <ctype.h>` allowed in 42.

---

**Index:** [[Projects/libft/libft|libft]]
