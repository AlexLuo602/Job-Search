---
question: "[[06.SubtreeOfAnotherTree|SubtreeOfAnotherTree]]"
topic: ["Tree"]
lc_difficulty: Easy
tags: ["neetcode-150"]
attempt_date: 2026-06-21
my_difficulty: Easy
status: Done
time_min: 10
review_concepts: ["DFS"]
---
# Subtree of Another Tree

_Traverse the main tree and check if any node serves as the root of a tree structurally and numerically identical to the target subtree._

## My Approach

I built off the standard "Same Tree" pattern by nesting an `isSameTree` helper function inside my main logic. The core idea is to treat every single node in the main tree as a potential starting point and verify if the subtree extending from it matches `subRoot` exactly.

The main function, `isSubtree`, drives the broader traversal. At each node, it first checks if the tree rooted there is identical to `subRoot`. If it is, we've found our match and return `True`.

If they don't match, the base case kicks in: if the current `root` is null, we've hit the bottom without a match and return `False`. Otherwise, I recursively call `isSubtree` on both the left and right children, combining them with an `or` operator since the matching subtree could be hidden anywhere down either branch.

## Complexity

|**Complexity**|**Why**|
|---|---|
|Time O(N * M)|Where N is the number of nodes in `root` and M is the number of nodes in `subRoot`. In the worst-case scenario (e.g., skewed trees with identical values), we perform a full identical tree check taking O(M) time at every single node in the main tree.|
|Space O(H)|Where H is the height of the main tree. The recursive call stack for `isSubtree` scales with the height of `root`, dominating the space complexity.|

## Key Insight

This problem is cleanly solved by decoupling the logic into two separate DFS traversals: one to search through the main tree, and another to verify equality once a starting node is chosen. Recognizing this separation of concerns prevents overly complex single-pass recursive functions. You are essentially doing a brute-force search over all nodes in the main tree, running a standard `isSameTree` check at each step.

## Mistakes / Gaps

None this attempt.

## Code

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def isSubtree(self, root: Optional[TreeNode], subRoot: Optional[TreeNode]) -> bool:
        def isSameTree(r1, r2):
            if not r1 or not r2:
                if r1 or r2:
                    return False
                else:
                    return True

            if r1.val != r2.val:
                return False
            
            return isSameTree(r1.left, r2.left) and isSameTree(r1.right, r2.right)

        if isSameTree(root, subRoot):
            return True
        
        if not root:
            return False
        
        return self.isSubtree(root.left, subRoot) or self.isSubtree(root.right, subRoot)
```
