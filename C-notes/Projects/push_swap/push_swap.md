---
tags: [projects, push_swap, index]
---

# push_swap

Sort a stack of integers in ascending order using two stacks (A and B) and the minimum number of operations.

---

## Components

| Note | Description |
|---|---|
| [[Projects/push_swap/Main\|Main]] | `main.c` — entry point, argument handling, stack init |
| [[Projects/push_swap/Parsing\|Parsing]] | `parsing.c` — input validation, duplicate check, indexing |
| [[Projects/push_swap/Operations\|Operations]] | `operations.c` — all stack ops: `sa`, `sb`, `ss`, `pa`, `pb`, `ra`, `rb`, `rr`, `rra`, `rrb`, `rrr` |
| [[Projects/push_swap/Sort Small\|Sort Small]] | `sort_small.c` — hardcoded optimal sort for ≤5 elements |
| [[Projects/push_swap/Radix Sort\|Radix Sort]] | `radix_sort.c` — bit-by-bit radix sort for large stacks |
| [[Projects/push_swap/Utils\|Utils]] | `utils.c` — stack helpers, size, top access |
| [[Projects/push_swap/Error Handling\|Error Handling]] | `error_handling.c` — validation errors, free and exit |

---

**Home:** [[Welcome]]
