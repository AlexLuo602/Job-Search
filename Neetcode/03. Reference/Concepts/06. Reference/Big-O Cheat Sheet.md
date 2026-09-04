---
type: concept
tags: ["concept"]
---

# Big-O Cheat Sheet

**TL;DR:** A quick-reference table of time/space complexities for common data structures and algorithms.

## When to use
- Sanity-check your solution's complexity before coding.
- Compare candidate approaches and rule out too-slow ones (n ≤ 10^5 → aim for O(n log n) or better).

## Template
```python
# Rough feasibility guide (operations per second ≈ 10^8)
# n <= 20        → O(2^n) or O(n!) OK  (backtracking, permutations)
# n <= 1_000     → O(n^2) OK
# n <= 100_000   → O(n log n) OK
# n <= 10^7      → O(n) OK
# n <= 10^18     → O(log n) only
```

## Key idea / invariant
Big-O describes asymptotic upper-bound growth rate, ignoring constants. Nested loops → multiply; sequential loops → add; halving the input each step → log factor. Always analyze the worst case unless the problem guarantees otherwise.

## Complexity
| Structure / Algorithm     | Access | Search | Insert | Delete | Space  |
|---------------------------|--------|--------|--------|--------|--------|
| Array                     | O(1)   | O(n)   | O(n)   | O(n)   | O(n)   |
| Hash Map / Set            | —      | O(1)*  | O(1)*  | O(1)*  | O(n)   |
| Linked List               | O(n)   | O(n)   | O(1)†  | O(1)†  | O(n)   |
| Binary Search (sorted arr)| O(1)   | O(log n)| O(n)  | O(n)   | O(1)   |
| Binary Search Tree (bal.) | O(log n)| O(log n)| O(log n)| O(log n)| O(n)|
| Min/Max Heap              | O(1)   | O(n)   | O(log n)| O(log n)| O(n) |
| Trie                      | O(L)   | O(L)   | O(L)   | O(L)   | O(N·L) |
| BFS / DFS                 | —      | O(V+E) | —      | —      | O(V)   |
| Sorting (merge / heap)    | —      | O(n log n)| —   | —      | O(n) merge; O(1) heap|
| Dijkstra (binary heap)    | —      | O((V+E) log V)| — | — | O(V)  |

*amortized average; †given pointer to node

## Common pitfalls
- Counting O(n) inner work inside an O(n) loop as O(n) total — it's O(n²).
- Forgetting that string concatenation in a loop is O(n²) — use `''.join(list)`.
- Calling `in` on a list (O(n)) inside a loop when a set (O(1)) would suffice.
- Overlooking recursion stack space — recursive DFS on a path graph is O(n) space.
