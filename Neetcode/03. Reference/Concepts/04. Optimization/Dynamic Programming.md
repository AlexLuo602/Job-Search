---
type: concept
tags: ["concept"]
---

# Dynamic Programming

**TL;DR:** Break a problem into overlapping subproblems, solve each exactly once, cache the result, and combine cached answers instead of recomputing them.

## When to reach for it
- Optimal substructure: the best answer to the whole problem is built from best answers to smaller versions of the same problem.
- Overlapping subproblems: a plain recursive solution calls itself with the *same* arguments many times.
- Keywords: "number of ways", "minimum/maximum cost", "is it possible to reach/partition", "longest/shortest subsequence".
- A brute-force recursion has exponential time, but the number of *distinct* argument combinations is small (polynomial).

## How it works
Design DP in four steps, in this order:

1. **State** — decide what `dp[i]` (or `dp[i][j]`) means in words first, e.g. "number of ways to climb to step i." Every quantity the answer depends on must appear as a dimension.
2. **Transition** — write the recurrence relating `dp[i]` to smaller states, e.g. `dp[i] = dp[i-1] + dp[i-2]`.
3. **Base cases** — the smallest states you can answer without recursing.
4. **Order** — decide what order to fill states so every dependency is already computed. Top-down ([[Memoization|memoization]]) gets this for free via the call stack. Bottom-up ([[Tabulation|tabulation]]) requires iterating in an order that respects the recurrence (increasing `i`, or diagonal for two-string problems).

**Trace — Climbing Stairs, n = 5** (`dp[i]` = ways to reach step `i`, moving 1 or 2 steps at a time, `dp[0] = dp[1] = 1`):

| i | dp[i-2] | dp[i-1] | dp[i] = dp[i-1] + dp[i-2] |
|---|---|---|---|
| 0 | — | — | 1 (base) |
| 1 | — | — | 1 (base) |
| 2 | 1 | 1 | 2 |
| 3 | 1 | 2 | 3 |
| 4 | 2 | 3 | 5 |
| 5 | 3 | 5 | 8 |

Answer: `dp[5] = 8`. Each row only reads the previous two rows — that observation is exactly what licenses **space rolling**: replace the `dp` array with two scalars `prev2, prev1` and drop the array entirely, since nothing older than `i-2` is ever touched again.

## Why it works
Two properties must both hold, and the trace above demonstrates each one:

- **Overlapping subproblems**: naive recursion `climb(n) = climb(n-1) + climb(n-2)` recomputes `climb(3)` once *for* `climb(5)`'s left branch and once *for* `climb(4)`'s right branch — the call tree branches but keeps revisiting the same states, giving `O(2^n)` calls over only `n + 1` distinct states. Caching each state's answer the first time collapses this to `O(n)` total work.
- **Optimal substructure**: the number of ways to reach step 5 truly is the sum of the ways to reach step 4 and step 3 — any path to step 5 takes its last move from one of those two steps, so the subproblem answers combine into the full answer with no correction term needed. If a problem's globally best combination of subproblems could still be beaten by a locally suboptimal choice at some smaller step, plain DP doesn't apply.

[[Memoization]] (top-down) and [[Tabulation]] (bottom-up) use the same recurrence in opposite directions. Memoization discovers needed states through recursion, while tabulation commits to a fill order before the loop starts.

## Template
```python
# Top-down (memoization)
memo = {}

def dp(i, *state):
    key = (i, *state)
    if key in memo:
        return memo[key]
    if base_case(i):
        return base_value

    memo[key] = combine(dp(i - 1, ...), dp(i - 2, ...))
    return memo[key]

# Bottom-up (tabulation) — 1-D example
n = len(nums)
table = [0] * (n + 1)
table[0] = base_case_0
table[1] = base_case_1
for i in range(2, n + 1):
    table[i] = transition(table[i - 1], table[i - 2], nums[i - 1])
return table[n]

# Space-optimized: only the last two states are ever read
prev2, prev1 = base0, base1
for i in range(2, n + 1):
    prev2, prev1 = prev1, prev1 + prev2
return prev1
```

## Complexity
Time: O(states × transition cost) — each distinct state is computed once, so total work is the number of distinct `(i, ...)` combinations times the O(1) or O(k) cost to combine smaller answers, not the exponential call count of naive recursion.
Space: O(state space) for the cache/table, often reducible to O(1) or O(n) when the recurrence only reaches back a fixed number of prior states (see rolling variables above).

## Common pitfalls
- Not identifying the correct state — omitting a dimension the answer actually depends on (e.g., remaining capacity in knapsack) silently produces a wrong recurrence, not a crash.
- Wrong base cases or an off-by-one between `dp[n]` and `dp[n-1]` as the final answer.
- Iterating tabulation in the wrong direction — 0/1 knapsack must fill capacity *backward* per item to avoid reusing an item twice; unbounded knapsack (like coin change) fills forward on purpose.
- Writing top-down recursion without an explicit cache lookup and write silently degrades back to exponential time with none of DP's benefit.

## NeetCode examples
- [[01.ClimbingStairs|ClimbingStairs]] — the Fibonacci-style 1-D DP traced above
- [[08.CoinChange|CoinChange]] — unbounded knapsack (minimum coins)
- [[11.LongestIncreasingSubsequence|LongestIncreasingSubsequence]] — O(n²) DP or O(n log n) with patience sort
- [[02.LongestCommonSubsequence|LongestCommonSubsequence]] — 2-D DP on two strings
- [[03.HouseRobber|HouseRobber]] — rolling variable optimization

## Full guide
- [[Job Search/Neetcode/01. Questions/13. 1-D Dynamic Programming/0.1DDynamicProgrammingGuide|1-D DP Guide]]
- [[Job Search/Neetcode/01. Questions/14. 2-D Dynamic Programming/0.2DDynamicProgrammingGuide|2-D DP Guide]]
