---
tags: [projects, get_next_line]
---

# Utils — get_next_line_utils.c

## ft_strnjoin — append buffer to growing line

```c
char    *ft_strnjoin(char *line, char const *buffer)
{
    // alloc: ft_strllen(line) + ft_strllen(buffer) + 1
    // copy line into out
    // copy buffer into out, stopping after '\n'
    // free(line) — old allocation always freed
    // return new string
}
```

> Key: `ft_strllen` counts up to and **including** `\n`, so the allocation is exactly right and we stop copying after the newline.

---

## ft_strllen — length up to (and including) newline

```c
size_t  ft_strllen(const char *source)
{
    size_t i = 0;
    if (!source) return (0);
    while (source[i])
        if (source[i++] == '\n') break;  // stop after counting '\n'
    return (i);
}
```

Differs from `ft_strlen`: stops at `\n` rather than `\0`.

---

## clean_buffer — shift leftover data to front

```c
char    *clean_buffer(char *buffer)
{
    // 1. find position of '\n' (or end of string)
    // 2. if '\n' found, advance past it (i++)
    // 3. copy remaining bytes to start of buffer
    // 4. zero-fill the rest
    return (buffer);
}
```

### Before / After example

```
BUFFER_SIZE = 5

Buffer before clean:  [ h | e | l | l | o | \n | w | o | r ]
                                              ^
After clean_buffer:   [ w | o | r | \0 | \0 | \0 | ... ]
```

The `\n` and everything before it is consumed; `w o r` slides to position 0.

---

## ft_strchr — find character in string

```c
char    *ft_strchr(const char *line, int c)
{
    char ch = (unsigned char)c;
    while (*line)
    {
        if (*line == ch) return ((char *)line);
        line++;
    }
    if (ch == '\0') return ((char *)line);  // can find the null terminator
    return (NULL);
}
```

Used in the main loop to detect `\n` in the accumulated line.

---

**Index:** [[Projects/get_next_line/get_next_line|get_next_line]]
