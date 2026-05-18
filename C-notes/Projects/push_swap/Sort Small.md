---
tags: [projects, push_swap]
---

# Sort Small — sort_small.c

## Strategy

| Size | Strategy |
|---|---|
| 2 | `sa` if top > next |
| 3 | `sort_three` (hardcoded, max 2 ops) |
| 4–5 | push smallest to B, sort remaining 3 in A, then `pa` all back |

---

## sort_three

```c
static void     sort_three(t_stack **a, int *move_count)
{
    int max = find_max_value(*a);

    if ((*a)->value == max)           // max is on top → rotate down
        exec_operation(a, NULL, "ra", move_count);
    else if ((*a)->next->value == max) // max is in middle → reverse rotate
        exec_reverse_operation(a, NULL, "rra", move_count);
    // now max is at bottom; if top > next, one swap finishes it
    if ((*a)->value > (*a)->next->value)
        exec_operation(a, NULL, "sa", move_count);
}
```

---

## sort_small (4–5 elements)

```c
void    sort_small(t_stack **a, t_stack **b, int *move_count)
{
    int size = ft_lstsize(*a);

    // push smallest elements to B until 3 remain in A
    while (size > 3)
    {
        rotate_min_to_top(a, b, move_count); // bring min to top
        exec_operation(a, b, "pb", move_count); // push to B
        size--;
    }
    sort_three(a, move_count);      // sort remaining 3 in A
    while (*b)                      // pull everything back
        exec_operation(a, b, "pa", move_count);
}
```

---

## rotate_min_to_top

```c
// find position of minimum value
// if min_pos <= size/2 → rotate forward (ra)
// else → reverse rotate (rra) — shorter path
```

Optimisation: choose the shorter rotation direction.

---

**Index:** [[Projects/push_swap/push_swap|push_swap]]
