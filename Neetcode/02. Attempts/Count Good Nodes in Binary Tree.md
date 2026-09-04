---
question: "[[10.CountGoodNodesInBinaryTree|CountGoodNodesInBinaryTree]]"
topic: ["Tree"]
lc_difficulty: Medium
tags: ["neetcode-150"]
attempt_date: 2026-06-23
my_difficulty: Easy
status: Done
time_min: 7
review_concepts: ["DFS"]
---
# Count Good Nodes in Binary Tree

_Traverse the tree top-down, passing the maximum value seen so far along the path to evaluate each node._

## My Approach

I reached for a Depth-First Search (DFS) recursive traversal. Since a node is considered "good" based entirely on the values of its ancestors, DFS allows us to track the path's maximum value natively by passing it down as a parameter.

The core logic uses a nested helper function `recurse(r, m)` where `m` represents the maximum value encountered on the path from the root down to node `r`. At each step, I check if the current node's value is greater than or equal to `m`. If it is, I count it as a "good" node and update `m` to the new node's value. 

I then recursively traverse the left and right children, passing the newly updated maximum, and sum the total "good" nodes found in both subtrees.

## Complexity

|**Complexity**|**Why**|
|---|---|
|Time O(N)|Every single node in the tree is visited exactly once to evaluate its "goodness".|
|Space O(H)|The call stack depth scales with the height of the tree (worst-case O(N) for skewed trees, O(log N) for balanced trees).|

## Key Insight

A node's "goodness" depends exclusively on the maximum value above it. By carrying a running maximum down the recursion tree as state, you avoid needing to look backwards or maintain a full path history. This transforms a potentially O(N²) ancestor-lookup approach into a clean, single-pass O(N) top-down traversal. 

## Mistakes / Gaps

None this attempt.

## Code

```python
class Solution:
    def goodNodes(self, root: TreeNode) -> int:
        def recurse(r, m):
            if not r:
                return 0
            
            good = 0
            if r.val >= m:
                good += 1
                m = r.val
            
            return good + recurse(r.left, m) + recurse(r.right, m)

        
        return recurse(root, root.val)
```

## Is My Solution Optimal?

To determine how many good nodes exist, we must evaluate every single node in the tree at least once. Therefore, O(N) time is the theoretical floor. Space complexity for a recursive traversal inherently requires O(H) to hold the recursion stack. My solution matches both bounds perfectly. 
**Yes, optimal.**

## Code Improvements

- **Inline ternary** — `good = 0` followed by an `if` block can be condensed into `good = 1 if r.val >= m else 0`.
- **Built-in max function** — Instead of a conditional reassignment for `m`, `m = max(m, r.val)` is more idiomatic and immediately communicates the intent.
- **Naming clarity** — Naming the helper `dfs(node, max_val)` is more standard and descriptive than `recurse(r, m)`.

## Best Solution

```python
class Solution:
    def goodNodes(self, root: TreeNode) -> int:
        def dfs(node, max_val):
            if not node:
                return 0
            
            # A node is good if it's >= the max value seen on its path
            good = 1 if node.val >= max_val else 0
            max_val = max(max_val, node.val)
            
            return good + dfs(node.left, max_val) + dfs(node.right, max_val)

        return dfs(root, root.val)
```

This uses the exact same optimal O(N) time and O(H) space logic as your solution. It is considered the canonical "best" approach because it streamlines the conditional logic using Python's inline ternary operator and the built-in `max()` function, making the state updates concise and highly readable.