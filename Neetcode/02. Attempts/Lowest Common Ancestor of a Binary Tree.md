---
question: "[[07.LowsetCommonAncestorOfABinarySearchTree]]"
topic:
  - Tree
lc_difficulty: Medium
tags:
  - neetcode-150
attempt_date: 2026-06-21
my_difficulty: Easy
status: Done
time_min: 10
review_concepts:
  - DFS
---
# Lowest Common Ancestor of a Binary Tree

_Traverse the tree using DFS and return the current node if it matches either target, or if the targets are found in its distinct subtrees._

## My Approach

First, to address your question: yes, your solution is perfectly optimal! It runs in O(N) time and O(H) space, which is the theoretical best you can do for an unstructured binary tree since you potentially have to search every node.

I expanded on the high-level design to build a bottom-up Depth-First Search (DFS). The core logic hinges on the fact that a node is the Lowest Common Ancestor (LCA) if `p` and `q` exist in entirely separate subtrees (left and right), or if the node itself is one of the targets and the other target is located further down one of its branches. 

The base cases are straightforward: if the node is `None`, return `None`. If the current node matches `p` or `q`, I immediately return that node. This cleanly handles the discovery of a target and naturally skips unnecessary further traversals down that branch (which elegantly solves the edge case where one target is a descendant of the other).

In the recursive step, I search the left and right subtrees. If both `ln` and `rn` return non-null nodes, it implies `p` and `q` are split across the left and right branches, meaning the current root is the LCA. If only one side returns a node, the found target (or the already-discovered LCA) is simply bubbled up the call stack.

## Complexity

|**Complexity**|**Why**|
|---|---|
|Time O(N)|In the worst-case scenario where the targets are deep leaves or don't exist, every node in the tree is visited exactly once.|
|Space O(H)|The recursive call stack depth is proportional to the height of the tree (O(N) for skewed trees, O(log N) for perfectly balanced trees).|

## Key Insight

By bubbling found targets up the recursive call stack, the LCA naturally identifies itself at the exact intersection point of the two paths. If a node receives non-null results from *both* its left and right children, it is the junction point (the LCA). Additionally, returning immediately upon finding `p` or `q` is a crucial optimization; if the other target happens to be a descendant of the found node, the found node is by definition the LCA, so we don't even need to keep searching.

## Mistakes / Gaps

None this attempt.

## Code

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution:
    def lowestCommonAncestor(self, root: 'TreeNode', p: 'TreeNode', q: 'TreeNode') -> 'TreeNode':
        def recurse(r):
            if not r:
                return None
            
            if r.val == p.val or r.val == q.val:
                return r
            
            ln = recurse(r.left)
            rn = recurse(r.right)

            if ln and rn:
                return r
            else:
                return ln or rn
        
        return recurse(root)
```
