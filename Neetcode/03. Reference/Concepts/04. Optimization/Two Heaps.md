---
type: concept
tags: [concept, dsa, pattern]
---

# Two Heaps

**TL;DR:** Split a stream into a max-heap of the smaller half and a min-heap of the larger half, kept balanced in size, so the median sits at one (or both) roots at all times.

## When to reach for it
- "Running median" / "median so far" over a stream of numbers arriving one at a time.
- Any problem needing fast access to both the largest small value and the smallest large value, not just one minimum or maximum.
- A [[Heap]] alone isn't enough because you need two different rankings (top of the lower half and bottom of the upper half) updated online as data arrives.

## How it works
Maintain two heaps:
- `lo`: a max-heap (Python uses negated values) holding the smaller half of the numbers seen so far.
- `hi`: a min-heap holding the larger half.

To insert a number, always route it through `lo` first, then rebalance:
1. Push the new value onto `lo`.
2. Pop `lo`'s max and push it onto `hi` (this guarantees every value in `lo` is ≤ every value in `hi`, even if the new number actually belonged in the upper half).
3. If `hi` now has more elements than `lo`, pop `hi`'s min and push it back onto `lo`.

This keeps `len(lo) == len(hi)` or `len(lo) == len(hi) + 1`, with `lo` always holding the extra element when the count is odd.

### Interactive walkthrough

This walkthrough inserts the stream `5, 15, 1, 3, 8, 7`. Each step shows the final heap state for one insertion and labels every value moved between heaps.

```freeform
const stream = [5, 15, 1, 3, 8, 7];
const steps = [
    { insert: 5, lower: [5], upper: [], move: "5: lower -> upper -> lower", right: [5], left: [5], median: 5, line: 5, action: "Insert 5 and move it back to lower so lower keeps the extra value." },
    { insert: 15, lower: [5], upper: [15], move: "15: lower -> upper", right: [15], left: [], median: 10, line: 3, action: "Insert 15 and move it to upper, which balances the two heap sizes." },
    { insert: 1, lower: [5, 1], upper: [15], move: "5: lower -> upper -> lower", right: [5], left: [5], median: 5, line: 5, action: "Insert 1 and move 5 back to lower so lower keeps the extra value." },
    { insert: 3, lower: [3, 1], upper: [5, 15], move: "5: lower -> upper", right: [5], left: [], median: 4, line: 3, action: "Insert 3 and move 5 to upper, which balances the two heap sizes." },
    { insert: 8, lower: [5, 3, 1], upper: [8, 15], move: "8: lower -> upper; 5: upper -> lower", right: [8], left: [5], median: 5, line: 5, action: "Insert 8, move 8 to upper, and return 5 to lower to keep the sizes valid." },
    { insert: 7, lower: [5, 3, 1], upper: [7, 8, 15], move: "7: lower -> upper", right: [7], left: [], median: 6, line: 6, action: "Insert 7 and move it to upper, which leaves both heaps with three values." },
];

const root = document.createElement("section");
root.style.cssText = "font:14px system-ui; max-width:720px; border:1px solid var(--background-modifier-border); border-radius:10px; padding:12px; background:var(--background-primary); color:var(--text-normal);";
root.innerHTML = `
    <style>
        .heaps-graph { width:100%; height:auto; display:block; }
        .heaps-state-grid { display:grid; grid-template-columns:1fr 1fr; gap:8px; margin-top:8px; }
        .heaps-code { overflow-x:auto; font-size:13px; }
        .heaps-controls { display:flex; align-items:center; gap:8px; flex-wrap:wrap; margin-top:10px; }
        .heaps-controls button, .heaps-controls select { min-height:44px; font-size:14px; touch-action:manipulation; }
        .heaps-controls button { padding:6px 12px; }
        @media (max-width:520px) {
            .heaps-state-grid { grid-template-columns:1fr; }
            .heaps-code { font-size:12px; }
            .heaps-controls { gap:6px; }
            .heaps-controls button { flex:1 1 72px; }
            .heaps-speed { flex:1 1 100%; display:flex; align-items:center; gap:8px; }
            .heaps-speed select { flex:1; }
            .heaps-counter { width:100%; margin-left:0 !important; text-align:center; }
        }
    </style>
    <svg class="heaps-graph" viewBox="0 0 560 335" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Interactive two heaps stepper">
        <defs>
            <marker id="heaps-arrow-right" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" fill="#2471a3"></path></marker>
            <marker id="heaps-arrow-left" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" fill="#6c3483"></path></marker>
        </defs>
        <text x="280" y="23" text-anchor="middle" font-size="18" font-weight="700" fill="#17202a">Stream: 5, 15, 1, 3, 8, 7</text>
        <g data-layer="stream"></g>
        <rect x="32" y="112" width="220" height="140" rx="12" fill="#eaf2f8" stroke="#2471a3" stroke-width="3"></rect>
        <rect x="308" y="112" width="220" height="140" rx="12" fill="#f4ecf7" stroke="#6c3483" stroke-width="3"></rect>
        <text x="142" y="139" text-anchor="middle" font-size="17" font-weight="700" fill="#154360">Lower max-heap</text>
        <text x="418" y="139" text-anchor="middle" font-size="17" font-weight="700" fill="#512e5f">Upper min-heap</text>
        <g data-layer="lower"></g>
        <g data-layer="upper"></g>
        <g data-layer="moves"></g>
        <text x="280" y="321" text-anchor="middle" font-size="17" font-weight="700" fill="#17202a" data-role="move-label"></text>
    </svg>
    <div style="display:flex; gap:12px; flex-wrap:wrap; margin:0 0 8px; font-size:12px;">
        <span>Orange border = inserted value</span><span>Blue arrow = move to upper</span><span>Purple arrow = move to lower</span>
    </div>
    <div data-role="status" aria-live="polite" style="min-height:22px; padding:9px 10px; border-radius:8px; background:var(--background-secondary);"></div>
    <div class="heaps-state-grid">
        <div style="padding:8px; border:1px solid var(--background-modifier-border); border-radius:8px;"><strong>Heap values, root first</strong><div data-role="heap-state" style="margin-top:4px; font-family:var(--font-monospace);"></div></div>
        <div style="padding:8px; border:1px solid var(--background-modifier-border); border-radius:8px;"><strong>Size rule and median</strong><div data-role="balance" style="margin-top:4px; font-family:var(--font-monospace);"></div></div>
    </div>
    <div class="heaps-code" data-role="code" style="margin-top:8px; padding:8px 10px; border-radius:8px; background:var(--background-secondary); font-family:var(--font-monospace); line-height:1.55;">
        <div data-line="1">1&nbsp; for num in stream</div>
        <div data-line="2">2&nbsp;&nbsp;&nbsp; push num into lower</div>
        <div data-line="3">3&nbsp;&nbsp;&nbsp; move lower max to upper</div>
        <div data-line="4">4&nbsp;&nbsp;&nbsp; if upper has more values</div>
        <div data-line="5">5&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; move upper min to lower</div>
        <div data-line="6">6&nbsp;&nbsp;&nbsp; read median from heap roots</div>
    </div>
    <div class="heaps-controls">
        <button data-action="previous" aria-label="Previous step">Previous</button>
        <button data-action="play">Play</button>
        <button data-action="next" aria-label="Next step">Next</button>
        <button data-action="reset">Reset</button>
        <label class="heaps-speed" style="margin-left:8px;">Speed <select data-action="speed"><option value="1200">Slow</option><option value="700" selected>Normal</option><option value="350">Fast</option></select></label>
        <strong class="heaps-counter" data-role="counter" style="margin-left:auto;"></strong>
    </div>
`;

const ns = "http://www.w3.org/2000/svg";
const svg = root.querySelector("svg");
const streamLayer = root.querySelector('[data-layer="stream"]');
const lowerLayer = root.querySelector('[data-layer="lower"]');
const upperLayer = root.querySelector('[data-layer="upper"]');
const moveLayer = root.querySelector('[data-layer="moves"]');

stream.forEach((value, i) => {
    const x = 60 + i * 88;
    const group = document.createElementNS(ns, "g");
    group.setAttribute("data-stream-index", String(i));
    group.innerHTML = `<circle cx="${x}" cy="70" r="24" fill="#f2f3f4" stroke="#566573" stroke-width="3"></circle><text x="${x}" y="71" text-anchor="middle" dominant-baseline="central" font-size="17" font-weight="700" fill="#17202a">${value}</text>`;
    streamLayer.appendChild(group);
});

function drawHeap(layer, values, side, inserted) {
    layer.replaceChildren();
    const positions = side === "lower" ? [[142, 172], [92, 222], [192, 222]] : [[418, 172], [368, 222], [468, 222]];
    values.forEach((value, i) => {
        const [x, y] = positions[i];
        if (i > 0) {
            const [rootX, rootY] = positions[0];
            const line = document.createElementNS(ns, "line");
            line.setAttribute("x1", String(rootX));
            line.setAttribute("y1", String(rootY + 22));
            line.setAttribute("x2", String(x));
            line.setAttribute("y2", String(y - 22));
            line.setAttribute("stroke", side === "lower" ? "#2471a3" : "#6c3483");
            line.setAttribute("stroke-width", "3");
            layer.appendChild(line);
        }
        const group = document.createElementNS(ns, "g");
        const isInserted = value === inserted;
        group.innerHTML = `<circle cx="${x}" cy="${y}" r="23" fill="${side === "lower" ? "#85c1e9" : "#d7bde2"}" stroke="${isInserted ? "#d35400" : side === "lower" ? "#1f618d" : "#6c3483"}" stroke-width="${isInserted ? "5" : "3"}"></circle><text x="${x}" y="${y + 1}" text-anchor="middle" dominant-baseline="central" font-size="17" font-weight="700" fill="#17202a">${value}</text>`;
        layer.appendChild(group);
    });
}

function drawMoves(step) {
    moveLayer.replaceChildren();
    if (step.right.length) {
        const path = document.createElementNS(ns, "path");
        path.setAttribute("d", "M 252 178 L 305 178");
        path.setAttribute("fill", "none");
        path.setAttribute("stroke", "#2471a3");
        path.setAttribute("stroke-width", "4");
        path.setAttribute("marker-end", "url(#heaps-arrow-right)");
        moveLayer.appendChild(path);
    }
    if (step.left.length) {
        const path = document.createElementNS(ns, "path");
        path.setAttribute("d", "M 308 214 L 255 214");
        path.setAttribute("fill", "none");
        path.setAttribute("stroke", "#6c3483");
        path.setAttribute("stroke-width", "4");
        path.setAttribute("marker-end", "url(#heaps-arrow-left)");
        moveLayer.appendChild(path);
    }
}

let index = 0;
let timer = null;

function stopPlaying() {
    if (timer !== null) clearInterval(timer);
    timer = null;
    root.querySelector('[data-action="play"]').textContent = "Play";
}

function render() {
    const step = steps[index];
    svg.setAttribute("aria-label", `Two heaps step ${index + 1} of ${steps.length}: ${step.action}`);

    for (const group of streamLayer.querySelectorAll("g[data-stream-index]")) {
        const i = Number(group.getAttribute("data-stream-index"));
        const circle = group.querySelector("circle");
        if (i === index) {
            circle.setAttribute("fill", "#f8c471");
            circle.setAttribute("stroke", "#b03a2e");
            circle.setAttribute("stroke-width", "5");
        } else if (i < index) {
            circle.setAttribute("fill", "#82e0aa");
            circle.setAttribute("stroke", "#1e8449");
            circle.setAttribute("stroke-width", "3");
        } else {
            circle.setAttribute("fill", "#f2f3f4");
            circle.setAttribute("stroke", "#566573");
            circle.setAttribute("stroke-width", "3");
        }
    }

    drawHeap(lowerLayer, step.lower, "lower", step.insert);
    drawHeap(upperLayer, step.upper, "upper", step.insert);
    drawMoves(step);
    root.querySelector('[data-role="move-label"]').textContent = `Moves: ${step.move}`;
    root.querySelector('[data-role="status"]').textContent = step.action;
    root.querySelector('[data-role="heap-state"]').textContent = `lower [${step.lower.join(", ")}]; upper [${step.upper.join(", ")}]`;
    const valid = step.lower.length === step.upper.length || step.lower.length === step.upper.length + 1;
    root.querySelector('[data-role="balance"]').textContent = `sizes ${step.lower.length}/${step.upper.length}; rule ${valid ? "holds" : "fails"}; median ${step.median}`;

    for (const line of root.querySelectorAll('[data-role="code"] [data-line]')) {
        const active = Number(line.getAttribute("data-line")) === step.line;
        line.style.background = active ? "#f8c471" : "transparent";
        line.style.color = active ? "#17202a" : "inherit";
        line.style.fontWeight = active ? "700" : "400";
        if (active) line.setAttribute("aria-current", "step");
        else line.removeAttribute("aria-current");
    }

    root.querySelector('[data-role="counter"]').textContent = `Step ${index + 1} / ${steps.length}`;
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

render();
display(root);
```

The final state matches sorted order `[1, 3, 5, 7, 8, 15]`. The median is the average of the two middle values, 5 and 7, so the result is 6.

## Why it works
Two rules remain true after every insert, so the roots give the median directly:

1. **Ordering invariant**: every value in `lo` is less than or equal to every value in `hi`. Every new value enters `lo` first, and `lo` sends its largest value to `hi`.
2. **Size invariant**: `|len(lo) - len(hi)| <= 1`, with `lo` never more than one ahead. The rebalance step (move `hi`'s min back to `lo` if `hi` grew too large) enforces this after every insert.

Given both invariants, the two heap roots are exactly the two middle elements of the fully sorted sequence: if sizes are equal, the median is the average of `lo`'s max and `hi`'s min; if `lo` has one extra, `lo`'s max alone is the median. No sorting of the whole stream is ever needed.

## Template
```python
import heapq

lo, hi = [], []  # lo: max-heap (negated), hi: min-heap

def add_num(num):
    heapq.heappush(lo, -num)
    heapq.heappush(hi, -heapq.heappop(lo))
    if len(hi) > len(lo):
        heapq.heappush(lo, -heapq.heappop(hi))

def find_median():
    if len(lo) > len(hi):
        return -lo[0]
    return (-lo[0] + hi[0]) / 2
```

## Complexity
Time: O(log n) per insert because each heap push or pop takes O(log n). A median query takes O(1) because it reads one or two roots.
Space: O(n) to hold every value seen so far across the two heaps.

## Common pitfalls
- Forgetting to negate values for `lo`. Python's `heapq` is a min-heap, so an un-negated `lo` would track the minimum of the lower half instead of the maximum.
- Letting the size gap between `lo` and `hi` grow past 1. The median formula assumes a gap of at most one value.
- Skipping the "route everything through `lo` first" step. A different insertion rule needs its own checks for empty heaps and values equal to the boundary.
- Changing which heap gets the extra value when the total count is odd. This version always gives the extra value to `lo`.

## NeetCode examples
- [[07.FindMedianFromDataStream|FindMedianFromDataStream]]: the standard two-heap running-median problem

## Full guide
[[Job Search/Neetcode/01. Questions/09. Heap or PriorityQueue/0.HeapOrPriorityQueueGuide|Heap / Priority Queue Guide]]
