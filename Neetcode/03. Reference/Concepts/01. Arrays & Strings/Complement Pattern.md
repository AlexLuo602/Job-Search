---
type: concept
tags: [concept, dsa, pattern]
---

# Complement Pattern

**TL;DR:** For a target sum, ask each element "what partner value would complete me, and have I already seen it?" instead of checking every pair.

## When to reach for it
- Problem asks for two (or more) elements that sum to a target value.
- You catch yourself writing a nested loop to check `nums[i] + nums[j] == target` for all pairs.
- The array is unsorted (if it's sorted, prefer [[0.TwoPointerGuide|Two Pointers]] — no hash map needed).
- The condition is symmetric in the two elements (order between them doesn't matter, only that both exist).

## How it works
For each element `x`, compute its complement, `target - x`, and check whether that complement has already been recorded. Trace **Two Sum** on `nums = [3, 2, 4]`, `target = 6`:

| i | x | complement (6 - x) | in `seen`? | action | `seen` after |
|---|---|---|---|---|---|
| 0 | 3 | 3 | no | insert 3 → 0 | `{3: 0}` |
| 1 | 2 | 4 | no | insert 2 → 1 | `{3: 0, 2: 1}` |
| 2 | 4 | 2 | **yes** (index 1) | return `[1, 2]` | — |

For **3Sum**, the pattern is applied one level up: fix one element `nums[i]`, then solve "find two elements in the rest of the array that sum to `-nums[i]`" — the same complement question, just on a sub-array (usually with [[0.TwoPointerGuide|Two Pointers]] instead of a hash map, since the array gets sorted first to handle duplicate-skipping cleanly).

## Why it works
Checking "has the complement appeared already?" against a running [[Hash Map]] is O(1), asked exactly once per element — so the total cost of finding a pair drops from O(n²) (compare every element to every other element) to O(n) (compare every element only to a *summary* of everything seen so far). The correctness argument is simple: a valid pair `(i, j)` with `i < j` and `nums[i] + nums[j] == target` is always discovered at step `j`, because by then `nums[i]` is guaranteed to already be in `seen` (it was inserted at step `i < j`). You never need to look ahead — only back — which is exactly what a single left-to-right pass with a hash map gives you for free.

## Template
```python
# Two-element complement search
def two_sum(nums, target):
    seen = {}
    for i, x in enumerate(nums):
        complement = target - x
        if complement in seen:
            return [seen[complement], i]
        seen[x] = i
    return []

# 3Sum: fix one element, reduce to a two-pointer complement search
def three_sum(nums):
    nums.sort()
    res = []
    for i in range(len(nums)):
        if i > 0 and nums[i] == nums[i - 1]:
            continue                      # skip duplicate anchors
        l, r = i + 1, len(nums) - 1
        while l < r:
            total = nums[i] + nums[l] + nums[r]
            if total < 0:
                l += 1
            elif total > 0:
                r -= 1
            else:
                res.append([nums[i], nums[l], nums[r]])
                l += 1
                while l < r and nums[l] == nums[l - 1]:
                    l += 1
    return res
```

## Complexity
Time: O(n) for the two-element hash map version (one pass, O(1) work per element); O(n²) for 3Sum (O(n) outer loop times an O(n) two-pointer inner scan). Space: O(n) for the hash map version; O(1) extra for the sorted two-pointer version (beyond the sort itself).

## Common pitfalls
- Inserting the current element into `seen` *before* checking for its complement — this can match an element against itself when `target == 2 * x`.
- Forgetting to skip duplicate values in 3Sum (both the anchor and the inner pointers), producing duplicate triplets.
- Reaching for a hash map when the array is already sorted — [[0.TwoPointerGuide|Two Pointers]] solves the same complement question in O(1) space.
- Returning values instead of indices (or vice versa) — re-read whether the problem wants positions or the numbers themselves.

## NeetCode examples
- [[03.TwoSum|TwoSum]] — the canonical complement lookup
- [[02.TwoSumII|TwoSumII]] — sorted variant, solved with [[0.TwoPointerGuide|Two Pointers]] instead of a hash map
- [[03.3Sum|3Sum]] — fix one element, reduce to a two-sum complement search

## Full guide
- [[Job Search/Neetcode/01. Questions/01. Arrays & Hashing/0.ArraysAndHashingGuide|Arrays & Hashing Guide]]
- [[Job Search/Neetcode/01. Questions/02. Two Pointer/0.TwoPointerGuide|Two Pointer Guide]]
