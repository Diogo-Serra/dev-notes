---
tags: [projects, ft_printf]
---

# Entry Point — ft_printf.c

## ft_printf

```c
int     ft_printf(const char *src, ...)
{
    va_list pargs;
    int     count = 0;
    int     i = 0;

    if (!src)
        return (0);
    va_start(pargs, src);          // init va_list
    while (src[i])
    {
        if (src[i] == '%' && src[i + 1])
            count += print_handler(pargs, src[++i]); // skip '%', pass specifier
        else
            count += write(1, &src[i], 1);           // regular char
        i++;
    }
    va_end(pargs);                 // always clean up
    return (count);
}
```

---

## How it works

1. Guard: if `src` is NULL, return 0 immediately.
2. `va_start(pargs, src)` — initialises the variadic argument list.
3. Walk the format string character by character.
   - On `%` + a next char: call `print_handler` with the **specifier** char (not `%`). `++i` skips the specifier so the main loop doesn’t print it.
   - Anything else: write it directly to stdout.
4. `va_end` — must always be called before return.
5. Return **total characters printed**.

---

## Key Concepts

| Concept | Notes |
|---|---|
| `va_list` | opaque type that holds the argument state |
| `va_start(ap, last)` | `last` = last named parameter before `...` |
| `va_arg(ap, type)` | extract next argument as `type` |
| `va_end(ap)` | cleanup — mandatory before function returns |

---

## Edge Cases

- `ft_printf(0)` — NULL format: guard returns 0, no crash.
- `%%` — `src[i] == '%'` and `src[i+1] == '%'`: `print_handler` receives `'%'` and writes a literal `%`.

---

**Index:** [[Projects/ft_printf/ft_printf|ft_printf]]
