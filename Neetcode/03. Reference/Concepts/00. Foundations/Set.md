---
type: concept
tags: [concept, dsa]
---

# Set

**TL;DR:** A hash set is a hash map with the values thrown away — O(1) average membership, insertion, and deletion, nothing else.

## When to reach for it
- You only care *whether* something exists, not any value attached to it — "have I seen this before?", "is this a duplicate?".
- Deduplicating a collection, or computing set operations (union, intersection, difference) between two collections.
- Checking a constraint naturally phrased as "no two of these can be equal" (e.g., no repeated character in a substring, no repeated digit in a Sudoku row).
- You're about to write a nested loop purely to check "does this value appear elsewhere?" — a set answers that in O(1) instead of O(n).

## How it works
Internally a set is exactly a hash map's bucket structure with the values dropped — each key hashes to a bucket, and membership is "is this bucket's slot occupied by an equal key?" Trace checking `nums = [1, 2, 3, 1]` for a duplicate:

| i | nums[i] | in `seen`? | action | `seen` after |
|---|---|---|---|---|
| 0 | 1 | no | add | `{1}` |
| 1 | 2 | no | add | `{1, 2}` |
| 2 | 3 | no | add | `{1, 2, 3}` |
| 3 | 1 | **yes** | return `True` | — |

No inner loop, no sorting — one pass, one O(1) check per element. The same shape solves "longest consecutive sequence": put every number in a set, then for each number with no predecessor (`n - 1 not in seen`), walk forward (`n+1`, `n+2`, ...) counting how far the run extends, using set membership instead of a sorted scan.

## Why it works
Like a hash map, a set relies on hashing to distribute elements across buckets, so membership checks touch O(1) buckets on average rather than scanning a list. The invariant that makes duplicate detection correct: at the moment you check `x in seen`, `seen` contains precisely the elements processed strictly before the current one — so a hit means some *earlier* element equals `x`, which is the definition of a duplicate. For "longest consecutive sequence," the trick that keeps it O(n) instead of O(n²) is the `n - 1 not in seen` guard: it ensures each run is only *started* from its smallest element, so every number is visited as part of at most one run's inner while-loop, even though the outer loop touches every number.

## Template
```python
# Membership / duplicate check
seen = set()
for x in nums:
    if x in seen:
        return True   # duplicate found
    seen.add(x)
return False

# Longest consecutive run using set membership
num_set = set(nums)
best = 0
for n in num_set:
    if n - 1 not in num_set:       # only start a run at its smallest element
        length = 1
        while n + length in num_set:
            length += 1
        best = max(best, length)
```

## Complexity
Time: O(n) — every element does O(1) average work (hash, insert, or check), and the consecutive-run version still totals O(n) because each number is only ever extended as part of one run. Space: O(n) to store the set.

## Common pitfalls
- Using a list instead of a set for membership checks — `x in list` is O(n), silently turning an intended O(n) algorithm into O(n²).
- Forgetting sets are unordered — you can't rely on iteration order or index into a set.
- Mutating a set while iterating over it, which raises a runtime error or skips elements.
- Reaching for a set when you actually need counts — a [[Frequency Counting|frequency map]] is the right tool when "how many times" matters, since a set can't tell you that.

## NeetCode examples
- [[01.ContainsDuplicate|ContainsDuplicate]] — textbook membership check
- [[07.ValidSodoku|ValidSodoku]] — one set per row/column/box to check uniqueness
- [[09.LongestConsecutiveSequence|LongestConsecutiveSequence]] — O(n) streak detection anchored on run-start membership

## Full guide
[[Job Search/Neetcode/01. Questions/01. Arrays & Hashing/0.ArraysAndHashingGuide|Arrays & Hashing Guide]]
