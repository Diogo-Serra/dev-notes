---
tags: [projects, libft]
---

# Memory Functions

## Functions

```c
void  *ft_memset(void *s, int c, size_t n);              // fill n bytes with c
void   ft_bzero(void *s, size_t n);                      // fill n bytes with 0
void  *ft_memcpy(void *dst, const void *src, size_t n);  // copy n bytes (no overlap)
void  *ft_memmove(void *dst, const void *src, size_t n); // copy n bytes (overlap-safe)
void  *ft_memchr(const void *s, int c, size_t n);        // first byte == c within n
int    ft_memcmp(const void *s1, const void *s2, size_t n); // compare n bytes
```

---

## Key Implementations

### ft_memset — fill byte by byte
```c
void    *ft_memset(void *s, int c, size_t n)
{
    unsigned char *p = (unsigned char *)s;
    while (n--)
        *p++ = (unsigned char)c;
    return (s);
}
```
> Cast to `unsigned char *` — operating on raw bytes, never raw `int`.

### ft_memcpy — straight forward copy
```c
void    *ft_memcpy(void *dst, const void *src, size_t n)
{
    unsigned char       *d = (unsigned char *)dst;
    const unsigned char *s = (const unsigned char *)src;
    while (n--)
        *d++ = *s++;
    return (dst);
}
```
> Undefined behaviour if regions overlap — use `ft_memmove` instead.

### ft_memmove — overlap-safe
Checks if `dst > src` and if so copies **backwards** to avoid clobbering source bytes mid-copy.

---

## Things to Remember

- Always cast to `unsigned char *` when working with raw memory — avoids sign-extension issues.
- `ft_bzero(s, n)` is just `ft_memset(s, 0, n)` — exists for legacy POSIX compat.
- `ft_memcmp` returns the difference of the first differing bytes (like `strncmp` but for raw memory).

---

**Index:** [[Projects/libft/libft|libft]]
