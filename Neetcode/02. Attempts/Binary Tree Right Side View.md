---
question: "[[09.BinaryTreeRightSideView|BinaryTreeRightSideView]]"
topic: ["Tree"]
lc_difficulty: Medium
tags: ["neetcode-150"]
attempt_date: 2026-06-23
my_difficulty: Easy
status: Done
time_min: 5
review_concepts: ["BFS", "DFS"]
---
# Binary Tree Right Side View

_Traverse the tree by depth, capturing the rightmost node of each level._

## My Approach

I reached for a Breadth-First Search (BFS) using a queue to process the tree level by level. Because the problem asks for the elements visible from the right side, we just need the final node at every depth.

The core logic relies on isolating each level in the queue. At the beginning of the `while` loop, the queue contains exactly all nodes for the current level. By appending the value of the last node in the queue (`nodes[-1].val`) to my result array, I trivially capture the rightmost visible node. I then loop `level` times to pop those processed nodes and enqueue their non-null children from left to right for the next iteration.

## Complexity

|**Complexity**|**Why**|
|---|---|
|Time O(N)|Every node in the tree is enqueued and dequeued exactly once.|
|Space O(N)|The queue holds at most one full level of the tree, which bounds to roughly N/2 for a perfect binary tree.|

## Key Insight

The right side view of a tree isn't just a single continuous path down the rightmost edges; a left child can be visible from the right if the right child is `null`. The right side view is simply the last node encountered at *every* depth. Grouping nodes strictly by depth makes finding the "rightmost" node trivial—it is always the last element enqueued for that level. 

## Mistakes / Gaps

None this attempt.

## Code

```python
class Solution:
    def rightSideView(self, root: Optional[TreeNode]) -> List[int]:
        result = []
        nodes = deque([])
        if root:
            nodes.append(root)

        while nodes:
            level = len(nodes)
            result.append(nodes[-1].val)

            for _ in range(level):
                node = nodes.popleft()

                if node.left:
                    nodes.append(node.left)

                if node.right:
                    nodes.append(node.right)

        return result
```

## Is My Solution Optimal?

Every node might need to be visited to ensure no deeper nodes are hidden, meaning O(N) time is the theoretical floor. The space complexity for BFS requires O(N) to hold the queue. My solution matches these bounds.
**Yes, optimal.**

## Code Improvements

- **Early exit** — Adding `if not root: return []` at the beginning prevents unnecessary initializations and simplifies the main logic.
- **Queue initialization** — `nodes = deque([root])` is cleaner and more idiomatic than initializing an empty deque and appending afterward.
- **Inlined length evaluation** — `for _ in range(len(nodes)):` evaluates the length exactly once at the start of the loop, removing the need for a standalone `level` variable.

## Best Solution

```python
class Solution:
    def rightSideView(self, root: Optional[TreeNode]) -> List[int]:
        res = []
        
        def dfs(node, depth):
            if not node:
                return
            
            # If this is the first time we've reached this depth, add the node
            if depth == len(res):
                res.append(node.val)
                
            # Traverse right side first
            dfs(node.right, depth + 1)
            dfs(node.left, depth + 1)
            
        dfs(root, 0)
        return res
```

While the BFS approach is optimal for time, a right-to-left Depth-First Search (DFS) is structurally cleaner and offers a tighter space bound on balanced trees. By prioritizing the right child in the recursion, the first node encountered at any new depth is guaranteed to be the rightmost visible node. This improves the space complexity on balanced trees from O(N) queue space to O(H) call stack space.