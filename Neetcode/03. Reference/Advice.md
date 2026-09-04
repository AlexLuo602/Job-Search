# Advice

## How to Know Whether to Keep Optimizing

Do not search for the cleverest solution before establishing a correct baseline. Start with the clearest approach, calculate its time and space complexity, and then ask what work every correct algorithm must perform. An algorithm is **proven asymptotically optimal** when its complexity matches that lower bound. If there is no matching lower-bound argument, call the solution the standard or expected approach rather than assuming it is optimal.

Use this process:

1. **Define the exact task.** Finding one solution, counting all solutions, and returning all solutions can have different lower bounds.
2. **Write the clearest correct approach.** A concrete baseline makes unnecessary work visible.
3. **Calculate its complexity.** Identify the number of states explored and the work performed in each state.
4. **Find the lower bound.** Ask what information must be read or what output must be produced.
5. **Look for a gap.** If the solution performs more work than the lower bound, determine whether that work comes from repeated computation, impossible states, slow operations, or unnecessary stored state.

Common signs that a solution can be improved:

- **Repeated subproblems** suggest dynamic programming or memoization.
- **Impossible prefixes** suggest backtracking with earlier pruning.
- **Repeated searches or updates** suggest a better data structure.
- **Monotonic input or decisions** may allow binary search, two pointers, a monotonic stack, or a greedy approach.
- **State that only depends on the previous step** may allow space compression.

Also check the constraints. A problem with `N <= 10` often expects exponential search, while `N <= 100,000` usually requires near-linear or `O(N log N)` time. Constraints do not prove optimality, but they reveal what complexity the problem was designed to accept.

### N-Queens

For [[09.N-Queens|N-Queens]], placing queens anywhere on the board explores many boards that violate obvious constraints. Since every valid board has exactly one queen in each row, recurse by row and choose one column for that row. Track occupied columns and diagonals so each invalid branch stops as soon as it becomes impossible.

This row-by-row backtracking approach is the standard solution for returning every arrangement. The search has an `O(N!)` upper bound after enforcing one queen per row and column, and diagonal checks prune it further. If there are `S` solutions, constructing the returned boards alone costs `O(S * N²)` because each board contains `N²` characters.

This output lower bound does not prove that row-by-row backtracking explores the fewest possible failed branches. Bitmasks, symmetry, and stronger search-ordering techniques can still reduce overhead. However, once the solution places one queen per row and immediately rejects occupied columns and diagonals, it has reached the expected interview approach. At that point, implement and explain it instead of searching for an unrelated greedy or DP trick unless the interviewer asks for further optimization.

### Best Time to Buy and Sell Stock III

Stock III has `N` prices, and any correct solution must inspect each price, giving an `O(N)` time lower bound. A one-pass `O(N)` solution therefore has optimal time complexity.

The standard solution tracks four states: the best value after the first buy, first sale, second buy, and second sale. It can look greedy because each state keeps the best value seen so far, but the reasoning is dynamic programming. A transaction that looks best now is not an independently safe choice because it affects the money and opportunities available for the second transaction.

The optimization is from an `O(N)` DP table to four variables, reducing space from `O(N)` to `O(1)`. It improves space and implementation, not the already optimal `O(N)` time.

### Interview Rule

Explain the progression instead of silently hunting for the final trick:

> “I’ll start with the clearest correct approach, determine its complexity, and then check whether repeated work or unused constraints leave room for improvement.”

Matching a lower bound proves asymptotic optimality, but smaller constants, lower space use, and clearer code may still be possible. Optimize those only after correctness and the main complexity are understood.

In an interview, stop searching for a different algorithm when either:

- The solution matches a clear lower bound, so its asymptotic complexity is proven optimal.
- The solution matches the expected pattern and target complexity implied by the constraints, and the remaining improvements only change constants or representation.

Keep optimizing when an unexplained gap remains, such as repeated work, a nested scan over the same data, delayed pruning, or an `O(N)` table whose next state only needs a few previous values.
