---
tags: [projects, push_swap]
---

# Error Handling — error_handling.c

## error_exit — single cleanup/exit path

```c
void    error_exit(t_stack **a, t_stack **b, char **split)
{
    if (split)  free_split(split);   // free ft_split result if present
    if (a)      free_stack(a);       // free stack A if present
    if (b)      free_stack(b);       // free stack B if present
    write(2, "Error\n", 6);         // print to stderr (fd=2)
    exit(1);
}
```

All three cleanups are conditional — pass NULL for any you don’t have yet.

---

## free_stack

```c
void    free_stack(t_stack **stack)
{
    t_stack *tmp;

    while (*stack)
    {
        tmp     = (*stack)->next;
        free(*stack);
        *stack  = tmp;
    }
    // *stack is NULL when done — no dangling pointer
}
```

---

## free_split

```c
void    free_split(char **split)
{
    int i = 0;
    while (split[i])
        free(split[i++]);   // free each string
    free(split);            // free the array of pointers
}
```

Mirrors the two-level allocation done by `ft_split`.

---

## Where error_exit is called

| Caller | Reason |
|---|---|
| `validate_and_add` | invalid number or overflow |
| `parse_input` | `ft_split` returns NULL or empty |
| `add_number` | `ft_lstnew` malloc fails |
| `main` | duplicate values detected |

---

**Index:** [[Projects/push_swap/push_swap|push_swap]]
