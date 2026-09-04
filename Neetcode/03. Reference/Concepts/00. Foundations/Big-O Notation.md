---
type: concept
tags: [concept, dsa, complexity]
---

# Big-O Notation

**TL;DR:** Big-O describes how an algorithm's cost grows as input size `n` grows, stripped of constants and lower-order terms, so you can compare approaches independent of hardware.

## When to reach for it
- Before coding, to sanity-check whether an approach can possibly fit the constraints (`n ≤ 10^5` rules out O(n²)).
- Comparing two candidate solutions when you're not sure which is "actually better."
- Explaining out loud, in an interview, *why* a solution is fast enough — not just stating the bound but showing where it comes from.
- Whenever you see the words "as large as," "up to," or a numeric constraint in a problem — that number is telling you the target complexity class.

## How to derive it
Big-O isn't something you memorize per problem — you derive it from the shape of the code.

**Loop counting.** A single loop over `n` elements doing O(1) work per iteration is O(n). Nested loops multiply: a loop over `n` inside a loop over `m` is O(n·m); two nested loops both over `n` is O(n²). Sequential (not nested) loops add, and addition of two different terms simplifies to the larger one: a loop over `n` followed by a separate loop over `m` is O(n + m), not O(n·m).

**Halving → log.** Any process that discards a constant fraction of the remaining input each step (not a fixed number of elements — a *fraction*) takes O(log n) steps, because you can only halve `n` about `log₂ n` times before reaching 1. [[0.BinarySearchGuide|Binary Search]] and balanced-tree operations get their O(log n) this way — see [[01.BinarySearch|BinarySearch]] for the canonical trace.

**Recursion trees.** Draw one node per recursive call, with children for the calls it makes; total work is the sum across every node. A recursion that halves the input and does O(1) extra work per call ([[0.BinarySearchGuide|Binary Search]]) has O(log n) nodes, each O(1) — O(log n) total. A recursion that halves the input but does O(n) extra work per call (merge sort's combine step) has `log n` levels each costing O(n) — O(n log n) total. A recursion that branches into two full-size subproblems without shrinking (naive recomputation) blows up exponentially. See [[Recursion]] and [[Divide and Conquer]] for worked derivations of each shape.

**Amortized analysis.** Some operations are expensive occasionally but cheap on average over a sequence of calls. Appending to a Python list is O(1) *amortized* — most appends are O(1), but occasionally the underlying array must be resized and copied, an O(n) operation. Averaged over `n` appends, the total cost is still O(n), so each append is O(1) amortized even though a single call can spike to O(n).

## Common interview traps
- **String concatenation in a loop.** `s = s + c` inside a loop over `n` characters is O(n²), because strings are immutable in Python — each `+` copies the entire string so far. Use `''.join(list_of_chars)` to get O(n).
- **Slicing.** `arr[i:]` or `arr[:k]` creates a new list and copies every element in the slice — O(k), not O(1). A loop that slices at every iteration silently turns an O(n) scan into O(n²).
- **Hidden work inside "simple" operations.** `x in some_list` is O(n) (linear scan); `x in some_set` or `some_dict` is O(1) average. Calling `in` on a list inside a loop is a very common accidental O(n²).
- **Forgetting recursion's stack space.** A recursive solution's *time* complexity might look fine, but its *space* complexity includes the call stack — a recursive DFS down a skewed structure can be O(n) space even though an equivalent iterative version with an explicit stack has the same bound but makes it visible.

## Complexity classes, fast to slow
- O(1) constant
- O(log n) logarithmic — halving each step, see [[01.BinarySearch|BinarySearch]]
- O(n) linear
- O(n log n) sort-bound — halving with linear combine work, see [[Divide and Conquer]]
- O(n²) quadratic — nested loops over the same input
- O(2ⁿ), O(n!) exponential / factorial — brute-force search, backtracking without pruning

For the full table of data-structure and algorithm complexities, see [[Big-O Cheat Sheet]] — this page covers *how to derive* a bound; that one is the lookup reference.

## Common pitfalls
- Quoting the average-case bound when the problem's worst case is what actually matters (or vice versa) — always state which case you mean.
- Treating O(n + m) as O(n·m) out of habit — sequential work adds, only nested work multiplies.
- Dropping a term that isn't actually lower-order for the problem's real constraints (e.g. treating O(n log n) and O(n) as "basically the same" when `n` is 10^8).
- Ignoring the space cost of "clean" recursive or slicing-heavy code that has a deceptively simple time bound.

## See also
- [[Big-O Cheat Sheet]] — quick-reference complexity table
- [[Space-Time Tradeoff]] — trading memory for a better time bound
- [[Recursion]] — recursion-tree complexity in depth
- [[Divide and Conquer]] — where the O(n log n) shape comes from
