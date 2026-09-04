---
type: concept
tags: ["concept"]
---

# Python Sorting and Binary Search

## bisect (Binary Search on Sorted Lists)

```python
import bisect

bisect.bisect_left(a, x, lo=0, hi=len(a))
# → index of first element >= x (leftmost insertion point)
# → if x is in a, returns its index; otherwise returns where it would go

bisect.bisect_right(a, x)   # alias: bisect.bisect()
# → index of first element > x (rightmost insertion point)
# → always returns a position AFTER existing occurrences of x

bisect.insort_left(a, x)    # insert x maintaining sort — O(n) due to list shift
bisect.insort(a, x)         # same as insort_right
```

### Common patterns

```python
# Is x present?
i = bisect.bisect_left(a, x)
found = i < len(a) and a[i] == x

# Count elements <= x
bisect.bisect_right(a, x)     # everything to the left is <= x

# Closest element to x
i = bisect.bisect_left(a, x)
candidates = []
if i < len(a): candidates.append(a[i])
if i > 0: candidates.append(a[i-1])
closest = min(candidates, key=lambda v: abs(v - x))
```

→ [[06.TimeBasedKeyValueStore|TimeBasedKeyValueStore]], [[11.LongestIncreasingSubsequence|LongestIncreasingSubsequence]] (O(n log n) with `bisect_left` on `tails` array)

---

## Sorting: key tricks

```python
# Sort intervals by start, break ties by end descending
intervals.sort(key=lambda x: (x[0], -x[1]))

# Sort strings by length then alphabetically
words.sort(key=lambda s: (len(s), s))

# Sort by multiple attributes of an object
items.sort(key=lambda x: (x.priority, x.name))

# Reverse only one dimension
sorted(pairs, key=lambda x: (-x[0], x[1]))   # desc first, asc second

# Custom total ordering (rare — prefer key=)
from functools import cmp_to_key
nums.sort(key=cmp_to_key(lambda a, b: 1 if str(a)+str(b) < str(b)+str(a) else -1))
```
