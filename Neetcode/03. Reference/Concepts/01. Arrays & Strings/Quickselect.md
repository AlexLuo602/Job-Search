---
type: concept
tags: [concept, dsa, pattern]
---

# Quickselect

**TL;DR:** Find the k-th smallest/largest element in average O(n) by partitioning around a pivot like quicksort, but discarding the half you don't need instead of recursing into both.

## When to reach for it
- Need the k-th smallest/largest element (or the top-k elements), but *don't* need the full array sorted.
- A [[Heap]] solution feels like overkill because you only need a single answer, not an ongoing stream.
- The problem explicitly allows average-case analysis (interviewers often accept O(n) average even though worst case is O(n²)).

## How it works
Partition the array around a pivot. Values smaller than the pivot move left, values larger than the pivot stay right, and the pivot lands at its final sorted index. Once that index is known, keep only the side containing the target index.

### Interactive walkthrough

This example finds the third largest value in `[7, 2, 1, 8, 6, 3, 5, 4]`. The target is index `8 - 3 = 5` in ascending order. The walkthrough uses the last value in the current range as the pivot so each partition is easy to follow.

```freeform
const original = [7, 2, 1, 8, 6, 3, 5, 4];
const target = 5;
const steps = [
    { nums:[7,2,1,8,6,3,5,4], lo:0, hi:7, pivotIndex:7, pivot:4, scan:null, store:0, settled:[], line:2, action:"Choose 4 as the pivot and start the store position at index 0." },
    { nums:[7,2,1,8,6,3,5,4], lo:0, hi:7, pivotIndex:7, pivot:4, scan:0, store:0, settled:[], line:3, action:"Keep 7 on the right side because it is not smaller than 4." },
    { nums:[2,7,1,8,6,3,5,4], lo:0, hi:7, pivotIndex:7, pivot:4, scan:1, store:1, settled:[], line:4, action:"Move 2 to index 0 and advance the store position to index 1." },
    { nums:[2,1,7,8,6,3,5,4], lo:0, hi:7, pivotIndex:7, pivot:4, scan:2, store:2, settled:[], line:4, action:"Move 1 to index 1 and advance the store position to index 2." },
    { nums:[2,1,3,8,6,7,5,4], lo:0, hi:7, pivotIndex:7, pivot:4, scan:5, store:3, settled:[], line:4, action:"Move 3 to index 2 and advance the store position to index 3." },
    { nums:[2,1,3,4,6,7,5,8], lo:4, hi:7, pivotIndex:3, pivot:4, scan:null, store:3, settled:[3], line:6, action:"Place 4 at index 3 and keep the right range because pivot index 3 is left of target index 5." },
    { nums:[2,1,3,4,6,7,5,8], lo:4, hi:7, pivotIndex:7, pivot:8, scan:6, store:7, settled:[3], line:4, action:"All three values are below pivot 8, so the store position moves to index 7." },
    { nums:[2,1,3,4,6,7,5,8], lo:4, hi:6, pivotIndex:7, pivot:8, scan:null, store:7, settled:[3,7], line:6, action:"Place 8 at index 7 and keep the left range because 7 is above target 5." },
    { nums:[2,1,3,4,6,7,5,8], lo:4, hi:6, pivotIndex:6, pivot:5, scan:5, store:4, settled:[3,7], line:3, action:"Keep 6 and 7 on the right because neither value is smaller than pivot 5." },
    { nums:[2,1,3,4,5,7,6,8], lo:5, hi:6, pivotIndex:4, pivot:5, scan:null, store:4, settled:[3,4,7], line:6, action:"Place 5 at index 4 and keep the right range because pivot index 4 is left of target index 5." },
    { nums:[2,1,3,4,5,7,6,8], lo:5, hi:6, pivotIndex:6, pivot:6, scan:5, store:5, settled:[3,4,7], line:3, action:"Keep 7 on the right because it is not smaller than pivot 6." },
    { nums:[2,1,3,4,5,6,7,8], lo:5, hi:5, pivotIndex:5, pivot:6, scan:null, store:5, settled:[3,4,5,7], line:6, action:"Place 6 at target index 5, so the third largest value is 6." }
];

const root = document.createElement("section");
root.style.cssText = "font:14px system-ui;max-width:720px;border:1px solid var(--background-modifier-border);border-radius:10px;padding:12px;background:var(--background-primary);color:var(--text-normal);";
root.innerHTML = `
<style>
  .qs-svg{width:100%;height:auto;display:block}
  .qs-state{display:grid;grid-template-columns:1fr 1fr;gap:8px;margin-top:8px}
  .qs-code{overflow-x:auto;font-size:13px}
  .qs-controls{display:flex;align-items:center;gap:8px;flex-wrap:wrap;margin-top:10px}
  .qs-controls button,.qs-controls select{min-height:44px;font-size:14px;touch-action:manipulation}
  .qs-controls button{padding:6px 12px}
  @media(max-width:520px){.qs-state{grid-template-columns:1fr}.qs-code{font-size:12px}.qs-controls button{flex:1 1 72px}.qs-speed{flex:1 1 100%;display:flex;align-items:center;gap:8px}.qs-speed select{flex:1}.qs-count{width:100%;text-align:center;margin-left:0!important}}
</style>
<svg class="qs-svg" viewBox="0 0 430 292" preserveAspectRatio="xMidYMid meet" role="img"></svg>
<div style="display:flex;gap:12px;flex-wrap:wrap;margin:2px 0 8px;font-size:12px"><span>P = pivot</span><span>S = next store index</span><span>i = checked index</span><span>T = target index</span></div>
<div data-role="status" aria-live="polite" style="min-height:22px;padding:9px 10px;border-radius:8px;background:var(--background-secondary)"></div>
<div class="qs-state">
  <div style="padding:8px;border:1px solid var(--background-modifier-border);border-radius:8px"><strong>Bounds and target</strong><div data-role="bounds" style="margin-top:4px;font-family:var(--font-monospace)"></div></div>
  <div style="padding:8px;border:1px solid var(--background-modifier-border);border-radius:8px"><strong>Pivot and store</strong><div data-role="partition" style="margin-top:4px;font-family:var(--font-monospace)"></div></div>
</div>
<pre class="qs-code" data-role="code" style="padding:8px;border-radius:8px;background:var(--background-secondary)"><code><span data-line="1">while lo &lt;= hi:</span>
<span data-line="2">    pivot = nums[hi]; store = lo</span>
<span data-line="3">    for i in range(lo, hi):</span>
<span data-line="4">        if nums[i] &lt; pivot: swap; store += 1</span>
<span data-line="5">    swap(nums[store], nums[hi])</span>
<span data-line="6">    return or keep the target side</span></code></pre>
<div class="qs-controls">
  <button type="button" data-action="previous">Previous</button><button type="button" data-action="play">Play</button><button type="button" data-action="next">Next</button><button type="button" data-action="reset">Reset</button>
  <label class="qs-speed">Speed <select data-action="speed"><option value="1400">Slow</option><option value="900" selected>Normal</option><option value="500">Fast</option></select></label>
  <span class="qs-count" data-role="counter" style="margin-left:auto"></span>
</div>`;

const svg = root.querySelector("svg");
const ns = "http://www.w3.org/2000/svg";
let index = 0;
let timer = null;

function svgNode(name, attrs, text) {
    const node = document.createElementNS(ns, name);
    for (const [key, value] of Object.entries(attrs)) node.setAttribute(key, String(value));
    if (text !== undefined) node.textContent = text;
    return node;
}

function stopPlaying() {
    if (timer !== null) clearInterval(timer);
    timer = null;
    root.querySelector('[data-action="play"]').textContent = "Play";
}

function renderArray(step) {
    svg.replaceChildren();
    svg.appendChild(svgNode("text", {x:215,y:22,"text-anchor":"middle","font-size":17,"font-weight":700,fill:"#17202a"}, "Third largest, target index 5"));
    step.nums.forEach((value, i) => {
        const col = i % 4;
        const row = Math.floor(i / 4);
        const x = 52 + col * 95;
        const y = 68 + row * 118;
        const inRange = i >= step.lo && i <= step.hi;
        const isSettled = step.settled.includes(i);
        let fill = inRange ? "#d6eaf8" : "#e5e7e9";
        if (isSettled) fill = "#abebc6";
        if (i === step.pivotIndex) fill = "#d7bde2";
        if (i === target && step.settled.includes(target)) fill = "#82e0aa";
        svg.appendChild(svgNode("rect", {x,y,width:58,height:48,rx:7,fill,stroke:i===target?"#1e8449":"#34495e","stroke-width":i===target?3:1.5}));
        svg.appendChild(svgNode("text", {x:x+29,y:y+30,"text-anchor":"middle","font-size":18,"font-weight":700,fill:"#17202a"}, value));
        svg.appendChild(svgNode("text", {x:x+29,y:y+66,"text-anchor":"middle","font-size":13,fill:"#34495e"}, "index " + i));
        const marks = [];
        if (i === step.pivotIndex) marks.push("P");
        if (i === step.store) marks.push("S");
        if (i === step.scan) marks.push("i");
        if (i === target) marks.push("T");
        if (marks.length) svg.appendChild(svgNode("text", {x:x+29,y:y-9,"text-anchor":"middle","font-size":14,"font-weight":700,fill:"#922b21"}, marks.join(" ")));
    });
    svg.appendChild(svgNode("text", {x:215,y:284,"text-anchor":"middle","font-size":13,"font-weight":600,fill:"#34495e"}, "Blue = current range | Green = fixed pivot | Gray = discarded range"));
}

function render() {
    const step = steps[index];
    renderArray(step);
    svg.setAttribute("aria-label", "Quickselect step " + (index + 1) + ". " + step.action + " Current bounds " + step.lo + " through " + step.hi + ".");
    root.querySelector('[data-role="status"]').textContent = step.action;
    root.querySelector('[data-role="bounds"]').textContent = "lo=" + step.lo + " | hi=" + step.hi + " | target=5";
    root.querySelector('[data-role="partition"]').textContent = "pivot=" + step.pivot + " | store=" + step.store;
    for (const line of root.querySelectorAll('[data-role="code"] [data-line]')) {
        const active = Number(line.getAttribute("data-line")) === step.line;
        line.style.display = "block";
        line.style.background = active ? "#f8c471" : "transparent";
        line.style.color = active ? "#17202a" : "inherit";
        line.style.fontWeight = active ? "700" : "400";
        if (active) line.setAttribute("aria-current", "step"); else line.removeAttribute("aria-current");
    }
    root.querySelector('[data-role="counter"]').textContent = "Step " + (index + 1) + " / " + steps.length;
    root.querySelector('[data-action="previous"]').disabled = index === 0;
    root.querySelector('[data-action="next"]').disabled = index === steps.length - 1;
}

root.querySelector('[data-action="previous"]').addEventListener("click", () => { stopPlaying(); index = Math.max(0, index - 1); render(); });
root.querySelector('[data-action="next"]').addEventListener("click", () => { stopPlaying(); index = Math.min(steps.length - 1, index + 1); render(); });
root.querySelector('[data-action="reset"]').addEventListener("click", () => { stopPlaying(); index = 0; render(); });
root.querySelector('[data-action="play"]').addEventListener("click", () => {
    if (timer !== null) { stopPlaying(); return; }
    if (index === steps.length - 1) index = 0;
    root.querySelector('[data-action="play"]').textContent = "Pause";
    const delay = Number(root.querySelector('[data-action="speed"]').value);
    timer = setInterval(() => { if (index === steps.length - 1) { stopPlaying(); return; } index += 1; render(); }, delay);
    render();
});
root.querySelector('[data-action="speed"]').addEventListener("change", () => { if (timer === null) return; stopPlaying(); root.querySelector('[data-action="play"]').click(); });

display(root);
render();
```

For this example, target index `5` contains `6`.

## Why it works
Quicksort is O(n log n) because after partitioning, it recurses into *both* halves — total work across all levels is O(n) per level times O(log n) levels. Quickselect only needs *one* answer, and after partitioning you know exactly which side it's in (everything on the other side is either all-smaller or all-larger, and provably can't contain the k-th element), so you discard the other half entirely instead of recursing into it. That turns the recurrence from `T(n) = 2T(n/2) + O(n)` (quicksort) into `T(n) = T(n/2) + O(n)` (quickselect), which solves to O(n) on average by the same geometric-series argument: the work at each level shrinks by half, so the total is dominated by the first level. The O(n²) worst case appears when partitioning is maximally unbalanced (e.g., already-sorted input with a naive last-element pivot) — random or median-of-three pivot selection makes that pathological case vanishingly unlikely in practice.

## Template
```python
import random

def quickselect(nums, k):
    """Return the k-th largest element (1-indexed)."""
    target = len(nums) - k     # index of k-th largest in ascending sorted order

    def partition(lo, hi):
        pivot_idx = random.randint(lo, hi)
        nums[pivot_idx], nums[hi] = nums[hi], nums[pivot_idx]
        pivot = nums[hi]
        store = lo
        for i in range(lo, hi):
            if nums[i] < pivot:
                nums[i], nums[store] = nums[store], nums[i]
                store += 1
        nums[store], nums[hi] = nums[hi], nums[store]
        return store

    lo, hi = 0, len(nums) - 1
    while True:
        p = partition(lo, hi)
        if p == target:
            return nums[p]
        elif p < target:
            lo = p + 1
        else:
            hi = p - 1
```

## Complexity
Time: O(n) average — each recursive call only processes one side, and the total work across shrinking segments sums to O(n) (geometric series); O(n²) worst case with a consistently bad pivot choice. Space: O(1) extra if partitioning in place (O(log n) average recursion depth if implemented recursively).

## Common pitfalls
- Confusing "k-th largest" with the target *index* in ascending order — an off-by-one here silently returns the wrong element.
- Using a fixed pivot (always first or last element) on data that's already sorted or reverse-sorted — this triggers the O(n²) worst case; randomize the pivot instead.
- Recursing into both partitions out of habit (copying quicksort) instead of only the side containing the target index — that's what makes it O(n) instead of O(n log n).
- Reaching for quickselect when a [[Heap]] of size k is simpler to reason about and reuse for streaming input (quickselect needs the whole array up front).

## NeetCode examples
- [[04.KthLargestElementInAnArray|KthLargestElementInAnArray]] — canonical quickselect application
- [[03.KClosestPointsToOrigin|KClosestPointsToOrigin]] — partition by distance instead of raw value

## Full guide
[[Job Search/Neetcode/01. Questions/09. Heap or PriorityQueue/0.HeapOrPriorityQueueGuide|Heap or PriorityQueue Guide]]
