---
question: "[[03.JumpII]]"
topic:
  - Array
  - Dynamic Programming
  - Greedy
lc_difficulty: Medium
tags:
  - neetcode-150
attempt_date: 2026-07-29
my_difficulty: Medium
status: Done
time_min: 15
review_concepts:
  - Greedy
  - BFS
---
# Jump Game II

_Treat the array as a tree and use an implicit Breadth-First Search (BFS) to find the shortest path by evaluating the maximum reach of each contiguous "level."_

## My Approach

I used a Greedy/BFS hybrid approach to find the minimum number of jumps. Instead of arbitrarily picking the largest jump right away, I evaluated "windows" of reachable indices to find the absolute furthest distance I could achieve on the *next* jump.

The core logic relies on simulating a level-by-level traversal. The `furthest` variable defines the right boundary of the current level. Inside the `while` loop, I established the `end` boundary for the current jump. The inner `for` loop then iterates through all indices within this current window, updating `furthest` with the maximum possible reach. 

Once the entire current level has been processed, it represents exactly one jump taken, so I incremented the `jumps` counter. The outer loop continues until the calculated maximum reach meets or exceeds the final index.

## Complexity

|**Complexity**|**Why**|
|---|---|
|Time O(N)|Despite the nested loops, the variable `i` continuously advances forward, ensuring each element in the array is visited exactly once.|
|Space O(1)|The solution only requires a few integer variables (`jumps`, `furthest`, `i`, `end`) to track window boundaries and jump counts.|

## Key Insight

Finding the minimum number of steps to reach a goal is fundamentally a shortest-path problem, which BFS solves optimally. In a 1D array where edges are jumps, we don't need a heavy queue data structure. The "levels" of the BFS tree are simply contiguous subarrays. By tracking the left and right pointers of our current level, we can scan it to find the furthest possible right pointer for the next level, effectively walking the graph in O(N) time.

## Mistakes / Gaps

None this attempt.

## Code