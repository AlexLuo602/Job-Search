---
question: "[[01.InvertBinaryTree|InvertBinaryTree]]"
topic: ["Tree"]
lc_difficulty: Easy
my_difficulty: Easy
status: Done
time_min: 5
tags:
  - neetcode-150
attempt_date: 2026-06-16
review_concepts: []
---
# Invert Binary Tree

_Swap the left and right children at every node, then recursively process the subtrees._

## My Approach

I reached for a Depth-First Search (DFS) recursive approach to traverse the tree. Since inverting a binary tree requires every single node's left and right pointers to be physically swapped, a recursive traversal fits the problem naturally.

The core logic uses a nested recursive function `invertRecurse(r)`. The base case handles empty subtrees by returning immediately. For the main step, I temporarily stored the left child, then overwrote the left pointer with the right child, and the right pointer with the stored left child.

After swapping the immediate children of the current node, I recursively invoked the function on those newly swapped subtrees to ensure the mirrored structure propagates all the way down to the leaf nodes.

## Complexity

|**Complexity**|**Why**|
|---|---|
|Time O(N)|Every single node in the tree is visited exactly once to perform the swap operation.|
|Space O(H)|The recursive call stack depth scales with the height of the tree (worst-case O(N) for skewed trees, O(log N) for balanced trees).|

## Key Insight

Tree inversion is a purely local operation. If you swap the left and right children of a parent node, and guarantee that the child subtrees are also individually inverted, the entire tree will be cleanly inverted. Because the problem exhibits optimal substructure, it breaks down perfectly into identical, smaller subproblems, making DFS recursion the ideal tool.

## Mistakes / Gaps

None this attempt.

## Code

Python

```
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def invertTree(self, root: Optional[TreeNode]) -> Optional[TreeNode]:
        def invertRecurse(r):
            if not r:
                return

            # Swap left and right child pointers
            temp = r.left
            r.left = r.right
            r.right = temp

            # Recursively invert the subtrees
            invertRecurse(r.left)
            invertRecurse(r.right)
        
        invertRecurse(root)
        return root
```