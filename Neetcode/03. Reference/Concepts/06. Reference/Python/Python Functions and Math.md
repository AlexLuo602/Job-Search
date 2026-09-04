---
type: concept
tags: ["concept"]
---

# Python Functions and Math

## Memoization

### Explicit dictionary cache

```python
def solve(start_state):
    memo = {}

    def dp(state):
        if state in memo:
            return memo[state]

        memo[state] = solve_smaller_states(state)
        return memo[state]

    return dp(start_state)
```

Cache keys must be hashable. Convert a list or dictionary state into a tuple, frozenset, string, or smaller hashable summary. Keep the dictionary inside the outer solution function so separate test cases do not share entries.

→ [[Memoization]], [[01.ClimbingStairs|ClimbingStairs]], [[08.CoinChange|CoinChange]], [[10.WordBreak|WordBreak]]

## functools

### `reduce`

```python
from functools import reduce
reduce(lambda acc, x: acc ^ x, nums, 0)   # XOR all elements
```

---

## math

```python
import math

math.inf              # same as float('inf') — use either
math.ceil(x)          # round up — O(1)
math.floor(x)         # round down — O(1); same as x // 1 for positive
math.isqrt(n)         # integer square root — no float precision issues vs int(math.sqrt(n))
math.gcd(a, b)        # O(log min(a,b))
math.gcd(*lst)        # Python 3.9+ — GCD of a list
math.lcm(a, b)        # Python 3.9+
math.log(x, base)     # O(1)
math.comb(n, k)       # C(n,k) — Python 3.8+
```

### Python integer gotchas

```python
# Python integers NEVER overflow — arbitrary precision
2 ** 1000             # works fine

# Division behavior differs from C++/Java
7 // 2      # 3   (floors toward -inf, not toward zero)
-7 // 2     # -4  (not -3!)
int(-7 / 2) # -3  (use this when you want truncation toward zero)

# Modulo is always non-negative for positive divisor
-1 % 5      # 4   (not -1 like in C++)
```
