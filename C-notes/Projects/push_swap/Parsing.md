---
tags: [projects, push_swap]
---

# Parsing — parsing.c

## parse_input

```c
t_stack *parse_input(int argc, char **argv)
{
    // For each argv[i]:
    //   ft_split(argv[i], ' ')  → handle "3 2 1" as single arg or multiple args
    //   process_split → validate and add each token
    // Returns head of stack A, or NULL on error
}
```

### Why ft_split?
The program accepts both `./push_swap 3 2 1` and `./push_swap "3 2 1"` — splitting on space handles both.

---

## validate_and_add

```c
static void     validate_and_add(t_stack **a, char *str, char **split)
{
    long num;

    if (!is_valid_number(str))              // must be digits (+ optional sign)
        error_exit(a, NULL, split);
    num = ft_atol(str);
    if (num > INT_MAX || num < INT_MIN)     // must fit in int
        error_exit(a, NULL, split);
    add_number(a, num);
}
```

---

## is_valid_number (in utils.c)

```c
// skip whitespace → optional '-' or '+' → must have at least one digit → all chars must be digits
int     is_valid_number(char *str);
```

Returns 0 for empty strings, non-digit chars, or only a sign.

---

## add_number (in utils.c)

```c
void    add_number(t_stack **a, long num)
{
    t_stack *new = ft_lstnew(num);   // reuses libft's ft_lstnew
    ft_lstadd_back(a, new);          // numbers added in order → first arg = top of stack
}
```

---

## Error Conditions

| Input | Behaviour |
|---|---|
| Non-integer arg | `write(2, "Error\n", 6)` + exit(1) |
| Overflow (`> INT_MAX`) | same |
| Duplicate values | detected in `main`, same |
| Empty string arg | same |

---

**Index:** [[Projects/push_swap/push_swap|push_swap]]
