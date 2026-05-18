---
tags: [projects, libft]
---

# Conversion & Output

## Conversion

```c
int    ft_atoi(const char *s);  // string → int  (stops at first non-digit)
char  *ft_itoa(int n);          // int → string  (malloc; caller must free)
void  *ft_calloc(size_t nmemb, size_t size); // malloc + zero-fill
```

## Output (all write to a file descriptor)

```c
void  ft_putchar_fd(char c, int fd);
void  ft_putstr_fd(char *s, int fd);
void  ft_putendl_fd(char *s, int fd);  // putstr + '\n'
void  ft_putnbr_fd(int n, int fd);
```

---

## Key Implementations

### ft_atoi — skip spaces → optional sign → digits
```c
int     ft_atoi(const char *s)
{
    int number = 0;
    int sign   = 1;

    while ((*s == 32) || (*s >= 9 && *s <= 13))  // skip whitespace
        s++;
    if (*s == '-' || *s == '+')
        if (*s++ == '-') sign *= -1;
    while (*s >= '0' && *s <= '9')
        number = (number * 10) + (*s++ - '0');
    return (number * sign);
}
```
> Does **not** handle overflow — use `ft_atol` (push_swap) when range matters.

### ft_itoa — build digits in a local array, then copy
```c
// Builds digits backwards into arr[11], then ft_calloc + ft_memcpy the slice.
// Handles INT_MIN by casting to long before negating.
```

### ft_calloc — overflow-safe malloc + zero
```c
void    *ft_calloc(size_t nmemb, size_t size)
{
    if (size != 0 && nmemb > SIZE_MAX / size)  // overflow guard
        return (NULL);
    ptr = malloc(nmemb * size);
    // zero-fill with ft_memset
}
```

---

## Things to Remember

- `ft_calloc` checks for multiplication overflow before calling `malloc`.
- `fd = 1` is stdout, `fd = 2` is stderr — all `_fd` functions work on any open fd.
- `ft_itoa` returns a heap-allocated string: always `free()` after use.

---

**Index:** [[Projects/libft/libft|libft]]
