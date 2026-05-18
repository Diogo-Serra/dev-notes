---
tags: [projects, push_swap]
---

# Main — main.c

## Entry Point

```c
int     main(int argc, char **argv)
{
    t_stack *a = NULL;
    t_stack *b = NULL;
    int      move_count = 0;

    if (argc < 2) return (0);          // no args → nothing to sort
    a = parse_input(argc, argv);       // build stack A from args
    if (!a) return (1);                // parse error (already printed)
    if (stack_has_duplicates(a))       // validate: no duplicates allowed
    { error_exit(&a, &b, NULL); return (1); }
    push_swap(&a, &b, &move_count);    // sort
    free_stack(&a);
    free_stack(&b);
    return (0);
}
```

---

## push_swap dispatcher

```c
static void     push_swap(t_stack **a, t_stack **b, int *move_count)
{
    int size;

    if (stack_is_sorted(*a)) return;   // already sorted → 0 moves
    size = ft_lstsize(*a);
    if (size <= 5)
        sort_small(a, b, move_count);
    else
        radix_sort(a, b, move_count);
}
```

---

## Flow

```
argv → parse_input → stack A built
         ↓
   duplicate check
         ↓
   already sorted? → exit early
         ↓
   size ≤ 5 → sort_small
   size > 5 → radix_sort
         ↓
   free both stacks, exit 0
```

---

## Test Command

```bash
# Generate 500 unique random numbers, sort, validate with checker
ARG=$(shuf -n 500 -i 1-100000 | tr '\n' ' '); echo $ARG; ./push_swap $ARG | ./checker_linux $ARG
```

---

**Index:** [[Projects/push_swap/push_swap|push_swap]]
