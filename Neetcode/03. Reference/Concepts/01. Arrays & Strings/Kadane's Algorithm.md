---
type: concept
tags: [concept, dsa, pattern]
---

# Kadane's Algorithm

**TL;DR:** Track the best sum ending *right here*, and reset to just the current element whenever carrying the previous sum forward would only hurt — that single decision finds the max-sum subarray in one pass.

## When to reach for it
- "Maximum sum contiguous subarray" or a variant (max product, max sum circular subarray).
- You need a running best over subarrays but can't afford to check all O(n²) subarray sums.
- The array can contain negative numbers (if everything is non-negative, the whole array is always the answer — Kadane's is only interesting because negatives can make a prefix a liability).

## How it works
Keep two values: `cur`, the best sum of a subarray *ending at the current index*, and `best`, the best sum seen anywhere so far. At each element, decide whether extending the previous subarray helps or hurts — if `cur` would go negative, starting fresh from the current element is strictly better. Trace `nums = [-2, 1, -3, 4, -1, 2]`:

| i | nums[i] | cur = max(nums[i], cur + nums[i]) | best |
|---|---|---|---|
| 0 | -2 | max(-2, -2) = -2 | -2 |
| 1 | 1 | max(1, -2+1=-1) = 1 | 1 |
| 2 | -3 | max(-3, 1-3=-2) = -2 | 1 |
| 3 | 4 | max(4, -2+4=2) = 4 | 4 |
| 4 | -1 | max(-1, 4-1=3) = 3 | 4 |
| 5 | 2 | max(2, 3+2=5) = 5 | 5 |

Final answer: `best = 5`, from the subarray `[4, -1, 2]`.

## Why it works
Kadane's is a one-dimensional DP where the state is exactly "the best subarray sum ending at index i" (`dp[i]`), with recurrence `dp[i] = max(nums[i], dp[i-1] + nums[i])`. The extend-or-restart choice is optimal by a simple exchange argument: if `dp[i-1]` is negative, then *any* subarray ending at `i` that includes index `i-1` could be improved by dropping the negative prefix and starting fresh at `i` — a negative prefix can never help a sum, only hurt it. So keeping a negative `dp[i-1]` around is never useful; discarding it (restarting at `nums[i]`) dominates every alternative. Because `dp[i]` only depends on `dp[i-1]`, you don't need to store the whole DP array — a single rolling variable (`cur`) suffices, and `best` just tracks the running maximum over all `dp[i]`, since the optimal subarray must end *somewhere*.

## Template
```python
def max_subarray(nums):
    cur = best = nums[0]
    for x in nums[1:]:
        cur = max(x, cur + x)
        best = max(best, cur)
    return best
```

## Complexity
Time: O(n) — one pass, O(1) work per element (a max of two numbers and an addition). Space: O(1) — only `cur` and `best` are tracked, no DP array needed since each state depends only on its immediate predecessor.

## Common pitfalls
- Initializing `cur = 0` instead of `nums[0]` — if all numbers are negative, this incorrectly allows an "empty" subarray to beat the true (negative) answer.
- Resetting `cur` to `0` instead of `nums[i]` when it goes negative — you must still consider starting a *new* subarray at the current element, not skip it.
- Confusing "reset when `cur` goes negative" with "reset when `nums[i]` is negative" — the reset condition depends on the running sum, not the individual element.
- Applying the same reset logic to max *product* subarray without tracking both a running min and max — a negative number can flip a very negative running product into the new maximum.

## NeetCode examples
- [[01.MaximumSubarray|MaximumSubarray]] — the canonical extend-or-restart application
- [[09.MaxProductSubArray|MaxProductSubArray]] — Kadane's adapted to track running min *and* max (negatives flip sign)

## Full guide
- [[Job Search/Neetcode/01. Questions/15. Greedy/0.GreedyGuide|Greedy Guide]]
- [[Job Search/Neetcode/01. Questions/13. 1-D Dynamic Programming/0.1DDynamicProgrammingGuide|1-D Dynamic Programming Guide]]
