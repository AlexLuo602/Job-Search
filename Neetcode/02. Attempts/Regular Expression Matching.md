---
question: "[[11.RegularExpressionMatching]]"
topic:
  - Dynamic Programming
lc_difficulty: Hard
tags:
  - neetcode-150
attempt_date: 2026-07-27
my_difficulty: Hard
status: Should Redo
time_min: 40
review_concepts:
  - DFS
  - Dynamic Programming
---
# Regular Expression Matching

_Use top-down Dynamic Programming to branch into two paths when encountering a star wildcard: matching multiple characters or matching zero characters._

## My Approach

I reached for a top-down Dynamic Programming (DFS with memoization) approach. The standard characters and the `.` wildcard move both pointers linearly, but the `*` wildcard introduces branching paths (using the character vs skipping the pattern). A recursive DFS naturally handles this branching.

The core logic uses two pointers, `i` for the string `s` and `j` for the pattern `p`. When checking a character, if a `*` wildcard is detected at `p[j+1]`, the recursion branches. One path skips the `*` pattern entirely by advancing `j + 2`. The other path attempts to consume the current character in `s` (if it matches) by advancing `i + 1` while keeping `j` the same. 

To prevent redundant calculations on overlapping subproblems, I initialized a 2D array `mem` of size `(n + 1) x (m + 1)` to cache the boolean results of each `(i, j)` state.

## Complexity

|**Complexity**|**Why**|
|---|---|
|Time O(N * M)|There are N * M possible combinations of states for the pointers `i` and `j`, and each state evaluates in O(1) time once cached.|
|Space O(N * M)|The 2D memoization grid requires (N + 1) * (M + 1) space. The recursion call stack takes at most O(N + M) space.|

## Key Insight

The `*` wildcard acts as a conditional fork in the road. Eagerly trying to consume as many matching characters as possible using a greedy approach will fail for patterns like `s="aaa", p="a*a"`. Instead, DFS lets you simultaneously explore the universe where the `*` matches zero characters and the universe where it matches one more character. By caching these overlapping subproblems, the exponential branching tree is flattened into a polynomial 2D grid of states.

## Mistakes / Gaps

1. **Base Case Premature Halt** — Initially used `i == n - 1` as the base case. This failed to account for trailing wildcards in the pattern (like `a*b*`) that must still be processed down to zero after the string is completely exhausted.
2. **Index Out of Bounds** — Evaluated `s[i] == p[j]` without verifying `i < n` first. This caused an `IndexError` on empty strings or when the string was fully matched but the pattern wasn't.
3. **Memoization Array Sizing** — Initialized `mem` as `n x m`. State indices must reach exactly `n` and `m` to represent fully exhausted strings and patterns, requiring the grid to be strictly sized at `(n + 1) x (m + 1)`.

## Code

```python
class Solution:
    def isMatch(self, s: str, p: str) -> bool:
        n, m = len(s), len(p)

        mem = [[-1] * (m + 1) for _ in range(n + 1)]

        def dp(i, j):
            if j >= m:
                return i >= n
            
            if mem[i][j] != -1:
                return mem[i][j]
            
            if j < m - 1 and p[j + 1] == "*":
                if i < n and (p[j] == "." or s[i] == p[j]) and dp(i + 1, j):
                    mem[i][j] = True
                    return True
                
                mem[i][j] = dp(i, j + 2)
                return mem[i][j]
            
            if i < n and (p[j] == "." or s[i] == p[j]):
                mem[i][j] = dp(i + 1, j + 1)
                return mem[i][j]
            else:
                mem[i][j] = False
                return False

        return dp(0, 0)
```

## Is My Solution Optimal?

Every possible pair of indices between the string and the pattern might need to be evaluated in the worst-case scenario. Therefore, O(N * M) time is the theoretical lower bound. The memoization table requires O(N * M) space to store these states. **Yes, optimal.**

## Code Improvements

- **Explicit memoization**: Use a dictionary keyed by `(i, j)` so the cached state, lookup, and write remain visible.
- **Extracting match logic** — Creating a boolean `match = i < n and (s[i] == p[j] or p[j] == ".")` at the top of the `dp` function avoids writing the same bounds-checking and matching logic twice.

## Best Solution

```python
class Solution:
    def isMatch(self, s: str, p: str) -> bool:
        memo = {}

        def dfs(i: int, j: int) -> bool:
            key = (i, j)
            if key in memo:
                return memo[key]

            if j >= len(p):
                memo[key] = i >= len(s)
                return memo[key]

            match = i < len(s) and (s[i] == p[j] or p[j] == ".")

            if j + 1 < len(p) and p[j + 1] == "*":
                result = dfs(i, j + 2) or (match and dfs(i + 1, j))
            elif match:
                result = dfs(i + 1, j + 1)
            else:
                result = False

            memo[key] = result
            return result

        return dfs(0, 0)
```

This version keeps the `O(N * M)` time and space complexity while making the memoization mechanics explicit. The `(i, j)` tuple is the state key, every repeated state returns from `memo`, and each newly solved state is written before returning. The `match` boolean keeps the character comparison in one place.