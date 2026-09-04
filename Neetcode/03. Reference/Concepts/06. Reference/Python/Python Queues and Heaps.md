---
type: concept
tags: ["concept"]
---

# Python Queues and Heaps

## collections.deque

O(1) on **both ends**. Use instead of list when you need `popleft()`.

```python
from collections import deque

dq = deque()
dq = deque(iterable)
dq = deque(iterable, maxlen=k)   # fixed-size; auto-evicts oldest on append

dq.append(x)       # right end — O(1)
dq.appendleft(x)   # left end — O(1)
dq.pop()           # right end — O(1)
dq.popleft()       # left end — O(1)  ← the reason to use deque over list

dq.rotate(k)       # rotate right k steps; negative = left — O(k)
dq[0] / dq[-1]    # peek without popping — O(1)
```

`list.pop(0)` is **O(n)**. For BFS queues and sliding-window deques, always use `deque`. → [[08.BinaryTreeLevelOrderTraversal|BinaryTreeLevelOrderTraversal]], [[06.RottenOranges|RottenOranges]], [[06.SlidingWindowMaximum|SlidingWindowMaximum]]

---

## heapq (Min-Heap)

Python only provides a **min-heap**. Negate values for max-heap.

```python
import heapq

heapq.heappush(h, x)          # O(log n)
heapq.heappop(h)              # O(log n) — removes and returns smallest
h[0]                          # O(1) peek at min WITHOUT removing

heapq.heapify(lst)            # O(n) — converts a list in-place; faster than n pushes

heapq.heappushpop(h, x)      # push x then pop min — O(log n), faster than two calls
heapq.heapreplace(h, x)      # pop min then push x — O(log n); errors on empty heap

heapq.nlargest(k, iterable, key=None)   # O(n log k)
heapq.nsmallest(k, iterable, key=None)  # O(n log k)
# Use these only for k << n; for k ≈ n, sorted() is faster
```

### Max-heap

```python
heapq.heappush(h, -val)
largest = -heapq.heappop(h)
```

### Tuple ordering (tiebreaker)

When heap entries are tuples, Python compares element by element. If two priorities tie and the next element isn't comparable (e.g., `ListNode`), add a counter:

```python
import itertools
counter = itertools.count()
heapq.heappush(h, (priority, next(counter), item))
```

→ [[01.kthLargestElementInAStream|KthLargestElementInAStream]], [[05.TaskScheduler|TaskScheduler]], [[07.FindMedianFromDataStream|FindMedianFromDataStream]], [[10.MergeKSortedLists|MergeKSortedLists]]
