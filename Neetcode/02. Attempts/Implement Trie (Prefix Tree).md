---
question: "[[01.ImplementTrie]]"
topic:
  - Trie
lc_difficulty: Medium
tags:
  - neetcode-150
attempt_date: 2026-06-27
my_difficulty: Medium
status: Failed
time_min: 20
review_concepts:
  - Trie
---
# Implement Trie (Prefix Tree)

_A tree where each node represents a character, used for efficient word and prefix lookups._

## My Approach

I implemented a nested `Node` class containing the character, a dictionary for children, and a boolean flag `is_end` to mark the completion of a valid word. The main `Trie` class initializes with a dummy root node.

For insertion, I iterated through the word character by character. If a character was missing from the current node's children, I instantiated a new `Node` and added it to the dictionary. I then traversed downwards, marking the final node's `is_end` flag as `True`.

For search and prefix matching, I iterated through the target string, traversing the children dictionaries. If a required character was missing at any point, I returned `False`. `search` strictly validates the final node's `is_end` flag, while `startsWith` simply returns `True` as long as the traversal successfully completes.

## Complexity

|**Complexity**|**Why**|
|---|---|
|Time O(M)|Every operation (insert, search, startsWith) traverses the length of the target string $M$, performing $O(1)$ dictionary lookups per character.|
|Space O(M)|In the worst case, insertion creates a new node for every character in the string. Search and startsWith use $O(1)$ auxiliary space.|

## Key Insight

A Trie represents shared prefixes by sharing branches in memory. Instead of storing entire words independently, words are defined by the path taken from the root to a node marked `is_end`. The children hash map at each node dictates the valid subsequent characters, allowing for $O(M)$ lookups regardless of how many total words exist in the dictionary.

## Mistakes / Gaps

1. **Mutable default argument** — Initialized the `Node` using `children = dict()` in the method signature. Default arguments are evaluated at definition time, causing all nodes across all test cases to share the exact same dictionary in memory. Fixed by defaulting to `None` and initializing `{}` inside the constructor. ([[Common Mistakes#Mutable Default Argument|↗]])
2. **Scope error** — Tried to instantiate the nested class using `Node(char)` from inside itself instead of `Trie.Node(char)`. ([[Common Mistakes#Nested Class Scope|↗]])
3. **Assignment typo** — Assigned the class definition (`Node`) to the children dictionary instead of the newly created instance (`child`). ([[Common Mistakes#Object vs Type Assignment|↗]])
4. **Variable naming** — Looped over `word` instead of the passed parameter `prefix` inside the `startsWith` method. ([[Common Mistakes#Copy-Paste Drift|↗]])

## Code

```python
class Trie:
    class Node:
        def __init__(self, char = None, children = None, is_end = False):
            self.char = char
            self.children = children if children else {}
            self.is_end = is_end

        def insert(self, char):
            if char in self.children:
                return
            
            child = Trie.Node(char)
            self.children[char] = child


    def __init__(self):
        self.root = self.Node()

    def insert(self, word: str) -> None:
        cur = self.root
        for c in word:
            cur.insert(c)
            cur = cur.children[c]

        cur.is_end = True

    def search(self, word: str) -> bool:
        cur = self.root
        for c in word:
            if c not in cur.children:
                return False
            
            cur = cur.children[c]
        
        return cur.is_end

        
    def startsWith(self, prefix: str) -> bool:
        cur = self.root
        for c in prefix:
            if c not in cur.children:
                return False
            
            cur = cur.children[c]
        
        return True
```

## Is My Solution Optimal?

Traversing a word character by character takes bounded O(M) time, which is the theoretical lower bound since we must look at every character of the string to confirm its existence. Space complexity is also optimal as we only allocate memory for unique prefixes. **Yes, optimal.**

## Code Improvements

- **Unnecessary nesting and indirection** — The `Node.insert` method adds unnecessary abstraction. The main `Trie` loop should handle dictionary assignments directly.
- **Redundant state** — The `char` attribute inside the `Node` is not needed. The character is entirely represented by the key pointing to the node in the parent's `children` dictionary.
- **Global class extraction** — Extracting the `Node` class outside of `Trie` completely prevents the scope `NameError` confusion and keeps the code cleaner.

## Best Solution

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word: str) -> None:
        cur = self.root
        for c in word:
            if c not in cur.children:
                cur.children[c] = TrieNode()
            cur = cur.children[c]
        cur.is_end = True

    def search(self, word: str) -> bool:
        cur = self.root
        for c in word:
            if c not in cur.children:
                return False
            cur = cur.children[c]
        return cur.is_end

    def startsWith(self, prefix: str) -> bool:
        cur = self.root
        for c in prefix:
            if c not in cur.children:
                return False
            cur = cur.children[c]
        return True
```

This represents the canonical implementation. Extracting `TrieNode` to the top level bypasses Python's tricky nested-class scope rules. Dropping the redundant `char` attribute and the nested `insert` helper streamlines the logic, leaving only the essential map traversals.