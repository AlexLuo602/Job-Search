---
type: concept
tags: ["concept"]
---

# Sweep Line

**TL;DR:** Decompose each interval into a `+1` start event and a `-1` end event, sort all events chronologically, and sweep through while tracking a running count — the running total (or its peak) answers concurrency questions in O(n log n).

## When to reach for it
- "How many meeting rooms/resources are needed at once", "maximum overlap", "peak concurrency" — anything asking for a running or peak *count*, not the merged ranges themselves.
- You don't need to know *which* intervals overlap, only *how many* do at any moment — if you need identity, reach for a [[Heap]] of end times instead.
- More generally: any problem where sorting entities by one coordinate and processing them strictly in that order lets a small running state answer the question — this generalizes past literal time. Sorting cars by position and sweeping (Car Fleet) or sorting building edges by x-coordinate and tracking a running max height (skyline-style problems) are the same idea applied to a different axis.

## How it works
1. Decompose each interval `[start, end)` into two events: `(start, +1)` and `(end, -1)`.
2. Sort all events by time. **Break ties by processing `-1` before `+1`** at the same timestamp — an interval ending at `t` frees its slot before one starting at `t` claims it. Flip this rule if your problem defines touching intervals as overlapping.
3. Sweep through the sorted events left to right, adding each event's delta to a running counter.
4. Read off whatever the problem needs from that counter — its maximum value over the sweep (peak concurrency), or its value at specific query points.

**Trace — peak concurrency for intervals `[1,5], [3,7], [6,9]`:**

Raw events: `(1,+1), (5,-1), (3,+1), (7,-1), (6,+1), (9,-1)`

Sorted by `(time, delta)` with `-1` before `+1` on ties: `(1,+1), (3,+1), (5,-1), (6,+1), (7,-1), (9,-1)`

| Event | curr | peak |
|---|---|---|
| (1, +1) | 1 | 1 |
| (3, +1) | 2 | 2 |
| (5, −1) | 1 | 2 |
| (6, +1) | 2 | 2 |
| (7, −1) | 1 | 2 |
| (9, −1) | 0 | 2 |

Peak = 2. Sanity check: at time 4, `[1,5]` and `[3,7]` are both active (2); at time 6.5, `[3,7]` and `[6,9]` are both active (2) since `[1,5]` has already ended — no moment ever has all three active, so 2 is correct.

## Why it works
Every interval contributes exactly one `+1` (at its start) and one `-1` (at its end) to the event stream. Summing deltas up to any time `t`, in chronological order, telescopes to exactly "number of intervals started by `t`" minus "number of intervals ended by `t`" — which is precisely the number of intervals active at `t`. Sorting is what makes "in chronological order" meaningful; without it the running sum has no relationship to concurrency at any real point in time. The tie-break rule (`-1` before `+1`) is where the open/closed boundary semantics of the problem actually live — it decides whether an interval ending exactly when another starts counts as an overlap.

Because the argument only relies on "sort by one coordinate, sweep in that order, maintain a running derived quantity," it transfers directly to problems that aren't phrased as time intervals at all: [[06.CarFleet|CarFleet]] sorts by starting position and sweeps from the car closest to the destination backward, tracking arrival time as the running state instead of a count.

## Template
```python
def max_overlap(intervals):
    events = []
    for start, end in intervals:
        events.append((start, 1))
        events.append((end, -1))
    # -1 before +1 on ties: an interval ending at t frees up
    # before one starting at t claims it (flip if touching should overlap)
    events.sort(key=lambda e: (e[0], e[1]))

    curr = peak = 0
    for _time, delta in events:
        curr += delta
        peak = max(peak, curr)
    return peak
```

## Complexity
Time: O(n log n) — dominated by sorting the 2n events; the sweep itself is a single O(n) pass.
Space: O(n) for the events list.

## Common pitfalls
- Wrong tie-break direction: sorting `+1` before `-1` at equal timestamps over-counts touching intervals as overlapping (or under-counts, if the problem intends the opposite) — pin down the boundary semantics before writing the sort key.
- Forgetting to sort after generating the events — an unsorted event stream makes the running count meaningless.
- Using sweep line when the problem needs to know *which* intervals overlap, not just how many — sweep line only tracks a count; switch to a [[Heap]] of end times when identity matters.
- Off-by-one between inclusive `[start, end]` and half-open `[start, end)` interval semantics — decide whether the `-1` event should fire at `end` or `end + 1`, and apply it consistently.

## NeetCode examples
- [[05.MeetingRoomsII|MeetingRoomsII]] — canonical event-based peak-concurrency problem
- [[06.MinimumIntervalToIncludeEachQuery|MinimumIntervalToIncludeEachQuery]] — chronological sweep over sorted queries, paired with a heap
- [[06.CarFleet|CarFleet]] — generalization, not a +1/-1 event sweep: sort by position and sweep once, carrying arrival time as the running state

## Full guide
[[Job Search/Neetcode/01. Questions/16. Intervals/0.IntervalsGuide|Intervals Guide]]
