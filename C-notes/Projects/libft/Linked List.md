---
tags: [projects, libft]
---

# Linked List

## Struct

```c
typedef struct s_list
{
    void            *content;   // any data
    struct s_list   *next;      // pointer to next node (NULL if last)
}   t_list;
```

## Functions

| Function | What it does |
|---|---|
| `ft_lstnew(content)` | malloc new node, set content, next = NULL |
| `ft_lstadd_front(lst, new)` | new becomes the new head |
| `ft_lstadd_back(lst, new)` | traverse to last node, append |
| `ft_lstsize(lst)` | count nodes |
| `ft_lstlast(lst)` | return pointer to last node |
| `ft_lstdelone(node, del)` | free one node (del frees content, free the node) |
| `ft_lstclear(lst, del)` | delete all nodes, set `*lst = NULL` |
| `ft_lstiter(lst, f)` | apply `f(content)` to every node |
| `ft_lstmap(lst, f, del)` | build new list from `f(content)`, clean up on error |

---

## Key Implementations

### ft_lstnew
```c
t_list  *ft_lstnew(void *content)
{
    t_list *node = malloc(sizeof(t_list));
    if (!node)
        return (NULL);
    node->content = content;
    node->next    = NULL;
    return (node);
}
```

### ft_lstadd_back
```c
void    ft_lstadd_back(t_list **lst, t_list *new)
{
    if (!*lst) { *lst = new; return ; }  // empty list
    t_list *last = *lst;
    while (last->next)
        last = last->next;
    last->next = new;
}
```

### ft_lstmap — most complex
```c
t_list  *ft_lstmap(t_list *lst, void *(*f)(void *), void (*del)(void *))
{
    // apply f() to each content, build new list
    // on malloc fail: del(content), ft_lstclear new list, return NULL
}
```
> Pattern: always have a cleanup path on malloc failure.

---

## Things to Remember

- `**lst` is used so we can update the caller’s head pointer.
- `ft_lstclear` sets `*lst = NULL` after freeing — prevents dangling pointer.
- `del` is a function pointer: handles freeing whatever `content` points to.

---

**Index:** [[Projects/libft/libft|libft]]
