---
type: concept
tags: ["concept"]
---

# Prefix Sum

**TL;DR:** Precompute cumulative sums once so any subarray sum becomes a single subtraction instead of a re-scan.

## When to reach for it
- Problem asks for the sum of a subarray or range, repeatedly, over the same array.
- Need to count subarrays whose sum equals k (pair prefix sums with a hash map).
- 2D grid problems asking for rectangle sums.
- You notice you'd otherwise recompute overlapping sums from scratch for every query.

## How it works
Build an array `prefix` where `prefix[i]` is the sum of the first `i` elements (`prefix[0] = 0`). Any range sum `nums[l..r]` is then `prefix[r+1] - prefix[l]` — the sum "up to r" minus the sum "up to l", which cancels out everything before index `l`. Trace the build on `nums = [1, 2, 3, 4, 5]`:

| i | nums[i] | prefix[i+1] = prefix[i] + nums[i] |
|---|---|---|
| 0 | 1 | prefix[1] = 0 + 1 = 1 |
| 1 | 2 | prefix[2] = 1 + 2 = 3 |
| 2 | 3 | prefix[3] = 3 + 3 = 6 |
| 3 | 4 | prefix[4] = 6 + 4 = 10 |
| 4 | 5 | prefix[5] = 10 + 5 = 15 |

So `prefix = [0, 1, 3, 6, 10, 15]`. Sum of `nums[1..3]` (`2+3+4=9`) is `prefix[4] - prefix[1] = 10 - 1 = 9`. For "count subarrays summing to k," instead of precomputing the whole array up front, scan once, keep a running sum, and ask a hash map "have I seen `cur_sum - k` before?" — every earlier prefix value that matches marks a subarray ending here that sums to k.

## Why it works
The invariant is definitional: `prefix[i] = nums[0] + ... + nums[i-1]`, so `prefix[r+1] - prefix[l] = nums[l] + ... + nums[r]` — the terms before index `l` appear in both prefix sums and cancel exactly. That's why one O(n) precomputation replaces what would otherwise be O(n) re-summation *per query*. The hash-map variant follows the same cancellation: `prefix[j] - prefix[i] == k` rearranges to `prefix[i] == prefix[j] - k`, so counting subarrays ending at `j` that sum to k is the same as counting how many earlier prefix values equal `prefix[j] - k` — which a hash map answers in O(1) instead of scanning all earlier indices.

## Template
```python
# 1-D prefix sum
nums = [1, 2, 3, 4, 5]
prefix = [0] * (len(nums) + 1)
for i, x in enumerate(nums):
    prefix[i + 1] = prefix[i] + x

# Sum of nums[l..r] inclusive:
def range_sum(l, r):
    return prefix[r + 1] - prefix[l]

# Count subarrays summing to k (hash map variant)
from collections import defaultdict
count, cur_sum = 0, 0
seen = defaultdict(int)
seen[0] = 1
for x in nums:
    cur_sum += x
    count += seen[cur_sum - k]
    seen[cur_sum] += 1
```

## Complexity
Time: O(n) to build the prefix array, then O(1) per range-sum query — the whole point is amortizing the O(n) cost of a naive re-sum across all future queries. Space: O(n) for the prefix array (or for the running hash map in the streaming variant).

## Common pitfalls
- Off-by-one: forgetting `prefix[0] = 0` (the empty prefix) or mixing up whether `prefix[i]` includes `nums[i]`.
- Forgetting to seed the hash map with `seen[0] = 1` before the loop — without it, a subarray starting at index 0 is never counted.
- Using the hash-map streaming variant when a plain precomputed array with O(1) direct range queries would be simpler and clearer.
- Rebuilding or reusing the prefix array after the underlying array mutates, forgetting it's now stale.

## NeetCode examples
- [[06.ProductOfArrayExceptSelf|ProductOfArrayExceptSelf]] — prefix and suffix product arrays (same cancellation idea, multiplication)
- [[09.MaxProductSubArray|MaxProductSubArray]] — contrast case: a [[Kadane's Algorithm|Kadane]]-style running min/max scan, not prefix cancellation. Products cannot be "subtracted out" the way prefix sums can.

## Full guide
[[Job Search/Neetcode/01. Questions/01. Arrays & Hashing/0.ArraysAndHashingGuide|Arrays & Hashing Guide]]
