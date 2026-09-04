---
question: "[[02.Jump]]"
topic:
  - Array
  - Greedy
lc_difficulty: Medium
tags:
  - neetcode-150
attempt_date: 2026-07-29
my_difficulty: Easy
status: Done
time_min: 5
review_concepts:
  - Greedy
---
# Jump Game

_Track the maximum reachable index at every step; if that boundary ever meets or exceeds the last index, the goal is reachable._

## My Approach

I used a forward-tracking Greedy approach to determine if the final index can be reached. Instead of simulating every possible sequence of jumps, this pattern simply maintains the maximum distance we can cover at any given point.

The core logic relies on a `furthest` variable. As I iterate through the array using `enumerate`, I calculate the maximum reach from the current position (`i + num`). I then update `furthest` to be the maximum of its previous value and this new reach. 

I implemented two early-exit conditions to optimize the loop. If `furthest` ever equals or exceeds the `goal` index, the end is reachable and the function immediately returns `True`. Conversely, if `furthest` equals the current index `i`, it means our maximum reach cannot extend past our current position (we landed on a zero with no momentum from earlier indices), so we return `False`.

## Complexity

|**Complexity**|**Why**|
|---|---|
|Time O(N)|The array is iterated at most once, performing constant-time maximum calculations and comparisons at each step.|
|Space O(1)|Only two integer variables (`goal` and `furthest`) are maintained regardless of the array's size.|

## Key Insight

You don't need to recursively simulate all valid jumps to see if a