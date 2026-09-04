---
type: concept
tags: ["concept"]
---

# Hash Map

**TL;DR:** Trade O(n) space for O(1) average lookup, turning "have I seen this?" and complement/frequency questions into a single pass.

## When to reach for it
- Need to detect duplicates or check membership in O(1).
- Problem involves counting frequencies of elements.
- Looking for pairs/groups satisfying a sum, difference, or equality condition.
- Need to group elements by a computed key (e.g., sorted characters for anagrams).
- You keep reaching for a nested loop just to ask "does X exist somewhere else in this array?"

## How it works
A hash map applies a hash function to each key to compute a bucket index, then stores the key-value pair in that bucket (a small list, to handle collisions). Lookup, insertion, and deletion all reduce to "compute the hash, jump to the bucket" — no scanning required.

The pattern that recurs across problems: walk the input once, and *before* moving on, ask the map "have I already seen the thing that would pair with me?" Trace **Two Sum** on `nums = [2, 7, 11, 15]`, `target = 9`:

| i | x | complement (9 - x) | complement in `seen`? | action | `seen` after |
|---|---|---|---|---|---|
| 0 | 2 | 7 | no | insert 2 → 0 | `{2: 0}` |
| 1 | 7 | 2 | **yes** (0) | return `[0, 1]` | — |

Only one pass, one lookup per index — no inner loop over the rest of the array.

## Why it works
Hashing distributes keys roughly uniformly across buckets, so on average each bucket holds O(1) items and a lookup touches O(1) of them — that's where the average-case O(1) comes from. Correctness of the "seen so far" trick rests on a simple loop invariant: at the top of iteration `i`, `seen` contains exactly the elements at indices `0..i-1`. So checking `complement in seen` before inserting `x` is equivalent to asking "does some *earlier* element pair with me?" — exactly what a nested loop checks, just without re-scanning. That's the whole trick: precompute a fast membership index instead of a linear search, replacing O(n) work per element with O(1).

## Template
```python
from collections import defaultdict, Counter

# Frequency count
freq = Counter(nums)

# Group by key
groups = defaultdict(list)
for val in items:
    key = compute_key(val)
    groups[key].append(val)

# Two-sum complement pattern
seen = {}
for i, x in enumerate(nums):
    complement = target - x
    if complement in seen:
        return [seen[complement], i]
    seen[x] = i
```

## Complexity
Time: O(n) average — each of the n insert/lookup operations is O(1) amortized, so a single pass over the input is linear. Space: O(n) for the map itself, since in the worst case every element gets its own entry.

## Common pitfalls
- Using a mutable object (list, dict) as a key — keys must be hashable; use `tuple()` instead.
- Overwriting an earlier index when you need the first occurrence (e.g., Two Sum: insert *after* checking, not before).
- Assuming O(1) worst-case — hash collisions (or an adversarial input against a weak hash) can degrade a bucket to O(n).
- Forgetting that dict resizing means insertion is only *amortized* O(1), not O(1) on every single call.

## NeetCode examples
- [[03.TwoSum|TwoSum]] — complement lookup in one pass
- [[04.GroupAnagrams|GroupAnagrams]] — group by sorted-tuple key
- [[09.LongestConsecutiveSequence|LongestConsecutiveSequence]] — O(n) streak detection via set
- [[05.TopKElements|TopKElements]] — frequency map fed into a heap

## Full guide
[[Job Search/Neetcode/01. Questions/01. Arrays & Hashing/0.ArraysAndHashingGuide|Arrays & Hashing Guide]]
