---
question: "[[15.SerializeAndDeserializeBinaryTree|SerializeAndDeserializeBinaryTree]]"
topic: ["Tree"]
lc_difficulty: Hard
tags: ["neetcode-150"]
attempt_date: 2026-06-23
my_difficulty: Hard
status: Redo (Too Long)
time_min: 50
review_concepts: ["DFS", "String Manipulation"]
---
# Serialize and Deserialize Binary Tree

_Encode the tree structure and node values into a string using a pre-order DFS traversal, then reconstruct it by consuming that string sequentially._

## My Approach

I used a Depth-First Search (DFS) approach to serialize the tree into a custom format: `left_exists|right_exists|value`, delimited by spaces. This explicit marking of children prevents ambiguity when reconstructing the structure. 

For serialization, I traversed the tree and appended the custom formatted string to a list, joining it with spaces at the end. For deserialization, I used a `nonlocal` index variable to sequentially consume the array of split strings. By checking the left/right existence flags extracted from the string, I recursively built the children. 

While it felt slightly unnatural to rely entirely on a parameterless `recurse()` function and global state (`nonlocal`), it correctly maintained the index during the recursive descent. Notably, this explicit child-flagging structure requires only minor adjustments to handle an N-ary tree, making it a robust pattern for variations of the problem (like Citadel's N-ary follow-up).

## Complexity

|**Complexity**|**Why**|
|---|---|
|Time O(N)|Every node is visited exactly once during serialization, and exactly once during deserialization.|
|Space O(N)|The string construction, split array, and the recursive call stack all scale linearly with the number of nodes in the tree.|

## Key Insight

A binary tree can be uniquely serialized and deserialized if you encode both its *data* (node values) and its *structure* (null pointers/child existence) in a consistent traversal order. Pre-order traversal (Root, Left, Right) is ideal because the root is processed first. During deserialization, this allows you to immediately instantiate the parent node before recursively delegating the remaining string to build its children.

## Mistakes / Gaps

1. **String function familiarity** — Struggled with Python string methods and mixed up syntax, accidentally writing `"|".split(node_str)` instead of `node_str.split("|")`.
2. **Trailing delimiters** — Initially used a comma delimiter but left a trailing comma, which created an empty string artifact upon splitting and threw off the parsing logic. ([[Common Mistakes#String Splitting Pitfalls|↗]])
3. **Empty string edge case** — Splitting an empty string returns `[""]` rather than an empty list, causing an off-by-one error for the empty tree edge case. ([[Common Mistakes#String Splitting Pitfalls|↗]])
4. **Type conversion oversights** — Forgot that `split()` returns strings; mistakenly evaluated `if left` (which evaluates to True for `"0"`) instead of `if left == "1"`. ([[Common Mistakes#String Truthiness Trap|↗]])
5. **Over-reliance on debugger** — Spammed the "Run" button heavily to step through and debug the string parsing. This builds a bad habit for whiteboard/live interviews.
6. **Unorthodox state management** — Using a parameterless `recurse()` with a `nonlocal` index works, but it breaks standard templates and can be harder to reason about than passing an iterator natively.

## Code

```python
class Codec:

    def serialize(self, root):
        result = []

        def recurse(r):
            if not r:
                return

            nonlocal result

            node_str = ["0", "|", "0", "|", str(r.val), " "]
            
            if r.left:
                node_str[0] = "1"
            
            if r.right:
                node_str[2] = "1"
            
            result.append("".join(node_str))

            recurse(r.left)
            recurse(r.right)
        
        recurse(root)
        
        return "".join(result)
        

    def deserialize(self, data):
        if not data:
            return None

        index = 0
        nodes = data.strip().split(" ")

        def recurse():
            nonlocal index, nodes

            if index >= len(nodes):
                return None

            node_str = nodes[index]
            index += 1

            left, right, val = node_str.split("|")
            node = TreeNode(int(val))

            if left == "1":
                node.left = recurse()
            
            if right == "1":
                node.right = recurse()
            
            return node
        
        return recurse()
```

## Is My Solution Optimal?

Every node must be processed to serialize the tree, and the full string must be parsed to deserialize it, making O(N) time the theoretical floor. Space must also be O(N) to hold the output string and recursion stack. My solution matches both bounds.
**Yes, optimal.**

## Code Improvements

- **Use an iterator** — Instead of maintaining a `nonlocal index` and checking bounds, creating an [[Python Lists and Iteration#Iterators and Sequential Token Consumption|iterator]] (`vals = iter(data.split())`) and calling `next(vals)` is far more Pythonic, robust, and avoids explicit index management entirely.
- **String builder simplification** — Constructing `node_str` via a mutable list of characters (`["0", "|", ...]`) is overly complex. An f-string like `f"{1 if r.left else 0}|{1 if r.right else 0}|{r.val}"` handles this cleanly in one line.
- **Simplify structural encoding** — While the `left|right|val` encoding is clever, standard Pre-order serialization using just `val` and a distinct marker (like `N`) for empty nodes is significantly simpler to write and less prone to parsing bugs.

## Best Solution

```python
class Codec:
    def serialize(self, root):
        res = []
        
        def dfs(node):
            if not node:
                res.append("N")
                return
            res.append(str(node.val))
            dfs(node.left)
            dfs(node.right)
            
        dfs(root)
        return ",".join(res)

    def deserialize(self, data):
        vals = iter(data.split(","))
        
        def dfs():
            val = next(vals)
            if val == "N":
                return None
            
            node = TreeNode(int(val))
            node.left = dfs()
            node.right = dfs()
            
            return node
            
        return dfs()
```

This is the canonical optimal approach. It maintains the same O(N) time and O(N) space complexities but drastically simplifies the string parsing. By replacing missing children with a literal `"N"`, the structure is preserved implicitly without needing complex delimiters or bit-flags. The use of Python's built-in `iter()` handles state consumption cleanly without `nonlocal` indices.
