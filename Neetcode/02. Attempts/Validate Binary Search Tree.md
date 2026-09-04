---
question: "[[11.ValidateBinarySearchTree|ValidateBinarySearchTree]]"
topic: ["Tree"]
lc_difficulty: Medium
tags: ["neetcode-150"]
attempt_date: 2026-06-24
my_difficulty: Easy
status: Done
time_min: 5
review_concepts: ["DFS"]
---
# Validate Binary Search Tree

_Recursively verify that every node falls strictly within its dynamically updated min and max allowed values._

## My Approach

I reached for a top-down Depth-First Search (DFS) because validating a BST requires more than just checking a node against its immediate children. A valid BST requires that a node satisfies the constraints established by *all* of its ancestors.

The core logic relies on a recursive helper function that tracks the valid range `(lm, rm)` for the current node. If a node's value falls outside this strictly increasing range, the tree is invalid. When traversing into the left subtree, the current node's value becomes the new strict upper bound (`rm`). When traversing into the right subtree, the current node's value becomes the new strict lower bound (`lm`).

By passing these bounds down the recursion stack, I ensured that every node is globally valid relative to the entire path from the root.

## Complexity

|**Complexity**|**Why**|
|---|---|
|Time O(N)|Every single node in the tree must be visited to confirm full validity.|
|Space O(H)|The recursive call stack depth is proportional to the tree's height, scaling from O(log N) for balanced trees to O(N) for skewed trees.|

## Key Insight

A local check (`node.left.val < node.val < node.right.val`) is a common trap because a right child deep inside a left subtree might still exceed the global root, violating the BST property. The conceptual unlock is to pass down a dynamic "valid window" from parent to child. As you traverse down the tree, moving left tightens the ceiling, and moving right tightens the floor.

## Mistakes / Gaps

None this attempt.

## Code

```python
class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        def recurse(node, lm, rm):
            if not node:
                return True

            if node.val <= lm or node.val >= rm:
                return False

            return recurse(node.left, lm, node.val) and recurse(node.right, node.val, rm)
        
        return recurse(root, float('-inf'), float('inf'))
```

## Is My Solution Optimal?

Every node must be checked to guarantee validity, making O(N) time the theoretical floor. O(H) space is required to track the state down the branches. My recursive DFS approach perfectly matches both lower bounds. **Yes, optimal.** ## Code Improvements

- **Variable naming** — `lm` and `rm` are slightly ambiguous. Using `min_val` and `max_val`, or `lower` and `upper`, instantly clarifies their mathematical purpose.
- **Math module import** — Using `math.inf` is often preferred over `float('inf')` in modern Python for readability and slight performance improvements during instantiation, though both are perfectly valid.

## Best Solution

```python
import math

class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        def dfs(node, min_val, max_val):
            if not node:
                return True
            
            if not (min_val < node.val < max_val):
                return False
                
            return dfs(node.left, min_val, node.val) and dfs(node.right, node.val, max_val)

        return dfs(root, -math.inf, math.inf)
```

This is functionally identical to my submitted approach, retaining the optimal O(N) time and O(H) space complexities. It is the "best" solution simply due to the more idiomatic Python chained comparison (`min_val < node.val < max_val`) and explicitly descriptive variable names, which makes the logic instantly readable.