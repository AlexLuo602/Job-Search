---
type: concept
tags: [concept, dsa, pattern]
---

# Tabulation

**TL;DR:** Solve dynamic programming states from the smallest base cases upward, storing each answer in a table before a later state needs it.

## When to reach for it

- You already know the recurrence and can order the states so every dependency is computed first.
- A recursive [[Memoization|memoized]] solution is correct, but you want to avoid recursion depth or function-call overhead.
- The recurrence only reads a small number of earlier states, so the table can later be reduced to a row or a few rolling variables.
- You need precise control over iteration direction, such as backward capacity updates for 0/1 knapsack or forward updates for unbounded knapsack.

## How it works

1. Define what each table entry means.
2. Initialize the smallest states that do not depend on other entries.
3. Visit the remaining states in dependency order.
4. Apply the recurrence once per state.
5. Return the entry that represents the original problem.

For Fibonacci, only `dp[0]` and `dp[1]` are base cases. Every later value follows the recurrence:

```python
def fib(n: int) -> int:
    if n <= 1:
        return n

    dp = [0] * (n + 1)
    dp[1] = 1

    for i in range(2, n + 1):
        dp[i] = dp[i - 1] + dp[i - 2]

    return dp[n]
```

`dp[2]` is not a base case. The loop computes it first as `dp[1] + dp[0]`.

The [[Job Search/Neetcode/01. Questions/13. 1-D Dynamic Programming/0.1DDynamicProgrammingGuide|1-D DP Guide]] includes an interactive `fib()` walkthrough that shows the fill order and active dependencies.

## Fill order

The correct direction follows the recurrence:

| State shape | Typical order | Reason |
| --- | --- | --- |
| `dp[i]` reads smaller indices | left to right | earlier indices must exist first |
| Grid cell reads top and left | row by row | both dependencies are already filled |
| Interval `dp[left][right]` reads shorter ranges | increasing interval length | shorter intervals must exist first |
| 0/1 knapsack capacity | right to left per item | prevents reusing the current item |
| Unbounded knapsack capacity | left to right per item | allows the current item to repeat |

A wrong iteration direction can change the recurrence even when the formula looks correct.

## Why it works

Each table entry stores the recurrence's answer for one state. Base cases are correct directly. If every dependency for the current state is already correct, applying the recurrence makes the current entry correct too. Filling states in dependency order extends that argument until the table contains the answer to the original problem.

## Space optimization

A full table is unnecessary when each state only reads a fixed-size region of earlier states.

- Two previous values can become two rolling variables.
- A 2-D table that only reads the previous row can become one or two rows.
- In-place updates are safe only when the iteration direction preserves the intended old and new values.

Keep the full table first when deriving or debugging the recurrence. Reduce the space only after the dependency pattern is clear.

## Complexity

Time: O(number of states x transition cost). Each state is filled once.

Space: O(number of stored states), often reducible when old states are no longer needed.

## Common pitfalls

- Treating a value such as `dp[2]` as a base case even though the recurrence can compute it from smaller states.
- Filling states before their dependencies are ready.
- Iterating capacity in the wrong direction and accidentally changing whether an item can be reused.
- Updating a rolling row in place before saving a value that a later cell still needs.
- Allocating the table before defining what each entry represents.

## Related concepts

- [[Dynamic Programming]]
- [[Memoization]]
- [[Space-Time Tradeoff]]

## NeetCode examples

- [[01.ClimbingStairs|ClimbingStairs]]: left-to-right 1-D fill, reducible to two variables
- [[08.CoinChange|CoinChange]]: minimum over previously solved amounts
- [[12.PartitionEqualSubsetSum|PartitionEqualSubsetSum]]: right-to-left 0/1 capacity updates
- [[02.LongestCommonSubsequence|LongestCommonSubsequence]]: row-by-row two-sequence table
- [[10.BurstBalloons|BurstBalloons]]: increasing interval-length fill

## Full guides

- [[Job Search/Neetcode/01. Questions/13. 1-D Dynamic Programming/0.1DDynamicProgrammingGuide|1-D DP Guide]]
- [[Job Search/Neetcode/01. Questions/14. 2-D Dynamic Programming/0.2DDynamicProgrammingGuide|2-D DP Guide]]
