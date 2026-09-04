---
question: "[[02.MaxDepthOfBinaryTree|MaxDepthOfBinaryTree]]"
topic: ["Tree"]
lc_difficulty: Easy
my_difficulty: Easy
status: Done
time_min: 5
tags:
  - neetcode-150
attempt_date: 2026-06-17
review_concepts: []
---
# Maximum Depth of Binary Tree

_Find the longest path from the root node to the farthest leaf node by either tracking the recursive call depth or counting iterative queue batches._

## My Approach

I implemented two distinct solutions for this problem: a recursive Depth-First Search (DFS) and an iterative Breadth-First Search (BFS).

For the DFS approach, I used a helper function to pass the current level down the call stack. By maintaining a `nonlocal` depth variable, I updated the maximum depth recorded every time the recursion reached a valid node.

For the BFS approach, I used a `deque` to process the tree strictly level by level. By capturing the `level_size` at the beginning of the `while` loop, I ensured that the inner `for` loop only processed nodes belonging to the current depth. I incremented a `levels` counter for each full batch processed.

## Complexity

|**Complexity**|**Why**|
|---|---|
|Time O(N)|Every node in the tree is visited exactly once in both the DFS and BFS approaches.|
|Space O(N)|In the worst case, the DFS call stack goes O(N) deep (skewed tree), and the BFS queue holds up to N/2 nodes at the leaf level (balanced tree).|

## Key Insight

The maximum depth of a binary tree is equivalent to the number of levels it contains. DFS solves this by diving all the way to the bottom and keeping a "high-water mark" of the deepest step taken. BFS solves this by peeling the tree layer by layer, counting each layer as it goes. Recognizing that a queue's batch size perfectly mirrors a tree's physical level is essential for mapping 2D tree structures into 1D iterative loops.

## Mistakes / Gaps

1. **Queue ordering:** Accidentally used `append` instead of `appendleft` when adding children during BFS, which corrupted the FIFO behavior required for level-order traversal; fixing the enqueue/dequeue logic restored the correct layer-by-layer processing.
    

## Code

Python

```
# Method 1: DFS (Recursive)
class Solution:
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        depth = 0
        def recurse(r, l):
            if not r:
                return

            nonlocal depth
            
            recurse(r.left, l + 1)
            recurse(r.right, l + 1)
            depth = max(depth, l) # Update highest level seen

        recurse(root, 1)
        return depth

# Method 2: BFS (Iterative)
from collections import deque

class Solution:
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        levels = 0
        level = deque([])
        
        if root:
            level.append(root)

        while level:
            levels += 1
            level_size = len(level)

            # Process entire level batch at once
            for _ in range(level_size):
                node = level.pop()
                
                # Use appendleft to maintain FIFO queue behavior
                if node.left:
                    level.appendleft(node.left)
                if node.right:
                    level.appendleft(node.right)

        return levels
```