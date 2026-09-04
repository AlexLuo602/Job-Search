---
type: concept
tags: ["concept"]
---

# Python Lists and Iteration

## Construction

```python
[0] * n                          # O(n) — flat list of zeros
[[0] * cols for _ in range(rows)]  # O(mn) — 2D grid  ✓
[[0] * cols] * rows              # ✗ WRONG — all rows are the SAME object
```

## Append / Pop

| Operation | Complexity | Notes |
|---|---|---|
| `lst.append(x)` | O(1) amortized | always use over insert at end |
| `lst.pop()` | O(1) | removes last element |
| `lst.pop(i)` | O(n) | shifts everything after index i |
| `lst.insert(i, x)` | O(n) | avoid in inner loops |
| `lst.remove(x)` | O(n) | removes **first** occurrence; raises ValueError if absent |

For O(1) pop from the front, use `collections.deque`.

## Sorting — O(n log n)

```python
sorted(lst)                            # new sorted list
sorted(lst, reverse=True)
sorted(lst, key=lambda x: x[1])       # sort by second element
sorted(lst, key=lambda x: (x[0], -x[1]))  # primary asc, secondary desc

lst.sort(key=..., reverse=...)         # in-place, same options
```

Python's sort is **stable** — equal keys preserve original order. Multi-key tuples work because tuple comparison is lexicographic. → [[02.MergeInterval|MergeInterval]], [[03.Non-OverlappingIntervals|Non-OverlappingIntervals]]

## Reversal — O(n)

```python
lst[::-1]         # new reversed list (also works on strings)
lst.reverse()     # in-place, returns None
reversed(lst)     # iterator — wrap in list() to materialize
```

## Aggregates — O(n)

```python
sum(lst)
min(lst) / max(lst)
min(lst, key=lambda x: x[1])    # min by custom key
any(x > 0 for x in lst)         # short-circuits on first True
all(x > 0 for x in lst)         # short-circuits on first False
```

## Useful iteration

```python
enumerate(lst, start=0)          # (index, value) pairs
zip(a, b)                        # stops at shortest; zip(*matrix) transposes
zip(lst, lst[1:])                # adjacent pairs — useful for interval problems
list(map(int, str_list))         # apply int() to each element
```

## Iterators and Sequential Token Consumption

An iterator returns one item at a time and remembers where it stopped. `iter(iterable)` creates an iterator, and `next(iterator)` returns its next item.

```python
values = ["1", "2", "3"]
items = iter(values)

next(items)  # "1"
next(items)  # "2"
next(items)  # "3"
next(items)  # raises StopIteration
```

`StopIteration` means that the iterator has no items left. A `for` loop handles it automatically. When calling `next()` directly, only catch `StopIteration` if reaching the end is an expected case.

For deserialization, split the serialized string once and consume one token in each recursive call. The iterator keeps the shared position, so the recursion does not need a `nonlocal` index.

```python
tokens = iter(data.split(","))

def dfs():
    token = next(tokens)
    if token == "N":
        return None

    node = TreeNode(int(token))
    node.left = dfs()
    node.right = dfs()
    return node
```

Each `dfs()` call consumes exactly one token. If the serialized data is valid, the recursive calls consume all tokens in preorder.

## `lst.index(x, start=0, end=len(lst))` — O(n)

Returns index of first occurrence; raises `ValueError` if absent. Use `x in lst` (also O(n)) if you only need existence.
