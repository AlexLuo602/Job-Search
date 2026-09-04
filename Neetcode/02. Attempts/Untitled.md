---
question: "[[14.BinaryTreeMaximumPathSum|BinaryTreeMaximumPathSum]]"
topic:
  - Tree
lc_difficulty: Hard
tags:
  - neetcode-150
attempt_date: 2026-06-22
my_difficulty: Medium
status: Done
time_min: 10
review_concepts:
  - DFS
---
# Binary Tree Maximum Path Sum

_Find the maximum path sum by traversing bottom-up, updating a global max with the local "split" path while returning only the highest single branch to the parent._

## My Approach

I used a bottom-up Depth-First Search (DFS) to process the tree recursively. The core requirement is that a valid path can only branch once at its highest point. Therefore, the recursive function must return the maximum *single-branch* sum extending down from the current node, allowing its parent to continue the path.

At each node, I recursively determine the maximum path sums from the left and right children. To find the highest possible sum spanning across the current node, I evaluate four scenarios: just the node itself, the node plus the left branch, the node plus the right branch, and the node connecting both left and right branches.

I track the absolute maximum path found anywhere in the tree using a `nonlocal` variable. After comparing the local peak against the global maximum, I return the best single continuous path (the node alone, or the node plus its best positive branch) upward to the caller.

## Complexity

|**Complexity**|**Why**|
|---|---|
|Time O(N)|Every single node in the tree is visited exactly once during the DFS traversal to calculate its local sums.|
|Space O(H)|The recursive call stack depth scales with the height of the tree (worst-case O(N) for skewed trees, O(log N) for balanced trees).|

## Key Insight

A path through a tree can only ever split *once* at its highest node (the "root" of that specific path). You must decouple the local check from the return value: update the global answer by assuming the current node is the split point (allowing both left and right branches), but only return a straight, unbranched line to the parent. Additionally, recognizing that paths can be negative is crucial; if a child branch yields a negative sum, it is always mathematically optimal to completely abandon it and start a fresh path at the current node.

## Mistakes / Gaps

1. Negative Path Trap — I almost forgot to handle scenarios where the left or right subtrees yield negative sums. I caught it and addressed it by explicitly evaluating combinations that exclude the children (e.g., taking just `cur`), ensuring I never drag down a path sum by adding a negative branch.

## Code

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def maxPathSum(self, root: Optional[TreeNode]) -> int:
        maxPath = root.val

        # returns max path from one branch to r
        def recurse(r):
            nonlocal maxPath
            
            if not r:
                return 0
            
            left = recurse(r.left)
            right = recurse(r.right)
            cur = r.val

            curMaxPath = max(cur, cur + left, cur + right, cur + left + right)
            maxPath = max(curMaxPath, maxPath)

            return max(cur, cur + left, cur + right)
        
        recurse(root)
        return maxPath
```
