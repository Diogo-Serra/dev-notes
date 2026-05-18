---
tags: [projects, push_swap]
---

# Radix Sort — radix_sort.c

## Why Radix?

Can’t compare arbitrary pairs directly — only limited stack ops. Radix processes all elements bit by bit, which maps naturally to `pb` (bit=0) / `ra` (bit=1).

---

## Step 1: assign_index

```c
static void     assign_index(t_stack *stack)
{
    // For each node: count how many values are smaller → that's its rank
    // smallest gets index 0, next gets 1, ...
    current->index = rank;
}
```

Transforms arbitrary integers into sequential indices `0 … n-1`.
Now we sort indices instead of values — much simpler to reason about bit patterns.

---

## Step 2: get_max_bits

```c
static int      get_max_bits(int size)
{
    int bits = 0;
    while ((size - 1) >> bits)
        bits++;
    return (bits);
}
```

We only need enough bits to represent `n-1` (the largest index).

---

## Step 3: radix_sort

```c
void    radix_sort(t_stack **a, t_stack **b, int *move_count)
{
    assign_index(*a);
    size     = ft_lstsize(*a);
    max_bits = get_max_bits(size);

    for (i = 0; i < max_bits; i++)      // for each bit position
    {
        for (j = 0; j < size; j++)      // process all elements
        {
            if (((*a)->index >> i) & 1) == 0)
                exec_operation(a, b, "pb", move_count);  // bit=0 → push to B
            else
                exec_operation(a, b, "ra", move_count);  // bit=1 → keep in A (rotate)
        }
        while (*b)
            exec_operation(a, b, "pa", move_count);      // restore all from B to A
    }
}
```

---

## Trace Example (n=4, indices 0–3)

```
Indices in A:  3 1 2 0

Bit 0 (LSB):
  3 (11) bit0=1 → ra   A: 1 2 0 3
  1 (01) bit0=1 → ra   A: 2 0 3 1
  2 (10) bit0=0 → pb   B: 2     A: 0 3 1
  0 (00) bit0=0 → pb   B: 0 2   A: 3 1
  pa pa → A: 2 0 3 1   (bit-0 pass done)

Bit 1:
  ... repeat for next bit
```

---

## Complexity

| | |
|---|---|
| Time | O(n × log n) |
| Space | O(1) auxiliary (only 2 stacks) |

---

**Index:** [[Projects/push_swap/push_swap|push_swap]]
