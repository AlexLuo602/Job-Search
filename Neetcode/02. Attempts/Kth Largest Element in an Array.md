---
question: "[[04.KthLargestElementInAnArray|KthLargestElementInAnArray]]"
topic: ["Divide and Conquer"]
lc_difficulty: Medium
tags: ["neetcode-150"]
attempt_date: 2026-08-01
my_difficulty: Hard
status: Redo (Too Long)
time_min: 70
review_concepts: ["Two Pointers"]
---
# Kth Largest Element in an Array

_Find the target element in average linear time by repeatedly partitioning the array around a random pivot and discarding the irrelevant half._

## My Approach

I reached for Quickselect to find the kth largest element. I initialized `l` and `r` pointers to act as the range for the algorithm and calculated the target index as `len(nums) - k`. 

To decrease the chance of hitting the $O(N^2)$ worst-case, I chose a randomized pivot for each iteration. Inside a `while True` loop, I partitioned the array using a three-pointer approach (`small`, `cur`, `big`) so that all numbers smaller than the pivot ended up on the left, numbers equal to the pivot stayed in the middle, and numbers larger went to the right. 

After partitioning, I checked if the target index fell within the block of pivot values. If it did, I returned the value. If not, I updated the `l` or `r` boundaries to discard the irrelevant half of the array and repeated the process until the target was found.

## Complexity

|**Complexity**|**Why**|
|---|---|
|Time O(N)|On average, half the search space is discarded each iteration ($N + N/2 + N/4... \approx 2N$). Worst case is $O(N^2)$ but mathematically highly improbable with a random pivot.|
|Space O(1)|The array is partitioned entirely in-place using pointers.|

## Key Insight

Quickselect is just Quicksort without the "conquer" step. Because you only care about one specific index, you don't need to process both sides of the partition. Combining this with a randomized pivot and three-way partitioning (Dutch National Flag) completely neutralizes malicious test cases (like already-sorted arrays or arrays of all duplicates), keeping the runtime reliably at $O(N)$.

## Mistakes / Gaps

1. **Lost pivot reference** — I initially used the pivot index for value comparisons inside the loop instead of saving the pivot value, which caused comparisons against garbage data once elements started swapping.
2. **Duplicate worst-case** — My original two-pointer partition swept all duplicates into the "smaller" bucket, shrinking the search space by only one element per pass and causing a Time Limit Exceeded error. I fixed this by switching to a 3-way partition to isolate duplicates in the middle.

## Code

```python
class Solution:
    def findKthLargest(self, nums: List[int], k: int) -> int:
        l, r = 0, len(nums) - 1
        target = len(nums) - k

        while True:
            # choose pivot and swap it with num at end
            pivot = random.randint(l, r)
            val = nums[pivot]

            # now update list so all nums < pivot are on its left
            small, cur, big = l, l, r

            while cur <= big:
                if nums[cur] < val:
                    nums[small], nums[cur] = nums[cur], nums[small]
                    small += 1
                    cur += 1
                elif nums[cur] == val:
                    cur += 1
                else:
                    nums[cur], nums[big] = nums[big], nums[cur]
                    big -= 1

            """
            all nums[l, small - 1] are smaller than pivot
            all nums[small, right] are equal to pivot
            all nums[right+1, r] are bigger than pivot
            """

            if small <= target <= big:
                return val
            elif target < small:
                r = small - 1
            else:
                l = big + 1
```

## Is My Solution Optimal?

To find the $K$-th largest element without prior sorting, you must theoretically look at every element at least once, making $O(N)$ time the absolute floor. Operating entirely in-place hits the $O(1)$ space floor. My solution matches both bounds. 
**Yes, optimal.**

## Code Improvements

None — code is already clean.

## Best Solution

```python
class Solution:
    def findKthLargest(self, nums: List[int], k: int) -> int:
        l, r = 0, len(nums) - 1
        target = len(nums) - k

        while True:
            pivot = random.randint(l, r)
            val = nums[pivot]

            small, cur, big = l, l, r

            while cur <= big:
                if nums[cur] < val:
                    nums[small], nums[cur] = nums[cur], nums[small]
                    small += 1
                    cur += 1
                elif nums[cur] == val:
                    cur += 1
                else:
                    nums[cur], nums[big] = nums[big], nums[cur]
                    big -= 1

            if small <= target <= big:
                return val
            elif target < small:
                r = small - 1
            else:
                l = big + 1
```

Your submitted code is already the canonical best iterative implementation. While a recursive version of Quickselect exists, the iterative approach is superior because it guarantees $O(1)$ auxiliary space and entirely avoids the risk of call stack overflows on massive arrays.
