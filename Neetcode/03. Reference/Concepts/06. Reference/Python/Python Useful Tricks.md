---
type: concept
tags: ["concept"]
---

# Python Useful Tricks

## Swap without temp

```python
a, b = b, a
```

## Conditional expression (ternary)

```python
x = val_if_true if condition else val_if_false
```

## Clamp a value

```python
x = min(max(x, lo), hi)
```

## 2D grid initialization (correct vs wrong)

```python
grid = [[0] * cols for _ in range(rows)]   # ✓ each row is independent
grid = [[0] * cols] * rows                 # ✗ all rows share the SAME list object
```

## String → list of chars and back

```python
chars = list(s)     # mutable
s = "".join(chars)  # back to string
```

## Unpacking

```python
first, *rest = lst              # head/tail split
*init, last = lst               # init/last split
a, b = lst                      # exactly 2 elements
x, y, z = point                 # exactly 3 elements
```

## `zip(*matrix)` transposes a matrix

```python
matrix = [[1,2,3],[4,5,6]]
transposed = list(zip(*matrix))   # [(1,4),(2,5),(3,6)]
```

→ [[01.RotateImage|RotateImage]] (transpose is step 1 of clockwise rotation)

## `any` / `all` short-circuit

```python
any(x == target for x in lst)   # stops at first True — O(1) best case
all(x > 0 for x in lst)         # stops at first False
```

## `enumerate` with a start offset

```python
for i, val in enumerate(lst, start=1):  # i starts at 1
    ...
```

## Integer bit tricks (no import needed)

```python
n & 1           # check odd (1) or even (0)
n >> 1          # floor divide by 2
n << 1          # multiply by 2
n & (n - 1)     # clear lowest set bit (Brian Kernighan)
n & (-n)        # isolate lowest set bit
bin(n)          # '0b1010' — string representation
bin(n).count('1')  # popcount (slower than bit tricks for large n)
```

→ [[02.NumberOf1Bits|NumberOf1Bits]], [[03.CountingBits|CountingBits]]

## `collections.OrderedDict` — insertion-order dict with move-to-end

```python
from collections import OrderedDict
od = OrderedDict()
od.move_to_end(key)          # move key to the right end (most recent)
od.move_to_end(key, last=False)  # move to the left end (least recent)
od.popitem(last=True)        # pop from right (most recent)
od.popitem(last=False)       # pop from left (least recent = LRU)
```

→ [[09.LRUCache|LRUCache]]

## `itertools` for combinatorics (use in backtracking problems to verify, not as solution)

```python
from itertools import combinations, permutations, product

list(combinations([1,2,3], 2))   # [(1,2),(1,3),(2,3)] — no repeats, order irrelevant
list(permutations([1,2,3], 2))   # [(1,2),(1,3),(2,1),...] — order matters
list(product([0,1], repeat=3))   # all 3-bit binary strings
```

→ Useful to check brute-force answers for [[01.Subsets|Subsets]], [[03.Permutations|Permutations]]

## `accumulate` for prefix sums/products

```python
from itertools import accumulate
import operator

list(accumulate([1,2,3,4]))                    # [1, 3, 6, 10] — prefix sums
list(accumulate([1,2,3,4], operator.mul))      # [1, 2, 6, 24] — prefix products
list(accumulate([1,2,3,4], initial=0))         # [0, 1, 3, 6, 10] — with sentinel
```

→ [[06.ProductOfArrayExceptSelf|ProductOfArrayExceptSelf]]
