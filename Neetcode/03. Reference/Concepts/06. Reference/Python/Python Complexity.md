---
type: concept
tags: ["concept"]
---

# Python Complexity

| Operation | Data Structure | Time |
|---|---|---|
| Lookup / insert / delete | dict / set | O(1) avg |
| Lookup | sorted list + bisect | O(log n) |
| Push / pop | heapq | O(log n) |
| Heapify | heapq | O(n) |
| Append / pop (right) | list / deque | O(1) |
| Pop left | deque | O(1) |
| Pop left | list | **O(n)** |
| Insert at index i | list | O(n) |
| Sort | list.sort / sorted | O(n log n) |
| Counter build | Counter | O(n) |
| most_common(k) | Counter | O(n log k) |
| Binary search | bisect | O(log n) |
| String concatenation in loop | `+=` | **O(n²)** — use join |
