---
question: "[[08.BinaryTreeLevelOrderTraversal|BinaryTreeLevelOrderTraversal]]"
topic: ["Tree"]
lc_difficulty: Medium
tags: ["neetcode-150"]
attempt_date: 2026-06-23
my_difficulty: Easy
status: Done
time_min: 5
review_concepts: ["BFS"]
---
---
question: "[[08.BinaryTreeLevelOrderTraversal|BinaryTreeLevelOrderTraversal]]"
topic: ["Tree"]
lc_difficulty: Medium
tags: ["neetcode-150"]
attempt_date: 2026-06-23
my_difficulty: Easy
status: Done
time_min: 5
review_concepts: ["BFS"]
---
# Binary Tree Level Order Traversal

_Traverse the tree layer by layer using a queue, capturing the queue size at each step to group nodes._

## My Approach

I used a Breadth-First Search (BFS) with a queue to traverse the tree. Since the problem asks for nodes grouped strictly by their depth, BFS is the natural fit because it explores all nodes at the current depth before moving deeper.

The core logic involves a `while` loop that continues as long as the queue has nodes. Inside, I snapshot the length of the queue (`level = len(nodes)`). This length tells me exactly how many nodes exist on the current level. I then loop exactly that many times, popping the nodes, recording their values, and pushing their non-null children into the queue for the next level.

## Complexity

|**Complexity**|**Why**|
|---|---|
|Time O(N)|Every node in the tree is enqueued and dequeued exactly once.|
|Space O(N)|The queue holds at most one full level of the tree. In the worst case (a perfect binary tree), the bottom level holds roughly N/2 nodes.|

## Key Insight

A standard BFS explores a tree level by level but mixes nodes from different levels in the queue simultaneously as it transitions. The conceptual unlock here is taking a snapshot of the queue's size *before* processing any nodes. By looping exactly `length` times, you perfectly isolate the current level and ensure all newly discovered children are reserved for the next iteration of the outer loop.

## Mistakes / Gaps

None this attempt.

## Code

```python
class Solution:
    def levelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        nodes = deque([])
        if root:
            nodes.append(root)
        
        result = []

        while nodes:
            level = len(nodes)
            level_vals = []
            
            for _ in range(level):
                node = nodes.popleft()
                level_vals.append(node.val)
                
                if node.left:
                    nodes.append(node.left)
                if node.right:
                    nodes.append(node.right)

            result.append(level_vals)

        return result
```

## Is My Solution Optimal?

To return the value of every node, we must visit every node, making O(N) time the theoretical floor. Space must also be O(N) to store the output structure and the BFS queue. My solution matches both bounds.
**Yes, optimal.**

## Code Improvements

- **Early exit** — If `not root`, returning `[]` immediately at the top of the function avoids unnecessary allocations and setups entirely.
- **Queue initialization** — `nodes = deque([root])` is cleaner than initializing an empty deque and appending afterward.
- **Inlined length evaluation** — `for _ in range(len(nodes)):` evaluates the length exactly once at the start of the loop, removing the need for a standalone `level` variable.

## Best Solution

```python
class Solution:
    def levelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        if not root:
            return []
            
        res = []
        queue = deque([root])
        
        while queue:
            level_vals = []
            for _ in range(len(queue)):
                node = queue.popleft()
                level_vals.append(node.val)
                if node.left:
                    queue.append(node.left)
                if node.right:
                    queue.append(node.right)
            res.append(level_vals)
            
        return res
```

This is the canonical BFS level-order approach. It maintains the same optimal O(N) time and O(N) space complexities but cleans up the initialization and removes redundant variables, making it highly readable and strictly idiomatic.