---
question: "[[12.KthSmallestElementInABST|KthSmallestElementInABST]]"
topic: ["Tree"]
lc_difficulty: Medium
tags: ["neetcode-150"]
attempt_date: 2026-06-24
my_difficulty: Medium
status: Done
time_min: 10
review_concepts: ["DFS"]
---
# Kth Smallest Element in a BST

_An in-order traversal of a Binary Search Tree naturally processes nodes in perfectly sorted ascending order._

## My Approach

I reached for a Depth-First Search (DFS) in-order traversal. Because a BST guarantees that the left subtree is strictly smaller than the root, and the right subtree is strictly larger, visiting left-root-right naturally visits the elements from smallest to largest.

The core logic uses a recursive helper function to perform this traversal while keeping track of a global count. I defined a `nonlocal` variable `index` to persist state across the recursive calls. 

I traversed down the left branch, incremented the index upon visiting the current node, and then checked if the index matched `k`. If it did, I returned the node's value. If the counter exceeded `k`, I bubbled up the result from the left subtree. Otherwise, I continued the traversal into the right subtree.

## Complexity

|**Complexity**|**Why**|
|---|---|
|Time O(N)|In the worst-case scenario (a right-skewed tree where `k` is the deepest node), we might visit every node, making it scale linearly.|
|Space O(H)|The recursive call stack takes space proportional to the height of the tree, which is O(N) for skewed trees and O(log N) for balanced ones.|

## Key Insight

You do not need to convert the entire tree into an array and then extract the `k`-th element. Since the inherent definition of a BST guarantees sorted order via an in-order traversal, you can just traverse the tree and maintain a simple counter. The moment that counter hits `k`, you have found your answer and can short-circuit the rest of the traversal.

## Mistakes / Gaps

1. **State management confusion** — Struggled with deciding whether the counter should be passed as a parameter, returned as an output of the recursive function, or managed globally, eventually settling on a `nonlocal` variable.
2. **Index synchronization** — Struggled with the incrementing logic, testing combinations like initializing at 1 and incrementing before the check, or initializing at 0 and checking before the increment; failing to keep the state checks tightly synced with the traversal step.

## Code

```python
class Solution:
    def kthSmallest(self, root: Optional[TreeNode], k: int) -> int:
        index = 0

        # return 
        def recurse(node):
            if not node:
                return
            
            nonlocal index
            
            left = recurse(node.left)
            index += 1

            if index > k:
                return left
            
            if index == k:
                return node.val
            
            return recurse(node.right)

        return recurse(root)
```

## Is My Solution Optimal?

The theoretical lower bound requires traversing down to the smallest element and then visiting `k` nodes, which yields a time complexity of O(H + k). In the worst case (skewed tree, `k` equals `N`), this degrades to O(N) time and O(H) space. My solution operates within these bounds and avoids visiting the right side of the tree once the answer is found. **Yes, optimal.**

## Code Improvements

- **Awkward value bubbling** — Checking `if index > k: return left` is a non-standard and slightly confusing way to bubble up the found integer. It's much cleaner to check if the left call yielded a non-None result (e.g., `if left is not None: return left`) immediately after the left recursive call, preventing unnecessary increments.
- **Inconsistent return types** — The function implicitly returns `None` for empty branches but explicitly returns integers when the target is found, which can cause type-hinting issues and makes the bubbling logic harder to reason about.

## Best Solution

```python
class Solution:
    def kthSmallest(self, root: Optional[TreeNode], k: int) -> int:
        stack = []
        curr = root
        
        while stack or curr:
            while curr:
                stack.append(curr)
                curr = curr.left
                
            curr = stack.pop()
            k -= 1
            
            if k == 0:
                return curr.val
                
            curr = curr.right
```

This iterative in-order traversal uses an explicit stack, matching the optimal O(H + k) time and O(H) space complexity. It is the "best" solution because it completely avoids the confusion of `nonlocal` variables and the tricky logic of bubbling a return value up a recursive call stack. Once `k` hits zero, it simply returns the value and terminates execution instantly.