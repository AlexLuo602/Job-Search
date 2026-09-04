---
question: "[[10.MergeKSortedLists|MergeKSortedLists]]"
topic: ["LinkedList"]
lc_difficulty: Hard
tags: ["neetcode-150"]
attempt_date: 2026-06-13
my_difficulty: Hard
status: Done
time_min: 37
review_concepts: []
---

# LeetCode: Merge k Sorted Lists
## Optimal In-Place Divide & Conquer

### 1. Algorithm Overview
**Approach:** Iterative Divide and Conquer with In-Place Array Compaction.
Instead of recursively splitting the lists or allocating new dynamic arrays for every level of the merge tree, this algorithm uses the front of the input `lists` array as a dynamic queue. It pairs up adjacent lists, merges them, and places the merged result at the `pos` index, effectively compacting the array from left to right.

### 2. Complexity Analysis
- **Time Complexity:** $\mathcal{O}(N \log k)$
  - *Why:* $N$ is the total number of nodes across all lists, and $k$ is the total number of linked lists. Each "level" of the divide-and-conquer tree processes all $N$ nodes, taking $\mathcal{O}(N)$ time. The number of active lists halves at each step, meaning there are $\log k$ levels.
- **Space Complexity:** $\mathcal{O}(1)$ Auxiliary Space
  - *Why:* Unlike standard divide-and-conquer approaches that create a new `merged_lists` array at each level, this solution overwrites the processed elements in the original `lists` array (`lists[pos] = merged_list`). Because `pos = i // 2`, `pos` is mathematically guaranteed to be $\le i$, ensuring unprocessed lists are never prematurely overwritten.

---

### 3. Clever Tricks Implemented

* **The Dummy Node:** Initializing `result = ListNode()` prevents you from having to write messy `if not head:` checks when picking the very first node. You just attach everything to `cur.next` and return `result.next`.
* **One-Pass Merging:** The `while i or j:` loop combined with the condition `if not j or (i and i.val < j.val):` beautifully handles both the merging phase and the exhaustion phase (when one list is empty) in a single unified loop.
* **Math-Free Ceiling Division:** Calculating the next size of the boundary using `unprocessed = (unprocessed + 1) // 2`. This avoids importing the `math` module while perfectly resolving the ceiling of integer division.
* **The Odd-Length Padding:** Appending `None` if the initial list is odd, and then relying on the fact that Python lists don't shrink physically. This means `lists[j]` safely pulls a `None` in later passes if `j` falls in the "abandoned" right half of the array.

---

### 4. Mistakes Encountered

1. [[Common Mistakes#Pointer Fatigue|Pointer Fatigue]] — forgot `cur = cur.next`, causing infinite loop
2. [[Common Mistakes#Off-By-One (Range Bounds)|Off-By-One (Range Bounds)]] — used `range(0, unprocessed + 1, 2)` instead of `range(0, unprocessed, 2)`
3. [[Common Mistakes#Floor vs Ceiling Division|Floor vs Ceiling Division]] — used `unprocessed //= 2`, dropping the odd list out
4. [[Common Mistakes#Empty Input (Void Case)|Empty Input (Void Case)]] — forgot to guard `len(lists) == 0` before accessing `lists[0]`

### 5. Frameworks Used

- [[Common Mistakes#State Change Checklist|State Change Checklist]]
- [[Common Mistakes#Extreme Boundary Tracing|Extreme Boundary Tracing]]
- [[Common Mistakes#The 0, 1, 2 Rule|The 0, 1, 2 Rule]]

---

### 6. Visualizing the Compaction

*Why the in-place overwrite doesn't destroy data — the `pos = i // 2` trick:*

```text
Initial lists: [A, B, C, D, E] -> padded to [A, B, C, D, E, None]
Unprocessed = 6

--- PASS 1 ---
i=0, pos=0: merge(A, B) -> store in lists[0]
i=2, pos=1: merge(C, D) -> store in lists[1]
i=4, pos=2: merge(E, None) -> store in lists[2]
Array state: [AB, CD, E, D, E, None] 
Unprocessed becomes (6+1)//2 = 3

--- PASS 2 ---
i=0, pos=0: merge(AB, CD) -> store in lists[0]
i=2, pos=1: merge(E, None) -> store in lists[1]
Array state: [ABCD, E, E, D, E, None]
Unprocessed becomes (3+1)//2 = 2

--- PASS 3 ---
i=0, pos=0: merge(ABCD, E) -> store in lists[0]
Array state: [ABCDE, E, E, D, E, None]
Unprocessed becomes (2+1)//2 = 1 -> Loop Ends. Return lists[0].
```
