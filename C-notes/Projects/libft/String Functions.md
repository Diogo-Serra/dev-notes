---
tags: [projects, libft]
---

# String Functions

## Inspection

```c
size_t  ft_strlen(const char *s)      // count chars until '\0'
char   *ft_strchr(const char *s, int c)  // first occurrence of c (finds '\0' too)
char   *ft_strrchr(const char *s, int c) // last occurrence of c
int     ft_strncmp(const char *s1, const char *s2, size_t n) // compare up to n bytes
char   *ft_strnstr(const char *h, const char *n, size_t len) // find needle in haystack
```

## Copy & Concat

```c
// both return total length they tried to write (not what fits)
size_t  ft_strlcpy(char *dst, const char *src, size_t size); // copy, always NUL-terminate
size_t  ft_strlcat(char *dst, const char *src, size_t size); // append, always NUL-terminate

char   *ft_strdup(const char *s);     // malloc + copy; caller must free
```

## Allocation-Based

```c
char   *ft_substr(char const *s, unsigned int start, size_t len);  // slice of s
char   *ft_strjoin(char const *s1, char const *s2);               // s1 + s2 in new string
char   *ft_strtrim(char const *s1, char const *set);              // strip set chars from both ends
char  **ft_split(const char *src, char sep);                      // split by sep, NULL-terminated array
```

## Apply / Iterate

```c
// ft_strmapi   — returns new string, applies f(index, char) to each char
// ft_striteri  — applies f(index, &char) in-place, no return
char   *ft_strmapi(char const *s, char (*f)(unsigned int, char));
void    ft_striteri(char *s, void (*f)(unsigned int, char *));
```

---

## Key Implementations

### ft_substr
```c
char    *ft_substr(char const *s, unsigned int start, size_t len)
{
    size_t  slen = ft_strlen(s);
    size_t  n;

    if (start >= slen)      n = 0;               // start past end → empty
    else if (len > slen - start) n = slen - start; // len too big → clamp
    else                    n = len;
    out = ft_calloc(n + 1, sizeof(char));
    ft_memcpy(out, s + start, n);
    return (out);
}
```

### ft_split
- `ft_nwords` counts words (transitions from sep to non-sep).
- Allocates `(nwords + 1)` pointers (last is NULL).
- On any `malloc` failure, `free_heap` cleans everything and returns NULL.

---

## Things to Remember

- `ft_strlcpy` / `ft_strlcat` return the **total length** they would have produced — use this to detect truncation.
- `ft_strdup` returns NULL on malloc failure; always check.
- `ft_split` returns a NULL-terminated `char **`; caller must free each string and then the array.

---

**Index:** [[Projects/libft/libft|libft]]
