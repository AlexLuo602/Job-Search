# Common Mistakes

## Pointer Fatigue
Forgetting to advance a pointer inside a `while` loop → infinite loop, overwriting the same node.
- After every loop body, ask: *did every pointer that drives the loop condition actually move?*

## Off-By-One (Range Bounds)
Using `range(0, n + 1, step)` instead of `range(0, n, step)` → `IndexError` on the final element.
- Trace with the smallest non-trivial input before assuming bounds are correct.

## Floor vs Ceiling Division
Using `n //= 2` when you need ceiling → odd element gets silently dropped.
- Ceiling: `(n + 1) // 2`

## Empty Input (Void Case)
Forgetting to guard against `len(input) == 0` → crash on `input[0]`.
- Apply the **0, 1, 2 Rule** before writing any loop logic (see below).

## Tree Min/Max: Forgetting the Root-Only and Split Cases
When computing a min/max value that can pass *through* the current node (e.g. maximum path sum, diameter), you must enumerate every candidate at each node — not just the best child subtree. The full candidate set at node `r` is:

| Candidate | When it applies |
|---|---|
| `left + r.val + right` | path runs through `r`, using both subtrees |
| `r.val + right` | left subtree is negative / not taken |
| `r.val + left` | right subtree is negative / not taken |
| `r.val` | both subtrees are negative / not taken |

A common mistake is computing `max(left, right) + r.val` — this forces exactly one child, silently discarding the split-path case (`left + r + right`) and the skip-child cases.

Pattern: compute `left = max(0, recurse(r.left))` and `right = max(0, recurse(r.right))` to clip negatives to zero, then the four candidates collapse to a single expression: update global max with `left + r.val + right`, and return `max(left, right) + r.val` to the parent (since a path returned upward can only continue in one direction).

**Clipping-to-zero caveat — all-negative trees**: clipping subtree contributions to 0 only works because `r.val` itself is always added unconditionally. The global max is initialised to `float('-inf')`, so even if every node is negative, the least-negative single node is still captured. The mistake to avoid is initialising the result to `0` instead of `float('-inf')` — that silently returns 0 on an all-negative tree, which is wrong when the problem requires at least one node in the path.

→ [[14.BinaryTreeMaximumPathSum|BinaryTreeMaximumPathSum]], [[03.DiameterOfABinaryTree|DiameterOfABinaryTree]]

## String Splitting Pitfalls
Two related bugs when building and parsing delimited strings:
- **Trailing delimiter artifact**: manually appending a delimiter (`result += val + ","`) leaves a trailing entry. On split, this creates a spurious empty string. Use `",".join(list)` instead — it never adds a trailing delimiter.
- **`"".split(",")` returns `[""]` not `[]`**: splitting an empty string with an explicit separator always yields a one-element list. Guard before splitting: `if not data: return None`.

## String Truthiness Trap
When a flag is stored as a string (`"0"` / `"1"`), `if flag` evaluates `True` for both values since all non-empty strings are truthy. Always use explicit comparison: `if flag == "1"`.

## Cross-Array Index Confusion
In problems that operate on two arrays simultaneously (e.g. preorder + inorder traversal, merge-sort style divide-and-conquer), it's tempting to reuse the boundary variables from one array to index into the other. The index spaces are independent — crossing them produces silently wrong values with no error raised.
- Before any indexing operation, ask: *which array does this index belong to, and is that the array I'm actually accessing?*
- If one array is accessed sequentially (always the next element, never random-access), replace its left/right bounds with a single advancing pointer or iterator to make the distinction impossible to confuse.

→ [[Construct Binary Tree from Preorder and Inorder Traversal]]

## Premature Optimization
Attempting to tighten edge cases, collapse indices, or squeeze complexity *before* a basic working model exists. This produces tangled logic that's harder to debug than the naïve version would have been.
- Process: (1) write the clearest solution that produces correct output, (2) trace through examples to verify, (3) optimize only after the logic is confirmed sound.
- Corollary: a clean O(N²) solution you understand beats a broken O(N) attempt every time.

## Mutable Default Argument
Using a mutable object (`[]`, `{}`, `set()`) as a default parameter value. Python evaluates defaults **once at definition time** — every call that relies on the default shares the exact same object, silently corrupting state across instances or invocations.
- Always default to `None` and initialize inside the body: `def __init__(self, children=None): self.children = children if children is not None else {}`
- Applies to any mutable type: list, dict, set, or a custom object.

→ [[Implement Trie (Prefix Tree)]]

## Nested Class Scope
A method inside a nested class cannot reference the outer class by its bare name — the outer class is **not in the inner class's scope**. Use the fully-qualified path instead: `Trie.Node(char)`, not `Node(char)`.
- Simplest fix: extract the inner class to module level, eliminating the scope issue entirely.

→ [[Implement Trie (Prefix Tree)]]

## Object vs Type Assignment
Assigning a class/type where an instance was intended — e.g. `d[key] = Node` instead of `d[key] = child`. Python raises no error; the container silently stores the class object itself rather than the constructed instance, causing downstream `AttributeError`s.
- After constructing an instance (`child = Node()`), double-check that all assignments reference the **variable**, not the class name.

→ [[Implement Trie (Prefix Tree)]]

## Copy-Paste Drift
Duplicating a nearly-identical method and forgetting to update the parameter names inside the body. The copy compiles and runs without error but silently operates on the wrong variable.
- Immediately after pasting, do a single pass replacing every occurrence of the old parameter name with the new one before writing any new logic.

→ [[Implement Trie (Prefix Tree)]]

## Data Structure Internal Type Confusion
Forgetting how the fields of your own data structure are typed — e.g., treating `children: dict` as a list and not knowing the right iteration API.
- When you define a structure (like `Node`), briefly recall its field types before writing any code that accesses them.
- For a dict: iterate keys with `.keys()`, values with `.values()`, or pairs with `.items()`. `for x in d` iterates **keys** only, not values.

→ [[Design Add and Search Words Data Structure]]

## Debugger Reliance
Reaching for the debugger before attempting to trace the call stack mentally. In an interview, a debugger isn't available — you must be able to step through your own recursion by hand.
- Before running code, trace one example by hand: write out each recursive call, its arguments, and what it returns.
- Reserve the debugger for confirming a mental model, not replacing it.

→ [[Design Add and Search Words Data Structure]]

---

## Frameworks

### State Change Checklist
Every `while` loop has a tollbooth at the end. Before moving on, highlight every pointer driving the loop condition and ask: *did all of them advance?*

### Extreme Boundary Tracing
Don't trace with large inputs. Use the smallest numbers that trigger your loop (e.g. `n = 2`, `n = 3`) and manually check index access at the boundaries.

### The 0, 1, 2 Rule
Before writing any logic, define what the algorithm returns for:
- **0 items** — handle explicitly (e.g. `if not lst: return None`)
- **1 item** — return it directly
- **2 items** — smallest input that exercises the main loop

