---
question: "[[05.SameTree|SameTree]]"
topic: ["Tree"]
lc_difficulty: Easy
tags: ["neetcode-150"]
attempt_date: 2026-06-20
my_difficulty: Easy
status: Done
time_min: 5
review_concepts: ["DFS"]
---
# Same Tree

_Traverse two trees simultaneously using DFS, returning false the moment a structural or value mismatch is found._

## My Approach

I reached for a Depth-First Search (DFS) recursive approach to traverse both trees synchronously. Since checking if two trees are identical requires verifying both their structure and node values, walking through them at the exact same pace is the most direct and intuitive solution.

The core logic relies on a nested helper function `recurse(r1, r2)` that evaluates matching pairs of nodes. The base cases handle structural alignment: if both nodes are null, we've safely reached the end of identical paths. If only one node is null while the other exists, the tree structures have diverged, and I immediately return `False`.

If both nodes exist, I first verify that their numerical values match. Assuming the current node pair is identical, I recursively invoke the function on both their left children and right children. Using the boolean `and` operator ensures that the current branch only resolves to `True` if every deeper subtree perfectly mirrors its counterpart.

## Complexity

|**Complexity**|**Why**|
|---|---|
|Time O(min(N, M))|Where N and M are the number of nodes in `p` and `q`. The traversal stops as soon as a mismatch is found, evaluating at most the size of the smaller tree (or full size if identical).|
|Space O(min(H1, H2))|The recursive call stack depth scales with the height of the trees, bound by the shallower tree where a structural mismatch would trigger a return.|

## Key Insight

Tree equality is a composite of structural shape and local node values. By evaluating the current nodes before branching out (a pre-order traversal pattern), you can "fail fast"—halting the recursion and unwinding the stack as soon as any single discrepancy is detected. The recursive leap of faith cleanly delegates the rest: two trees are identical if the current root values match, AND their left subtrees are identical, AND their right subtrees are identical.

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
    def isSameTree(self, p: Optional[TreeNode], q: Optional[TreeNode]) -> bool:
        def recurse(r1, r2):
            if not r1 or not r2:
                if r1 or r2:
                    return False
                else:
                    return True
            
            if r1.val != r2.val:
                return False
            
            return recurse(r1.left, r2.left) and recurse(r1.right, r2.right)

        return recurse(p, q)
```
