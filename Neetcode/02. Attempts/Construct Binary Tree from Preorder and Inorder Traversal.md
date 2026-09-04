---
question: "[[13.ConstructBinaryTreeFromPreorderAndInorderTraversal]]"
topic:
  - Tree
lc_difficulty: Medium
tags:
  - neetcode-150
attempt_date: 2026-06-27
my_difficulty: Trick
status: Gave Up
time_min: 60
review_concepts:
  - Hash Map
  - DFS
---
# Construct Binary Tree from Preorder and Inorder Traversal

_Use the preorder array to fetch roots sequentially, and an inorder hash map to cleanly partition the left and right subtrees._

## My Approach

I knew this required a recursive Depth-First Search (DFS) approach to rebuild the tree, leveraging the mathematical properties of the two traversals. The `preorder` array gives us the root of the current subtree immediately, while the `inorder` array tells us exactly which elements belong in the left subtree versus the right subtree. 

Initially, I struggled with pointer management, trying to track left and right boundary indices for both arrays simultaneously. I eventually landed on a cleaner logic: using a global/nonlocal pointer (`pre_i`) to sequentially grab the next root from `preorder`, while passing `left` and `right` boundaries to slice the `inorder` array. 

To optimize the search for the root's index within the `inorder` array, I precomputed a hash map storing the value-to-index mappings, reducing what would be an $O(N)$ linear scan into an $O(1)$ lookup per node.

## Complexity

|**Complexity**|**Why**|
|---|---|
|Time O(N)|Every node is created exactly once, and finding its split point in the inorder array takes O(1) time using the hash map.|
|Space O(N)|The hash map stores N elements, and the recursive call stack takes O(H) space, which scales to O(N) in the worst case.|

## Key Insight

The first element of a preorder traversal is *always* the root of the current tree (or subtree). Once you identify that root, finding its position in the inorder traversal strictly divides the remaining nodes into the left subtree (everything to the left of the root) and the right subtree (everything to the right). Because we build the tree in a pre-order fashion (`Root -> Left -> Right`), simply incrementing a pointer across the `preorder` array will perfectly hand you the correct root for every recursive call.

## Mistakes / Gaps

1. **Pointer mismatch** — Used `left` and `right` inorder boundaries to index into the `preorder` array (`val = preorder[left]`). This pulled the wrong root values, scrambled the boundaries, and caused a Time Limit Exceeded (TLE) error via infinite recursion. ([[Common Mistakes#Cross-Array Index Confusion|↗]])
2. **Pointer soup** — Got confused trying to track four separate indices (left/right for both arrays) instead of realizing the preorder array only needs a single, continuously incrementing pointer. ([[Common Mistakes#Pointer Fatigue|↗]])
3. **Missing the hash map (initially)** — Didn't immediately consider the hash map trick, which is essential to bring the time complexity down from $O(N^2)$ to $O(N)$. ([[Space-Time Tradeoff|↗]])
4. **Premature optimization** — Tried to optimize the logic before establishing a basic, functional recursive model, leading to overthinking and wasted time. ([[Common Mistakes#Premature Optimization|↗]])

## Code

```python
class Solution:
    def buildTree(self, preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
        ini = dict()
        for i, val in enumerate(inorder):
            ini[val] = i
            
        pre_i = 0

        def recurse(left, right):
            if left > right:
                return None

            nonlocal pre_i
            
            val = preorder[pre_i]
            pre_i += 1
            val_i = ini[val]

            node = TreeNode(val)

            node.left = recurse(left, val_i - 1)
            node.right = recurse(val_i + 1, right)
            return node
        
        return recurse(0, len(inorder) - 1)
```

## Is My Solution Optimal?

We must process every single node from the input arrays to construct the tree, establishing a strict O(N) time floor. We also must allocate O(N) space to store the actual tree nodes in memory. My solution matches both theoretical bounds.
**Yes, optimal.**

## Code Improvements

- **Use an iterator** — Instead of manually managing a `pre_i` index and declaring it `nonlocal`, converting `preorder` into an iterator (`pre_iter = iter(preorder)`) and calling `next(pre_iter)` handles the state natively and idiomatically.
- **Dictionary comprehension** — The manual loop to populate the `ini` dictionary can be written as a clean one-liner: `ini = {val: i for i, val in enumerate(inorder)}`.
- **Descriptive naming** — `val_i` and `ini` are slightly cryptic; `mid` and `inorder_map` communicate their specific roles in the algorithm much more clearly.

## Best Solution

```python
class Solution:
    def buildTree(self, preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
        inorder_map = {val: i for i, val in enumerate(inorder)}
        preorder_iter = iter(preorder)
        
        def build(left, right):
            if left > right:
                return None
            
            # iter() safely advances the pointer natively, no nonlocal needed
            root_val = next(preorder_iter)
            root = TreeNode(root_val)
            
            mid = inorder_map[root_val]
            
            root.left = build(left, mid - 1)
            root.right = build(mid + 1, right)
            
            return root
            
        return build(0, len(inorder) - 1)
```

This represents the canonical best approach. It retains the exact optimal O(N) time and space profile of your solution but refines the Python mechanics. Replacing the manual `nonlocal` index with `iter()` removes unnecessary state management and prevents the kinds of boundary-mismatch bugs that caused the initial TLE.