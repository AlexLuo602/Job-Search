---
type: concept
tags: [concept, dsa]
---

# Frequency Counting

**TL;DR:** Tally how many times each element appears — usually with a hash map — to turn "are these the same multiset?" or "what's most/least common?" into an O(n) scan.

## When to reach for it
- Need to compare two collections as multisets (same elements, same counts, order doesn't matter).
- "Most frequent," "least frequent," or "top k frequent" appears in the problem.
- Need to check a per-character or per-element constraint (e.g., "at most two distinct characters", "can rearrange so no two adjacent are equal").
- You'd otherwise sort just to group equal elements together — counting is O(n), sorting is O(n log n).

## How it works
A frequency map records, for each distinct value, how many times it occurs. Trace building one for `t = "nagaram"` (checking it's an anagram of `s = "anagram"`):

| char | count after processing |
|---|---|
| n | {n:1} |
| a | {n:1, a:1} |
| g | {n:1, a:1, g:1} |
| a | {n:1, a:2, g:1} |
| r | {n:1, a:2, g:1, r:1} |
| a | {n:1, a:3, g:1, r:1} |
| m | {n:1, a:3, g:1, r:1, m:1} |

Then decrement the same map once per character of `s`; if every count returns to zero, the strings are anagrams. For "top k frequent," build the frequency map first, then feed `(count, value)` pairs into a heap or bucket array — the counting step is what makes "frequent" a well-defined, computable quantity in the first place.

## Why it works
Building the map is one O(n) pass where each element does O(1) work (hash to a bucket, increment a counter) — the invariant is that after processing index `i`, the map holds the exact count of every value in `nums[0..i]`. Comparing two frequency maps for equality (or checking all counts are zero) is then O(distinct values), at most O(n). This is strictly better than sorting (O(n log n)) whenever the *only* thing that matters is counts, not order — sorting proves multiset-equality by forcing a canonical arrangement, while counting proves it more directly by comparing tallies, without paying for a total order you don't need.

## Template
```python
from collections import Counter

# Multiset equality (anagram check)
def is_anagram(s, t):
    return Counter(s) == Counter(t)

# Top k frequent elements
def top_k_frequent(nums, k):
    freq = Counter(nums)
    return [val for val, _ in freq.most_common(k)]

# Manual increment/decrement without Counter
count = {}
for x in nums:
    count[x] = count.get(x, 0) + 1
```

## Complexity
Time: O(n) to build the frequency map (each element does O(1) hash/increment work); comparing two maps or finding the max is an additional O(n) or O(n log k) if a heap is involved. Space: O(n) worst case (all distinct elements), or O(1) if the alphabet is fixed-size (e.g., 26 lowercase letters).

## Common pitfalls
- Using `dict[key] += 1` without a default — raises `KeyError` on the first occurrence; use `Counter`, `defaultdict(int)`, or `.get(key, 0)`.
- Comparing two `Counter`s built from strings of very different lengths without an early length check — the comparison is still correct, but a length mismatch is a free O(1) short-circuit you're skipping.
- Reaching for [[Sorting]] to compare multisets when a direct count comparison is simpler and asymptotically better.
- Forgetting that `Counter` treats a missing key as count 0 automatically, but a plain `dict` does not — mixing the two causes `KeyError`s.

## NeetCode examples
- [[02.ValidAnagram|ValidAnagram]] — multiset equality via counting
- [[04.GroupAnagrams|GroupAnagrams]] — counts (or sorted form) as a grouping key
- [[05.TopKElements|TopKElements]] — frequency map feeding a heap or bucket sort

## Full guide
[[Job Search/Neetcode/01. Questions/01. Arrays & Hashing/0.ArraysAndHashingGuide|Arrays & Hashing Guide]]
