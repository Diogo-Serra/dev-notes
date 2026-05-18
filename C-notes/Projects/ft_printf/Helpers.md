---
tags: [projects, ft_printf]
---

# Helpers — printfhelper.c / ft_printf.c

## print_handler — format dispatcher

```c
static int  print_handler(va_list pargs, const char flag)
{
    if (flag == 'c' || flag == 's' || flag == '%')
        return (print_chr(pargs, flag));
    if (flag == 'd' || flag == 'i' || flag == 'u' || flag == 'x' || flag == 'X')
        return (print_nbr(pargs, flag));
    if (flag == 'p')
    {
        ptr = (unsigned long)va_arg(pargs, void *);
        if (ptr == 0)
            return (write(1, "(nil)", 5));
        count = write(1, "0x", 2);
        count += ft_putnbr_base(ptr, LOWER_HEX, 16);
        return (count);
    }
    return (0);  // unknown specifier → print nothing
}
```

---

## print_chr — character/string handler

```c
// %s → va_arg char*, print full string (NULL → "(null)")
// %c → va_arg int (char is promoted to int), write 1 byte
// %% → write literal '%'
```
> Note: `%c` extracts `int` from va_list because `char` is promoted when passed as variadic arg.

---

## print_nbr — numeric handler

```c
// %d / %i  → va_arg int,          DECIMAL base
// %u       → va_arg unsigned int,  DECIMAL base
// %x       → va_arg unsigned int,  LOWER_HEX base ("0123456789abcdef")
// %X       → va_arg unsigned int,  UPPER_HEX base ("0123456789ABCDEF")
```

---

## ft_putnbr_base — the core converter

```c
int     ft_putnbr_base(long n, const char *digits, int base)
{
    char arr[21];
    int  i = 20;
    long nb = (n < 0 && base == 10) ? -n : n;

    if (nb == 0)  arr[--i] = '0';
    while (nb)
    {
        arr[--i] = digits[nb % base];  // build digits right-to-left
        nb /= base;
    }
    if (n < 0 && base == 10)
        arr[--i] = '-';
    write(1, arr + i, 20 - i);
    return (20 - i);
}
```

### Why `long n`?
Handles negative values without overflow (passing `int` as `long` is safe).

### Why build right-to-left in `arr[21]`?
Digits are produced least-significant first. Build into the array backwards, then write the slice from `arr + i` to end.

---

## Base strings (defined in ft_printf.h)

```c
#define DECIMAL   "0123456789"
#define LOWER_HEX "0123456789abcdef"
#define UPPER_HEX "0123456789ABCDEF"
```

---

**Index:** [[Projects/ft_printf/ft_printf|ft_printf]]
