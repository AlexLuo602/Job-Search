---
question: "[[04.GasStation]]"
topic:
  - Greedy
  - Array
lc_difficulty: Medium
tags:
  - neetcode-150
attempt_date: 2026-08-10
my_difficulty: Trick
status: Gave Up (Redo)
time_min: 12
num_mistakes: 1
review_concepts:
  - Greedy
---
# Gas Station

_If total gas is sufficient, a valid start is guaranteed, meaning we can find it in a single pass by resetting our starting index whenever the tank drops below zero._

## My Approach

I recognized the second core Greedy observation right away: if starting at station $A$ fails at station $B$, no station between $A$ and $B$ can reach $B$ either. Based on this, I initially considered an $O(2N)$ approach using modulo arithmetic to simulate a full circular loop.

However, I missed the implications of the first observation: checking if the total gas is less than the total cost up front. I thought about it briefly but didn't go deeper. After peeking at the solution, I realized that if `sum(gas) >= sum(cost)`, a valid solution is mathematically guaranteed to exist. This means we don't need to simulate the wrap-around with a modulo cycle at all; a single $O(N)$ pass that pushes the `start` pointer forward on failure will naturally end up on the correct starting station.

## Complexity

|**Complexity**|**Why**|
|---|---|
|Time O(N)|Calculating the sum takes one full pass, and the greedy iteration takes a second full pass. $O(2N)$ simplifies to $O(N)$.|
|Space O(1)|Only keeping track of three integer variables (`start`, `remaining`, and the sums).|

## Key Insight

The global sum check (`sum(gas) >= sum(cost)`) guarantees a solution exists. Because a solution is guaranteed, a single pass that pushes the `start` pointer forward whenever the tank drops below zero will inevitably land on the correct starting station by the end of the array. You don't actually need to simulate the wrap-around because the global check already proves the chosen start will survive the rest of the circle.

## Mistakes / Gaps

1. **Missed the global invariant** — I understood the greedy skip logic but missed that doing a total sum check up front mathematically eliminates the need to simulate the circular route with modulo. This was a conceptual gap that I corrected after reviewing the solution, allowing me to avoid writing the $O(2N)$ cycle logic entirely.
	- For example, I would have done the modulo approach, and done a complete cycle even if starting index would somewhere at the end of the list, leading to repeated operations.

## Code

```python
class Solution:
    def canCompleteCircuit(self, gas: List[int], cost: List[int]) -> int:
        if sum(cost) > sum(gas):
            return -1

        start = 0
        remaining = 0
        
        for i in range(len(gas)):
            remaining += gas[i] - cost[i]

            if remaining < 0:
                remaining = 0
                start = i + 1
        
        return start
```

## Is My Solution Optimal?

Every gas station's cost and supply must be evaluated to determine if a circuit is possible, making $O(N)$ time the strict theoretical floor. The state requires only a few integers, hitting the $O(1)$ space floor. **Yes, optimal.**

## Code Improvements

None — code is already perfectly clean and idiomatic.

## Best Solution

```python
class Solution:
    def canCompleteCircuit(self, gas: List[int], cost: List[int]) -> int:
        if sum(cost) > sum(gas):
            return -1

        start = 0
        remaining = 0
        
        for i in range(len(gas)):
            remaining += gas[i] - cost[i]

            if remaining < 0:
                remaining = 0
                start = i + 1
        
        return start
```

Your submitted code is the canonical optimal solution. It relies purely on the two greedy observations to avoid nested loops and unnecessary modulo arithmetic, achieving the $O(N)$ lower bound flawlessly.