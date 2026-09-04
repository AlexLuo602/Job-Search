---
type: concept
tags: [concept, dsa, pattern]
---

# Interval Scheduling

**TL;DR:** Problems over ranges `[start, end]` almost always reduce to sorting by one endpoint and then making a single linear pass — the sort order you pick determines what the pass can answer.

## When to reach for it
- The input is a list of `[start, end]` pairs, and the question asks about overlap, merging, counting conflicts, or selecting a maximum compatible subset.
- Phrasing like "can all meetings be attended", "merge these ranges", "minimum number to remove", or "how many rooms/resources are needed at once".
- You catch yourself writing an O(n²) pairwise-comparison loop over the intervals — sorting almost always buys an O(n) scan instead.

## How it works
1. **Identify what's being asked** — this decides which endpoint to sort by, not the other way around.
2. **Sort by that endpoint.** Sorting turns a global, pairwise overlap question into a *local*, adjacent-pair question.
3. **Sweep once, tracking a small piece of running state** — a running end boundary, a count, or a running concurrency total, depending on which specific technique (below) the problem calls for.

**Trace — detecting whether any two intervals overlap**, input `[[4,7],[1,3],[6,9]]`:

Sort by start → `[1,3], [4,7], [6,9]`. Scan while tracking `last_end`:

| Interval | start vs. last_end | Overlap? | last_end after |
|---|---|---|---|
| [1, 3] | — (first) | — | 3 |
| [4, 7] | 4 > 3 | no | 7 |
| [6, 9] | 6 ≤ 7 | **yes** | 9 (or merge to 9 if merging) |

Sorting by start made the overlap between `[4,7]` and `[6,9]` detectable by comparing only *adjacent* entries — no pair further apart needed to be checked, because anything before `[4,7]` in sorted order ends no later than `[4,7]` does.

This concept card is the umbrella; the exact mechanism for the sweep step is one of three specialized concepts:

| Need | Sort by | Mechanism |
|---|---|---|
| Merge / detect any overlap | start | extend a running group's end with `max(group_end, end)` — see [[0.GreedyGuide\|Greedy]] |
| Max non-overlapping count / min removals | end | greedily keep each interval that doesn't conflict with the last kept one — see [[0.GreedyGuide\|Greedy]] |
| Peak concurrency / room count | event time | convert to `+1`/`-1` events and track a running total — see [[Sweep Line]], or a min-heap of end times — see [[Heap]] |

## Why it works
The correctness of "sort, then look only at adjacent entries" rests on one fact: once intervals are sorted by start, if interval `i` doesn't overlap the interval immediately before it in sorted order, it cannot overlap anything earlier either — every earlier interval's relevant boundary is no further along than the one directly preceding `i`. That's what collapses an O(n²) all-pairs check into an O(n) adjacent scan. Which specific boundary you track (`last_end` for merging, `prev_end` for greedy selection, a running signed count for concurrency) depends on the sub-technique, but all three share this same sorting-buys-locality argument.

## Template
```python
# General overlap-detection / merge skeleton
intervals.sort(key=lambda x: x[0])
merged = []
for start, end in intervals:
    if merged and start <= merged[-1][1]:
        merged[-1][1] = max(merged[-1][1], end)   # extend, don't just overwrite
    else:
        merged.append([start, end])
```

## Complexity
Time: O(n log n) — the sort dominates; the scan itself is O(n).
Space: O(n) for the sort (or O(1) extra if sorting in place and not merging into a new list), plus O(n) for any output list.

## Common pitfalls
- Sorting by the wrong endpoint for the sub-technique in play — merging needs sort-by-start, maximizing a non-overlapping count needs sort-by-end; using the other produces a plausible-looking but wrong answer.
- Treating "detect any overlap" and "merge overlapping intervals" as the same operation — merging requires tracking the running `max` end across the whole group, not just the previous interval's own end, or a shorter interval nested inside a longer one will incorrectly shrink the group.
- Being inconsistent about whether touching endpoints count as overlapping (`<` vs `<=`) — decide once and apply it to every comparison in the scan.
- Reaching for this pattern on subarray problems (`nums[i:j]`) that aren't explicit `[start, end]` ranges — that's [[0.SlidingWindowGuide|Sliding Window]] or [[Dynamic Programming]] territory, not interval scheduling.

## NeetCode examples
- [[02.MergeInterval|MergeInterval]] — sort by start, merge
- [[03.Non-OverlappingIntervals|Non-OverlappingIntervals]] — sort by end, greedy count
- [[04.MeetingRooms|MeetingRooms]] — sort by start, adjacent-overlap check
- [[01.InsertInterval|InsertInterval]] — already sorted input, three-phase scan

## Full guide
[[Job Search/Neetcode/01. Questions/16. Intervals/0.IntervalsGuide|Intervals Guide]]
