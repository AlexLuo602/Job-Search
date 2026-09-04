---
question: "[[01.MaximumSubarray]]"
topic:
  - 1-D DP
  - Array
lc_difficulty: Medium
tags:
  - neetcode-150
attempt_date: 2026-07-29
my_difficulty: Trick
status: Done
time_min: 10
review_concepts:
  - Dynamic Programming
  - Greedy
---
# Maximum Subarray

_Keep a running sum of the subarray, but abandon it and start a new subarray if the current element alone is greater than the accumulated sum plus the current element._

## My Approach

I used Kadane's Algorithm to find the contiguous subarray with the largest sum. This pattern operates in a single pass by making a local greedy/dynamic programming choice at every single index.

The core logic relies on maintaining two variables: a running cumulative sum (`cum_sum`) and the maximum sum encountered so far (`global_max`). As I iterate through the array, at each step I must decide whether to append the current element to the existing running subarray or to start a fresh subarray directly at the current element.

I made this decision using `max(cum_sum + num, num)`. If adding the current number to the running total results in a value smaller than the number itself, it means the previous prefix was a net negative