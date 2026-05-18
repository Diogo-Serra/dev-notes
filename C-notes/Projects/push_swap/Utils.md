---
tags: [projects, push_swap]
---

# Utils — utils.c

## Stack Checks

```c
int     stack_is_sorted(t_stack *stack)
{
    while (stack && stack->next)
    {
        if (stack->value > stack->next->value)
            return (0);     // found out-of-order pair
        stack = stack->next;
    }
    return (1);             // all pairs in order
}

int     stack_has_duplicates(t_stack *stack)
{
    // O(n²): nested loop, compare every pair
    // returns 1 if any two nodes share the same value
}
```

---

## Number Helpers

```c
long    ft_atol(const char *str);
// Same logic as ft_atoi but returns long — allows overflow detection in validate_and_add

int     is_valid_number(char *str);
// skip whitespace → optional sign → at least one digit → all digits
// returns 0 for empty, sign-only, or non-digit content
```

---

## Stack Builder

```c
void    add_number(t_stack **a, long num)
{
    t_stack *new = ft_lstnew(num);   // (void*)num cast to void* for t_list reuse)
    if (!new) error_exit(a, NULL, NULL);
    ft_lstadd_back(a, new);          // maintain order: first arg → top of stack
}
```

> push_swap reuses libft’s `ft_lstnew` and `ft_lstadd_back` with its own `t_stack` type.

---

## Why `long` in ft_atol?

```c
// INT_MAX =  2147483647
// INT_MIN = -2147483648
// ft_atoi returns int — overflow is silent/UB
// ft_atol returns long — safe to compare against INT_MAX / INT_MIN before casting
if (num > INT_MAX || num < INT_MIN)
    error_exit(...);
```

---

**Index:** [[Projects/push_swap/push_swap|push_swap]]
