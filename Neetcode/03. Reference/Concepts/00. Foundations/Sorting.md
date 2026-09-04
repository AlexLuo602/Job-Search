---
type: concept
tags: [concept, dsa]
---

# Sorting

**TL;DR:** Arrange elements in order — usually O(n log n) — to expose structure (duplicates, runs, canonical forms) that turns a hard comparison problem into an easy sequential scan.

## When to reach for it
- Two collections should be compared as multisets (anagrams, "can be rearranged into") — sorting gives a canonical form to compare directly.
- Need to process elements by rank (smallest first, merge in order, greedy choice by size).
- Overlapping or "consecutive" structure needs adjacency — sorting puts related elements next to each other.
- A brute-force pairwise comparison feels like it should collapse if only the data were ordered.

## How it works
Sorting imposes a total order, and many problems become trivial once adjacent elements are guaranteed to be comparable in one pass. Trace checking whether `s = "anagram"` and `t = "nagaram"` are anagrams by sorting:

| step | value |
|---|---|
| `sorted(s)` | `['a','a','a','g','m','n','r']` |
| `sorted(t)` | `['a','a','a','g','m','n','r']` |
| compare | equal → anagrams |

Two O(n log n) sorts plus one O(n) comparison beats matching characters pairwise. The same idea drives **merge intervals**: sort intervals by start time, then walk once — because once sorted, the only interval that could possibly overlap the current one is the *previous* one in the merged result, not any arbitrary earlier interval.

## Why it works
Sorting is a one-time O(n log n) investment that buys a much stronger local property: after sorting, "related" elements (equal, adjacent in value, or overlapping in range) are guaranteed to sit next to each other in the array. That turns an all-pairs question (does *any* pair overlap/match?) into a single linear scan comparing each element only to its immediate neighbor — if two elements were related but not adjacent after sorting, something between them in sorted order would have to be related to both, contradicting minimality of the gap. For merge intervals specifically: sorting by start time guarantees that if interval `i` doesn't overlap the current merged interval, no interval after `i` can either (their start is even later), so a single forward pass never needs to look back.

## Template
```python
# Canonical form comparison (anagrams)
def is_anagram(s, t):
    return sorted(s) == sorted(t)

# Merge intervals after sorting by start
def merge(intervals):
    intervals.sort(key=lambda iv: iv[0])
    merged = [intervals[0]]
    for start, end in intervals[1:]:
        if start <= merged[-1][1]:      # overlaps the last merged interval
            merged[-1][1] = max(merged[-1][1], end)
        else:
            merged.append([start, end])
    return merged
```

## Complexity
Time: O(n log n) for the sort itself, dominating whatever O(n) work follows — Python's Timsort guarantees this bound and is stable. Space: O(n) for a sorted copy (or O(log n) for the sort's own recursion), O(1) extra if sorting in place and ignoring the sort's internal stack.

## Common pitfalls
- Sorting when only counts or membership matter — a [[Hash Map]] or [[Set]] answers those in O(n), strictly better than O(n log n).
- Forgetting a custom `key=` function, so tuples/objects sort by an unintended field (e.g., sorting intervals by end instead of start).
- Assuming sort is free — repeatedly sorting inside a loop turns an O(n log n) algorithm into O(n² log n).
- Not accounting for sort's O(log n) or O(n) extra space when a problem demands strict O(1) space.

## NeetCode examples
- [[02.MergeInterval|MergeInterval]] — sort by start time, then merge in one linear pass
- [[02.ValidAnagram|ValidAnagram]] — sort both strings and compare
- [[04.GroupAnagrams|GroupAnagrams]] — sorted characters as a canonical grouping key

## Full guide
- [[Job Search/Neetcode/01. Questions/01. Arrays & Hashing/0.ArraysAndHashingGuide|Arrays & Hashing Guide]]
- [[Job Search/Neetcode/01. Questions/16. Intervals/0.IntervalsGuide|Intervals Guide]]
