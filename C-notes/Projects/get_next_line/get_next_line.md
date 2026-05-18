---
tags: [projects, get_next_line, index]
---

# get_next_line

Reads one line at a time from a file descriptor using a static buffer that persists between calls.

---

## Components

| Note | Description |
|---|---|
| [[Projects/get_next_line/Core Logic\|Core Logic]] | `get_next_line()` — entry point, main loop, read phase, newline detection |
| [[Projects/get_next_line/Utils\|Utils]] | `ft_strnjoin()`, `clean_buffer()` — line building and buffer management |

---

## Key Concept

`BUFFER_SIZE` is set at compile time: `gcc -D BUFFER_SIZE=42`

---

**Home:** [[Welcome]]
