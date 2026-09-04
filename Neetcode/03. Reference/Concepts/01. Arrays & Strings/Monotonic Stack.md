---
type: concept
tags: ["concept"]
---

# Monotonic Stack

**TL;DR:** Keep a stack in strictly increasing or decreasing order so each new element resolves the "next greater/smaller" query for everything still waiting on the stack, in amortized O(1) per element.

## When to reach for it
- Need the next (or previous) greater or smaller element for every index.
- Computing largest rectangle in a histogram, trapped water, or "span" problems (how far back/forward before something bigger appears).
- Simplifying expressions or evaluating with value-ordered constraints (e.g., car fleets catching up to each other).

## How it works
Walk the array once, keeping indices on a stack such that the values they point to are monotone (say, decreasing, for a "next greater element" query). When the current element beats the stack's top, that's proof the top has found its answer — pop it and resolve it. Trace **next greater element** on `nums = [2, 1, 5, 3]`:

### Interactive walkthrough

This walkthrough separates each comparison, push, and pop so the multi-pop at `5` is easy to follow.

```freeform
const nums = [2, 1, 5, 3];
const steps = [
    { cursor: null, top: null, stack: [], result: [null, null, null, null], line: 1, action: "Start with an empty waiting stack and no resolved answers." },
    { cursor: 0, top: null, stack: [0], result: [null, null, null, null], line: 5, action: "Push 2 because the waiting stack is empty." },
    { cursor: 1, top: 0, stack: [0], result: [null, null, null, null], line: 3, action: "Compare 1 with 2 and keep 2 waiting because 1 is smaller." },
    { cursor: 1, top: 0, stack: [0, 1], result: [null, null, null, null], line: 5, action: "Push 1 because it still needs a greater value." },
    { cursor: 2, top: 1, stack: [0, 1], result: [null, null, null, null], line: 3, action: "Compare 5 with 1 and prepare to resolve index 1." },
    { cursor: 2, top: 1, stack: [0], result: [null, 5, null, null], line: 4, action: "Pop index 1 and set its next greater value to 5." },
    { cursor: 2, top: 0, stack: [0], result: [null, 5, null, null], line: 3, action: "Compare 5 with the new top 2 and continue the same while loop." },
    { cursor: 2, top: 0, stack: [], result: [5, 5, null, null], line: 4, action: "Pop index 0 and set its next greater value to 5." },
    { cursor: 2, top: null, stack: [2], result: [5, 5, null, null], line: 5, action: "Push 5 after it resolves every smaller waiting value." },
    { cursor: 3, top: 2, stack: [2], result: [5, 5, null, null], line: 3, action: "Compare 3 with 5 and keep 5 waiting because 3 is smaller." },
    { cursor: 3, top: 2, stack: [2, 3], result: [5, 5, null, null], line: 5, action: "Push 3 because it has not found a greater value yet." },
    { cursor: null, top: null, stack: [2, 3], result: [5, 5, -1, -1], line: 6, action: "Finish with indices 2 and 3 still waiting, so their answers stay -1." },
];

const root = document.createElement("section");
root.style.cssText = "font:14px system-ui; max-width:700px; border:1px solid var(--background-modifier-border); border-radius:10px; padding:12px; background:var(--background-primary); color:var(--text-normal);";
root.innerHTML = `
    <style>
        .ms-svg { width:100%; height:auto; display:block; }
        .ms-states { display:grid; grid-template-columns:1fr 1fr; gap:8px; margin-top:8px; }
        .ms-code { overflow-x:auto; font-size:13px; }
        .ms-controls { display:flex; align-items:center; gap:8px; flex-wrap:wrap; margin-top:10px; }
        .ms-controls button, .ms-controls select { min-height:44px; font-size:14px; touch-action:manipulation; }
        .ms-controls button { padding:6px 12px; }
        @media (max-width:520px) {
            .ms-states { grid-template-columns:1fr; }
            .ms-code { font-size:12px; }
            .ms-controls { gap:6px; }
            .ms-controls button { flex:1 1 72px; }
            .ms-speed { flex:1 1 100%; display:flex; align-items:center; gap:8px; }
            .ms-speed select { flex:1; }
            .ms-counter { width:100%; margin-left:0 !important; text-align:center; }
        }
    </style>
    <svg class="ms-svg" viewBox="0 0 480 255" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Monotonic stack before the first step">
        <text x="240" y="24" text-anchor="middle" font-size="17" font-weight="700" fill="#17202a">nums = [2, 1, 5, 3]</text>
        <text x="22" y="88" font-size="14" font-weight="700" fill="#34495e">value</text>
        <g data-layer="values"></g>
        <text x="22" y="181" font-size="14" font-weight="700" fill="#34495e">answer</text>
        <g data-layer="answers"></g>
        <text x="240" y="238" text-anchor="middle" font-size="14" font-weight="600" fill="#34495e" data-role="svg-note"></text>
    </svg>
    <div style="display:flex; gap:12px; flex-wrap:wrap; margin:0 0 8px; font-size:12px;">
        <span>Orange = current value</span><span>Blue = waiting</span><span>Green = resolved</span>
    </div>
    <div data-role="status" aria-live="polite" style="min-height:22px; padding:9px 10px; border-radius:8px; background:var(--background-secondary);"></div>
    <div class="ms-states">
        <div style="padding:8px; border:1px solid var(--background-modifier-border); border-radius:8px;"><strong>Waiting stack</strong><div data-role="stack" style="margin-top:4px;"></div></div>
        <div style="padding:8px; border:1px solid var(--background-modifier-border); border-radius:8px;"><strong>Resolved answers</strong><div data-role="result" style="margin-top:4px;"></div></div>
    </div>
    <pre class="ms-code" data-role="code" style="padding:9px; border-radius:8px; background:var(--background-secondary);"><code><span data-line="1">result = [-1] * len(nums)</span>\n<span data-line="2">for i, value in enumerate(nums):</span>\n<span data-line="3">    while stack and value &gt; nums[stack[-1]]:</span>\n<span data-line="4">        result[stack.pop()] = value</span>\n<span data-line="5">    stack.append(i)</span>\n<span data-line="6">return result</span></code></pre>
    <div class="ms-controls">
        <button type="button" data-action="previous">Previous</button>
        <button type="button" data-action="play">Play</button>
        <button type="button" data-action="next">Next</button>
        <button type="button" data-action="reset">Reset</button>
        <label class="ms-speed">Speed <select data-action="speed"><option value="1600">Slow</option><option value="950" selected>Normal</option><option value="500">Fast</option></select></label>
        <span class="ms-counter" data-role="counter" style="margin-left:auto;"></span>
    </div>
`;

const svg = root.querySelector("svg");
const ns = "http://www.w3.org/2000/svg";
let index = 0;
let timer = null;

function svgElement(name, attrs, text) {
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

function render() {
    const step = steps[index];
    const values = root.querySelector('[data-layer="values"]');
    const answers = root.querySelector('[data-layer="answers"]');
    values.replaceChildren();
    answers.replaceChildren();

    nums.forEach((value, i) => {
        const x = 78 + i * 96;
        const waiting = step.stack.includes(i);
        const current = step.cursor === i;
        const resolved = step.result[i] !== null;
        const fill = current ? "#f8c471" : waiting ? "#85c1e9" : resolved ? "#abebc6" : "#f4f6f7";
        const stroke = step.top === i ? "#d35400" : "#566573";
        const width = step.top === i ? 4 : 2;
        values.appendChild(svgElement("rect", { x, y: 55, width: 68, height: 54, rx: 8, fill, stroke, "stroke-width": width }));
        values.appendChild(svgElement("text", { x: x + 34, y: 88, "text-anchor": "middle", "font-size": 20, "font-weight": 700, fill: "#17202a" }, value));
        values.appendChild(svgElement("text", { x: x + 34, y: 128, "text-anchor": "middle", "font-size": 13, fill: "#34495e" }, `i=${i}`));

        const answerFill = resolved ? "#abebc6" : "#f4f6f7";
        answers.appendChild(svgElement("rect", { x, y: 150, width: 68, height: 46, rx: 8, fill: answerFill, stroke: "#566573", "stroke-width": 2 }));
        answers.appendChild(svgElement("text", { x: x + 34, y: 180, "text-anchor": "middle", "font-size": 18, "font-weight": 700, fill: "#17202a" }, resolved ? step.result[i] : "?"));
    });

    const cursorText = step.cursor === null ? "none" : `${nums[step.cursor]} at index ${step.cursor}`;
    const topText = step.top === null ? "none" : `${nums[step.top]} at index ${step.top}`;
    root.querySelector('[data-role="svg-note"]').textContent = `Current: ${cursorText} | stack top: ${topText}`;
    svg.setAttribute("aria-label", `Monotonic stack step ${index + 1} of ${steps.length}. ${step.action}`);
    root.querySelector('[data-role="status"]').textContent = step.action;
    root.querySelector('[data-role="stack"]').textContent = step.stack.length ? `[${step.stack.map(i => `${nums[i]} (i=${i})`).join(", ")}]` : "empty";
    root.querySelector('[data-role="result"]').textContent = `[${step.result.map(value => value === null ? "?" : value).join(", ")}]`;

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

Remaining indices `[2, 3]` never find a greater element, so `result[2] = result[3] = -1`. Final: `result = [5, 5, -1, -1]`.

## Why it works
Each index is pushed exactly once and popped at most once, so total stack operations across the whole run are bounded by 2n — that's the source of the amortized O(1)-per-element cost, even though there's a `while` loop inside the `for` loop. The invariant: at any point, the stack (bottom to top) holds indices in increasing order of position whose "next greater" answer is *still unknown*, with values monotonically decreasing bottom-to-top. Popping is safe because when `nums[i]` beats `nums[stack[-1]]`, `nums[i]` is provably the *first* element to the right that's bigger — nothing between the popped index and `i` could be the true answer instead, since anything bigger would already have triggered its own pop. That's why you never need to "undo" a pop or reconsider a resolved index.

## Template
```python
# Next Greater Element (monotonic decreasing stack)
n = len(nums)
result = [-1] * n
stack = []  # stores indices

for i in range(n):
    # pop while current element is greater than stack top
    while stack and nums[i] > nums[stack[-1]]:
        idx = stack.pop()
        result[idx] = nums[i]
    stack.append(i)
# anything remaining in stack has no greater element (-1)

# Monotonic increasing stack (next smaller element):
# flip the comparison: while stack and nums[i] < nums[stack[-1]]
```

## Complexity
Time: O(n) — each index is pushed once and popped at most once, so the while loop's total work across all iterations is bounded by the number of pushes. Space: O(n) for the stack in the worst case (e.g., a strictly increasing input never pops anything until the end).

## Common pitfalls
- Storing values instead of indices — most problems need both the value (for comparison) and the position (for the result array or a distance calculation).
- Getting the monotone direction backwards (increasing vs. decreasing) for what the problem actually asks (next *greater* vs. next *smaller*).
- Forgetting to handle wrap-around for circular array problems (run two passes over the array, or conceptually double its length).
- Popping with `>=` instead of `>` (or vice versa) — changes whether equal elements resolve each other, which can silently break span-counting problems.

## NeetCode examples
- [[05.DailyTemperatures|DailyTemperatures]] — next greater temperature via decreasing stack
- [[07.LargestRectangleInHistogram|LargestRectangleInHistogram]] — increasing stack to find left/right boundaries
- [[06.CarFleet|CarFleet]] — decreasing stack over sorted positions

## Full guide
[[Job Search/Neetcode/01. Questions/04. Stack/0.StackGuide|Stack Guide]]
