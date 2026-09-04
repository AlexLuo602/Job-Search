---
question: "[[03.WordSearchII|WordSearchII]]"
topic: ["Trie", "Backtracking"]
lc_difficulty: Hard
tags: ["neetcode-150"]
attempt_date: 2026-07-02
my_difficulty: Hard
status: Unoptimal
time_min: 43
review_concepts: ["Trie", "DFS", "Backtracking"]
---

# Word Search II

*Combine a Trie for prefix matching with DFS backtracking to efficiently find multiple words in a grid without redundant traversal.*

## My Approach

I built a Trie containing all the target words to allow O(1) prefix lookups and early path termination. The Trie stores characters in a `.children` dictionary and marks the end of a valid word with a `.word` string pointer.

I then iterated through every cell in the M x N grid. If the cell's character existed in the Trie's root, I initiated a Depth-First Search (DFS) backtracking algorithm.

During the DFS, I kept track of visited nodes using a set to avoid cyclical paths. I traversed down the Trie alongside the grid exploration; if a `node.word` was found, I appended it to the results and set it to `None` to prevent duplicate entries. The backtracking component reverted the visited state after exploring all valid adjacent neighbors.

## Complexity

|**Complexity**|**Why**|
|---|---|
|Time O(M * N * 3^L)|We iterate through all M*N cells, and in the worst case, explore 3 directions (excluding the origin cell) up to length L (maximum word length).|
|Space O(W + L)|W is the total number of characters in all words (Trie storage), and L is the maximum depth of the recursion stack.|

## Key Insight

Searching for multiple words sequentially using standard DFS results in massive redundant traversal. A Trie inverts this bottleneck: instead of checking if a grid path matches a specific word, the Trie lets you check if a path matches *any* valid dictionary prefix. By advancing a Trie node pointer simultaneously with the grid DFS, you explore overlapping prefixes concurrently and halt immediately when a sequence exits the dictionary.

## Mistakes / Gaps

1. **TrieNode Access** — Attempted to use dictionary bracket notation directly on the `TrieNode` instance rather than its `.children` dictionary.
2. **Set Method** — Incorrectly used `.append()` instead of `.add()` to add elements to the `visited` set.
3. **Starting Character Logic** — Triggered the DFS without verifying the starting cell against the Trie root, skipping the consumption of the initial character.
4. **Trie Pruning (Optimization Gap)** — Failed to dynamically remove exhausted branches from the Trie (when `node.word` is `None` and `.children` is empty), forcing the DFS to continually re-explore dead paths.

## Code

```python
from typing import List

class TrieNode:
    def __init__(self):
        self.children = dict()
        self.word = None

class Solution:
    def findWords(self, board: List[List[str]], words: List[str]) -> List[str]:
        result = []
        root = TrieNode()

        n = len(board)
        m = len(board[0])

        for word in words:
            cur = root
            for c in word:
                if c not in cur.children:
                    cur.children[c] = TrieNode()
                cur = cur.children[c]
            cur.word = word
        
        visited = set()
        
        def dfs(i, j, node):
            visited.add((i, j))

            if node.word:
                result.append(node.word)
                node.word = None
            
            neighbors = [(i-1, j), (i+1, j), (i, j-1), (i, j+1)]

            for x, y in neighbors:
                if x < 0 or x >= n or y < 0 or y >= m:
                    continue
                if (x, y) in visited:
                    continue

                c = board[x][y]
                if c not in node.children:
                    continue
                
                dfs(x, y, node.children[c])

            visited.discard((i, j))

        for i in range(n):
            for j in range(m):
                if board[i][j] in root.children:
                    dfs(i, j, root.children[board[i][j]])
        
        return result
```

## Is My Solution Optimal?

The worst-case theoretical lower bound requires exploring valid paths up to the maximum word length from every cell, so O(M * N * 3^L) time is the floor. My solution matches this conceptually, but practically fails to prune exhausted branches, leading to drastically slower runtimes (bottom 30%) on dense inputs with lots of overlap.
**No — O(M * N * 3^L) time / O(W + L) space is achievable with pruning.**
Dynamically removing Trie nodes once they have no words left prevents the DFS from re-exploring dead branches.

## Code Improvements

- **In-place visited tracking** — Instead of maintaining a separate `visited` set, temporarily modify the cell `board[i][j] = '#'` and revert it after the DFS to save space and hashing overhead.
- **Parent node parameter** — Pass the parent node and the current character into the DFS to enable dynamic deletion (`del parent.children[char]`).

## Best Solution

```python
from typing import List

class TrieNode:
    def __init__(self):
        self.children = {}
        self.word = None

class Solution:
    def findWords(self, board: List[List[str]], words: List[str]) -> List[str]:
        root = TrieNode()
        for word in words:
            cur = root
            for c in word:
                if c not in cur.children:
                    cur.children[c] = TrieNode()
                cur = cur.children[c]
            cur.word = word
            
        ROWS, COLS = len(board), len(board[0])
        res = []
        
        def dfs(r, c, parent, char):
            node = parent.children[char]
            
            if node.word:
                res.append(node.word)
                node.word = None
                
            # In-place visited tracking
            temp = board[r][c]
            board[r][c] = "#"
            
            directions = [(0, 1), (0, -1), (1, 0), (-1, 0)]
            for dr, dc in directions:
                nr, nc = r + dr, c + dc
                if 0 <= nr < ROWS and 0 <= nc < COLS and board[nr][nc] in node.children:
                    dfs(nr, nc, node, board[nr][nc])
                    
            board[r][c] = temp
            
            # Trie Pruning
            if not node.children and not node.word:
                del parent.children[char]

        for r in range(ROWS):
            for c in range(COLS):
                char = board[r][c]
                if char in root.children:
                    dfs(r, c, root, char)
                    
        return res
```

This solution introduces Trie pruning and in-place visited tracking. By passing the parent node into the DFS, it dynamically deletes branches that no longer contain valid words (`if not node.children and not node.word:`), drastically reducing the search space for subsequent DFS calls. Modifying the board directly removes the need for a `visited` set, improving spatial efficiency and execution speed.