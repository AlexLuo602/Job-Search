---
question: "[[03.DiameterOfABinaryTree|DiameterOfBinaryTree]]"
topic: ["Tree"]
lc_difficulty: Easy
tags: ["neetcode-150"]
attempt_date: 2026-06-20
my_difficulty: Easy
status: Done
time_min: 7
review_concepts: ["DFS"]
---
# Diameter of Binary Tree

_The diameter is the maximum sum of left and right subtree heights passing through any node, computed dynamically during a bottom-up DFS._

## My Approach

I reached for a bottom-up Depth-First Search (DFS) approach. Since the longest path between any two nodes might not pass through the global root, the algorithm needs to evaluate every single node to see if the path arching through it is the maximum.

The core logic relies on a recursive function that natively computes the maximum depth (height) of a given subtree. At each node, it retrieves the left child's height and the right child's height. The local diameter arching through that specific node is simply the sum of those two heights.

For implementation, I used a `nonlocal` variable `diameter` to maintain the highest diameter seen so far across all recursive frames. This choice allows the recursion to fulfill its primary contract—returning the subtree height to its parent—while simultaneously updating the global maximum diameter in the background.

## Complexity

|**Complexity**|**Why**|
|---|---|
|Time O(N)|Every single node in the tree is visited exactly once during the recursive DFS traversal.|
|Space O(H)|The recursive call stack depth scales with the height of the tree (worst-case O(N) for skewed trees, O(log N) for balanced trees).|

## Key Insight

The longest path passing through a specific node is always the sum of the maximum heights of its left and right subtrees. By computing the height of the tree from the bottom up, you naturally have both child heights available at every single node. This means you can evaluate the local diameter at each step for "free" while solving a standard max-depth problem, capturing the global maximum along the way.

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
    def diameterOfBinaryTree(self, root: Optional[TreeNode]) -> int:
        diameter = 0

        # return height of subtree at r
        def recurse(r):
            if not r:
                return 0
            
            nonlocal diameter

            left_height = recurse(r.left)
            right_height = recurse(r.right)

            diameter = max(diameter, left_height + right_height)

            return max(left_height, right_height) + 1
        
        recurse(root)

        return diameter
```