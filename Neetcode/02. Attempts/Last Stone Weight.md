---
question: "[[02.LastStoneWeight]]"
topic:
  - Heap
lc_difficulty: Easy
tags:
  - neetcode-150
attempt_date: 2026-07-29
my_difficulty: Easy
status: Done
time_min: 8
review_concepts:
  - Heap
---
# Last Stone Weight

_Continuously smash the two largest stones together by maintaining a max-heap._

## My Approach

I recognized that the problem requires repeatedly extracting the two maximum elements from a dynamically changing collection. A max-heap (Priority Queue) is the ideal data structure for this pattern. 

I initialized a max-heap with all the stones. Inside a `while` loop that runs as long as there is more than one stone left, I popped the top two heaviest stones. If their weights are not equal, the remaining weight (`y - x`) is pushed back into the heap. Finally, if the heap is empty I return 0; otherwise, I return the single remaining stone.

## Complexity

|**Complexity**|**Why**|
|---|---|
|Time O(N log N)|Heapifying the initial array takes O(N). Each of the N worst-case loop iterations performs up to three heap operations (two pops, one push) taking O(log N) each.|
|Space O(N)|Storing the elements in the heap structure requires O(N) space.|

## Key Insight

When a problem asks you to repeatedly access the maximum or minimum element while simultaneously inserting new elements, a priority queue is almost always the answer. A standard array sort would require an O(N) insertion step after every smash to keep the array sorted, leading to O(N²) time. A heap naturally maintains the sorted property dynamically in O(log N) per insertion.

## Mistakes / Gaps

None this attempt. 

## Code

```python
class Solution:
    def lastStoneWeight(self, stones: List[int]) -> int:
        heapq.heapify_max(stones)

        while len(stones) > 1:
            y = heapq.heappop_max(stones)
            x = heapq.heappop_max(stones)

            if y != x:
                heapq.heappush_max(stones, y - x)

        if len(stones) == 0:
            return 0
        else:
            return stones[0]
```

## Is My Solution Optimal?

We must process N stones and constantly re-evaluate the max after each dynamically generated difference, bounding us to O(N log N) operations for the comparisons and re-insertions. **Yes, optimal.**

## Code Improvements

- **Fictional `heapq` methods** — Python's standard `heapq` module only supports min-heaps natively. There is no public `heapify_max` or `heappush_max`. You must simulate a max-heap by negating the stone weights before pushing, and negating them again after popping.
- **List truthiness** — `if len(stones) == 0:` can be shortened to `return stones[0] if stones else 0` by relying on Python's implicit boolean evaluation of lists.

## Best Solution

```python
import heapq

class Solution:
    def lastStoneWeight(self, stones: List[int]) -> int:
        # Negate all values to simulate a max-heap using Python's min-heap
        max_heap = [-s for s in stones]
        heapq.heapify(max_heap)

        while len(max_heap) > 1:
            first = -heapq.heappop(max_heap)
            second = -heapq.heappop(max_heap)

            if first > second:
                heapq.heappush(max_heap, -(first - second))

        return -max_heap[0] if max_heap else 0
```

This represents the canonical, functioning Python implementation. It maintains the exact same O(N log N) time and O(N) space complexity but runs perfectly using the standard library. By explicitly negating the popped values into `first` and `second`, the mathematical difference (`first - second`) remains easy to read before being negated once more to be pushed back into the simulated max-heap.