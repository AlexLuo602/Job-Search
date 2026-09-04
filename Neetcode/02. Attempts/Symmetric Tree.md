---
question: "SymmetricTree (LC 101 — not in NeetCode 150)"
topic: ["Tree"]
lc_difficulty: Easy
tags: ["general"]
attempt_date: 2026-06-30
my_difficulty: Medium
status: Failed
time_min: 5
review_concepts: ["DFS"]
---
# Symmetric Tree

_Check if a tree is a mirror image of itself by simultaneously traversing its left and right subtrees in opposite directions._

## My Approach

I rushed the implementation without clearly defining what "symmetrical" structurally means for a binary tree. My initial approach used a single-pointer DFS that checked if a given node's immediate left and right children matched. It then recursed down the left and right branches completely independently. 

Because the recursion didn't cross-reference the two main subtrees, the logic was flawed. It essentially checked if *every individual node* had identical left and right children, rather than checking if the left half of the entire tree mirrored the right half.

To fix this, the recursion must take two pointers to simultaneously traverse the tree outward and inward, validating that the outer branches match each other, and the inner branches match each other.

## Complexity

|**Complexity**|**Why**|
|---|---|
|Time O(N)|In the worst case (a perfectly symmetric tree), every node in both the left and right subtrees must be visited and compared exactly once.|
|Space O(H)|The recursive call stack reaches a maximum depth equal to the height of the tree (worst-case O(N) for skewed trees, O(log N) for balanced trees).|

## Key Insight

Symmetry in a binary tree is a two-pointer problem, not a single-pointer problem. You cannot determine symmetry by looking at a single node's children. You must structurally compare the root's left subtree against the root's right subtree. This requires traversing them simultaneously: pairing a leftward movement on one side with a rightward movement on the other (comparing `left.left` to `right.right`, and `left.right` to `right.left`).

## Mistakes / Gaps

1. **Misunderstanding Symmetry** — I checked for local equality (do a node's two children match?) instead of mirrored structural equality (does the left branch mirror the right branch?). 
2. **Independent Recursion** — My recursive calls (`recurse(left)` and `recurse(right)`) operated in complete isolation. Fixed by rewriting the helper function to accept two nodes so they can be continuously cross-checked.

## Code

```python
# The corrected, optimal code based on the two-pointer approach:
class Solution:
    def isSymmetric(self, root: Optional[TreeNode]) -> bool:
        def recurse(l, r):
            if not l and not r:
                return True
            
            if not l or not r:
                return False

            if l.val != r.val:
                return False
            
            # Outer edges must match, and inner edges must match
            return recurse(l.left, r.right) and recurse(l.right, r.left)
        
        return recurse(root.left, root.right)
```

## Is My Solution Optimal?

To prove a tree is symmetric, every corresponding pair of nodes must be checked at least once. Therefore, $O(N)$ time is the absolute lower bound. A recursive approach will fundamentally require $O(H)$ space for the call stack. The corrected solution hits both optimal boundaries. **Yes, optimal.**

## Code Improvements

None — the corrected two-pointer code is already highly readable, properly handles all base cases (both null, one null, value mismatch), and uses standard Pythonic boolean checks. 

## Best Solution

```python
class Solution:
    def isSymmetric(self, root: Optional[TreeNode]) -> bool:
        def recurse(l, r):
            if not l and not r: return True
            if not l or not r or l.val != r.val: return False
            return recurse(l.left, r.right) and recurse(l.right, r.left)
        
        return recurse(root.left, root.right) if root else True
```

This is identical in complexity to the provided optimal code, but collapses the three failure conditions (left missing, right missing, or values mismatched) into a single line for brevity. An iterative approach using a queue (BFS) is also completely valid and avoids recursion limits, but DFS is generally shorter and easier to write under pressure.