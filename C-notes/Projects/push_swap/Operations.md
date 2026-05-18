---
tags: [projects, push_swap]
---

# Operations — operations.c

## The 3 Primitives

### swap(stack)
```c
// Swap value of node[0] and node[1] — no pointer changes
tmp = stack->value;
stack->value = stack->next->value;
stack->next->value = tmp;
```

### push(dst, src)
```c
// Detach top of src, prepend to dst
top  = *src;
*src = (*src)->next;
top->next = *dst;
*dst = top;
```

### rotate(stack, is_reverse)
```c
// Normal (ra/rb): first → last
// Reverse (rra/rrb): last → first
// Traverses to find last node + prev, then rewires pointers
```

---

## exec_operation — sa/sb/ss/pa/pb/ra/rb/rr

```c
void    exec_operation(t_stack **a, t_stack **b, char *flag, int *move_count)
{
    if      (flag == "sa") swap(*a);
    else if (flag == "sb") swap(*b);
    else if (flag == "ss") { swap(*a); swap(*b); }
    else if (flag == "pa") push(a, b);
    else if (flag == "pb") push(b, a);
    else if (flag == "ra") rotate(a, 0);
    else if (flag == "rb") rotate(b, 0);
    else                   { rotate(a, 0); rotate(b, 0); }  // rr
    ft_putendl_fd(flag, 1);    // print the operation name to stdout
    (*move_count)++;
}
```

## exec_reverse_operation — rra/rrb/rrr

```c
// flag[2] tells us which stack: 'a', 'b', or combined (rrr)
void    exec_reverse_operation(t_stack **a, t_stack **b, char *flag, int *move_count);
```

---

## All Operations Reference

| Op | Effect |
|---|---|
| `sa` | swap top 2 of A |
| `sb` | swap top 2 of B |
| `ss` | `sa` + `sb` |
| `pa` | push top of B → A |
| `pb` | push top of A → B |
| `ra` | rotate A up (top → bottom) |
| `rb` | rotate B up |
| `rr` | `ra` + `rb` |
| `rra` | reverse rotate A (bottom → top) |
| `rrb` | reverse rotate B |
| `rrr` | `rra` + `rrb` |

---

**Index:** [[Projects/push_swap/push_swap|push_swap]]
