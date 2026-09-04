---
question: "[[02.DesignAddandSearchWordDataStructure]]"
topic:
  - Trie
lc_difficulty: Medium
tags:
  - neetcode-150
attempt_date: 2026-06-30
my_difficulty: Medium
status: Done
time_min: 15
review_concepts:
  - Trie
  - DFS
---
# Design Add and Search Words Data Structure

_Use a Trie for efficient prefix storage, and recursively apply DFS backtracking to handle wildcard character searches._

## My Approach

I implemented a standard Trie structure to store the words. The `addWord` method uses a simple iterative traversal to insert characters, creating new nodes in the `children` dictionary as needed and flagging the final node with `is_word`.

The complexity lies in the `search` method due to the `.` wildcard requirement. Because a wildcard can represent any character, an iterative approach is insufficient. I reached for a recursive helper function to perform Depth-First Search (DFS). When the function encounters a standard character, it traverses down normally. When it encounters a `.`, it iterates over all available child nodes in the `children` dictionary and branches out, recursively calling the search function on every possible path. I used `result = result or recurse(...)` to short-circuit the search the moment a valid path returns `True`.

## Complexity

|**Complexity**|**Why**|
|---|---|
|Time O(M) / O(26^M)|Adding a word takes $O(M)$ time where $M$ is the word length. Searching without wildcards is $O(M)$. Searching with wildcards triggers DFS branching, taking up to $O(26^M)$ time in the absolute worst case (e.g., `.....` in a dense Trie).|
|Space O(M)|The Trie itself stores characters, but auxiliary space per `search` call scales to $O(M)$ due to the recursive call stack depth reaching the length of the searched word.|

## Key Insight

A standard Trie lookup is a single-path traversal. Introducing a wildcard transforms the lookup into a backtracking problem. When you don't know the exact path to take (because of a `.`), you must branch out and try *every* possible path currently available in the tree. DFS handles this elegantly by exploring one full branch at a time and gracefully backing out if it hits a dead end. 

## Mistakes / Gaps

1. [[Common Mistakes#Data Structure Internal Type Confusion|Data Structure Internal Type Confusion]] — Briefly forgot `children` is a dict, not a list, causing friction when determining how to iterate branches during the wildcard search step.
2. [[Common Mistakes#Debugger Reliance|Debugger Reliance]] — Leaned too heavily on the debugger to trace execution flow instead of mentally walking through the recursive call stack.
3. **Forgot `is_word`** — Forgot to set `is_word = True` on the final node after the insertion loop.

## Code

```python
class Node:
    def __init__(self):
        self.children = dict()
        self.is_word = False

class WordDictionary:

    def __init__(self):
        self.root = Node()

    def addWord(self, word: str) -> None:
        node = self.root

        for c in word:
            if c not in node.children:
                node.children[c] = Node()
        
            node = node.children[c]
        
        node.is_word = True

    def search(self, word: str) -> bool:
        def recurse(node, start):
            for i in range(start,len(word)):
                c = word[i]
                if c == '.':
                    result = False
                    for _, child in node.children.items():
                        result = result or recurse(child, i + 1)
                    return result

                if c not in node.children:
                    return False
            
                node = node.children[c]
            
            return node.is_word
        
        return recurse(self.root, 0)
```

## Is My Solution Optimal?

The theoretical lower bound for adding a word to a Trie is $O(M)$. For searching with wildcards, testing every existing branch is fundamentally required, meaning $O(N \cdot 26^M)$ worst-case time is unavoidable. Space is limited to the call stack size of $O(M)$. **Yes, optimal.**

## Code Improvements

- **Iterating dictionary values** — `for _, child in node.children.items():` extracts both keys and values but ignores the key. Use `for child in node.children.values():` instead.
- **Simplifying boolean logic** — Accumulating `result = result or recurse(...)` works, but explicitly returning `True` inside the loop upon finding a match is more readable and explicitly short-circuits.
- **Top-level Node class** — The `Node` class can just be renamed to `TrieNode` to avoid namespace collisions in larger systems.

## Best Solution

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_word = False

class WordDictionary:
    def __init__(self):
        self.root = TrieNode()

    def addWord(self, word: str) -> None:
        cur = self.root
        for c in word:
            if c not in cur.children:
                cur.children[c] = TrieNode()
            cur = cur.children[c]
        cur.is_word = True

    def search(self, word: str) -> bool:
        def dfs(j, root):
            cur = root
            for i in range(j, len(word)):
                c = word[i]
                if c == ".":
                    for child in cur.children.values():
                        if dfs(i + 1, child):
                            return True
                    return False
                else:
                    if c not in cur.children:
                        return False
                    cur = cur.children[c]
            return cur.is_word
        
        return dfs(0, self.root)
```

This represents the cleanest refinement of the logic. It maintains the $O(M)$ time for additions and recursive DFS for wildcards, but improves the inner loop's readability by directly returning `True` on a wildcard match and safely returning `False` if all branches fail. Calling `.values()` cleans up the dictionary iteration.