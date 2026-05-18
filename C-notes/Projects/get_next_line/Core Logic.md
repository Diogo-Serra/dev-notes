---
tags: [projects, get_next_line]
---

# Core Logic — get_next_line.c

## The Function

```c
char    *get_next_line(int fd)
{
    static char buffer[BUFFER_SIZE + 1];  // persists between calls
    char        *line;
    ssize_t      bytes;

    if (fd < 0 || BUFFER_SIZE < 1)
        return (NULL);
    line  = NULL;
    bytes = 1;
    while (bytes > 0 || buffer[0])         // loop while data available
    {
        if (!buffer[0])                    // buffer empty → read more
            bytes = read(fd, buffer, BUFFER_SIZE);
        if (bytes < 0)                     // read error
            return (clean_buffer(buffer), free(line), NULL);
        if (bytes == 0)                    // EOF
            return (line);
        line = ft_strnjoin(line, buffer);  // append buffer content to line
        if (!line)
            return (clean_buffer(buffer), NULL);
        clean_buffer(buffer);              // shift buffer past consumed content
        if (line && ft_strchr(line, '\n')) // found newline → done
            break ;
    }
    return (line);
}
```

---

## State Machine

```
call 1:  buffer empty → read → join → clean → newline found → return line
call 2:  buffer has leftover → skip read → join → clean → newline found → return line
call N:  bytes==0 AND buffer empty → return NULL (EOF signal)
```

---

## Why `static char buffer[]`?

- Lives between calls — holds data read past the last `\n`.
- Without it, bytes read after the newline would be lost.
- Shared across calls to the **same fd** (limitation of the single-buffer approach).

---

## Loop Condition: `bytes > 0 || buffer[0]`

| State | Meaning |
|---|---|
| `bytes > 0` | last `read()` returned data |
| `buffer[0]` | leftover data in buffer from a previous read |

Both must be false to stop — prevents dropping buffered data when `read()` returns 0.

---

## Compile

```bash
gcc -D BUFFER_SIZE=42 get_next_line.c get_next_line_utils.c
```

---

**Index:** [[Projects/get_next_line/get_next_line|get_next_line]]
