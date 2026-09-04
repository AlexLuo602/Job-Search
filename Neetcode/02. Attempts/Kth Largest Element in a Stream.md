---
question: "[[01.kthLargestElementInAStream|KthLargestElementInAStream]]"
topic: ["Heap"]
lc_difficulty: Easy
tags: ["neetcode-150"]
attempt_date: 2026-07-04
my_difficulty: Easy
status: Should Redo
time_min: 10
review_concepts: ["Heap"]
---
# Kth Largest Element in a Stream

_Maintain a min-heap of size k to track the k largest elements seen so far; the root will always be the k-th largest._

## My Approach

I reached for a min-heap (priority queue) to keep track of the largest k elements in the stream. By constraining the heap to never exceed k elements, the smallest element in that subset (which sits at the root) is guaranteed to be the k-th largest overall.

During initialization and the `add` method, I push elements onto the heap until it reaches size k. Once full, I compare incoming elements to the heap's root. If an element is larger than the root, it belongs in the top k, so I pop the root and push the new element. If it's smaller, it isn't part of the top k, so I simply ignore it.

## Complexity

|**Complexity**|**Why**|
|---|---|
|Time O(N log k)|Initialization processes N items at O(log k) each. The `add` method takes O(log k) to maintain the heap.|
|Space O(k)|The min-heap stores exactly k elements at any given time.|

## Key Insight

When you need to find the "largest" elements in a stream, you use a "min-heap". By keeping only k elements in a min-heap, every element smaller than the top k is systematically discarded. Because the heap naturally bubbles the smallest of its contents to the root, checking or removing the current k-th largest item becomes an O(1) lookup and O(log k) removal, saving you from sorting the entire stream.

## Mistakes / Gaps

1. **Heap capacity logic** — I initially forgot to handle the case where the heap is already full (length >= k) but the new value is smaller than the root. It needs to be explicitly ignored, which I successfully fixed with the `elif self.heap[0] < val` condition before submitting.

## Code

```python
class KthLargest:

    def __init__(self, k: int, nums: List[int]):
        self.heap = []
        self.k = k
        
        for num in nums:
            if len(self.heap) < k:
                heapq.heappush(self.heap, num)
            elif self.heap[0] < num:
                heapq.heappop(self.heap)
                heapq.heappush(self.heap, num)

    def add(self, val: int) -> int:
        if len(self.heap) < self.k:
            heapq.heappush(self.heap, val)
        elif self.heap[0] < val:
            heapq.heappop(self.heap)        
            heapq.heappush(self.heap, val)

        return self.heap[0]
```

## Is My Solution Optimal?

Processing a stream element must take at least O(log k) time if we are maintaining a dynamically sorted state for k items, and we must allocate O(k) space to store them. **Yes, optimal.**

## Code Improvements

- **Built-in push-pop** — `heapq.heappushpop(self.heap, val)` executes the pop and push natively, avoiding the overhead and extra lines of separate calls.
- **Initialization efficiency** — Pushing elements one by one takes O(N log k). Sorting the array first or using `heapq.heapify()` on a slice can optimize the initialization step.
- **Redundant logic** — The logic inside `__init__` is identical to `add()`. We can just call `self.add(num)` in the loop to DRY up the code.

## Best Solution

```python
class KthLargest:
    def __init__(self, k: int, nums: List[int]):
        self.k = k
        self.heap = nums
        heapq.heapify(self.heap)
        
        # Pop elements until the heap is exactly size k
        while len(self.heap) > k:
            heapq.heappop(self.heap)

    def add(self, val: int) -> int:
        if len(self.heap) < self.k:
            heapq.heappush(self.heap, val)
        elif val > self.heap[0]:
            heapq.heappushpop(self.heap, val)
            
        return self.heap[0]
```

This version speeds up initialization by leveraging `heapq.heapify()`, which transforms the initial list into a heap in O(N) time. It then pops the excess smallest elements. The `add` method utilizes `heappushpop` to elegantly and efficiently handle incoming values larger than the root without separate method calls.