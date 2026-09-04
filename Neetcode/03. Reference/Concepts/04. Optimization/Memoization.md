---
type: concept
tags: [concept, dsa, pattern]
---

# Memoization

**TL;DR:** Cache each subproblem's result the first time it's computed, keyed on its input state, so every repeat call becomes an O(1) lookup instead of a recomputation.

## When to reach for it
- A recursive solution is correct but slow, and adding `print` statements to the recursion shows the *same arguments* being passed in repeatedly.
- The recursion has a clean, hashable "state" (an index, a pair of indices, a bitmask) that fully determines the answer for that subproblem.
- You want the top-down form of [[Dynamic Programming]], where the recurrence is a recursive function instead of a table filled in a planned order.

## How it works
Wrap (or manually guard) a recursive function with a cache keyed on its arguments: before doing any work, check whether this exact state has been seen; if so, return the cached answer; if not, compute it, store it, then return it.

### Interactive walkthrough

This walkthrough computes `fib(4)`. It shows the active calls, each cache write, and the repeated calls whose work is skipped.

```freeform
const nodes = [
    { id: "n4", label: "fib(4)", x: 280, y: 52 },
    { id: "n3", label: "fib(3)", x: 165, y: 132 },
    { id: "n2hit", label: "fib(2)", x: 395, y: 132 },
    { id: "n2", label: "fib(2)", x: 90, y: 222 },
    { id: "n1hit", label: "fib(1)", x: 240, y: 222 },
    { id: "n1", label: "fib(1)", x: 45, y: 322 },
    { id: "n0", label: "fib(0)", x: 135, y: 322 },
];

const edges = [
    { id: "n4-n3", from: "n4", to: "n3" },
    { id: "n4-n2hit", from: "n4", to: "n2hit" },
    { id: "n3-n2", from: "n3", to: "n2" },
    { id: "n3-n1hit", from: "n3", to: "n1hit" },
    { id: "n2-n1", from: "n2", to: "n1" },
    { id: "n2-n0", from: "n2", to: "n0" },
];

const positions = Object.fromEntries(nodes.map((node) => [node.id, node]));
const steps = [
    { current: "n4", edge: null, stack: [4], cache: {}, done: [], hits: [], values: {}, line: 1, action: "Start fib(4) with an empty cache." },
    { current: "n3", edge: "n4-n3", stack: [4, 3], cache: {}, done: [], hits: [], values: {}, line: 4, action: "Call fib(3) because fib(4) needs its first smaller result." },
    { current: "n2", edge: "n3-n2", stack: [4, 3, 2], cache: {}, done: [], hits: [], values: {}, line: 4, action: "Call fib(2) because fib(3) needs its first smaller result." },
    { current: "n1", edge: "n2-n1", stack: [4, 3, 2, 1], cache: { 1: 1 }, done: ["n1"], hits: [], values: { n1: 1 }, line: 3, action: "Cache fib(1) = 1 at the base case." },
    { current: "n0", edge: "n2-n0", stack: [4, 3, 2, 0], cache: { 0: 0, 1: 1 }, done: ["n1", "n0"], hits: [], values: { n1: 1, n0: 0 }, line: 3, action: "Cache fib(0) = 0 at the base case." },
    { current: "n2", edge: null, stack: [4, 3, 2], cache: { 0: 0, 1: 1, 2: 1 }, done: ["n1", "n0", "n2"], hits: [], values: { n1: 1, n0: 0, n2: 1 }, line: 6, action: "Add 1 + 0 and cache fib(2) = 1." },
    { current: "n1hit", edge: "n3-n1hit", stack: [4, 3, 1], cache: { 0: 0, 1: 1, 2: 1 }, done: ["n1", "n0", "n2"], hits: ["n1hit"], values: { n1: 1, n0: 0, n2: 1, n1hit: 1 }, line: 2, action: "Read fib(1) = 1 from the cache and skip its repeated work." },
    { current: "n3", edge: null, stack: [4, 3], cache: { 0: 0, 1: 1, 2: 1, 3: 2 }, done: ["n1", "n0", "n2", "n3"], hits: ["n1hit"], values: { n1: 1, n0: 0, n2: 1, n1hit: 1, n3: 2 }, line: 6, action: "Add 1 + 1 and cache fib(3) = 2." },
    { current: "n2hit", edge: "n4-n2hit", stack: [4, 2], cache: { 0: 0, 1: 1, 2: 1, 3: 2 }, done: ["n1", "n0", "n2", "n3"], hits: ["n1hit", "n2hit"], values: { n1: 1, n0: 0, n2: 1, n1hit: 1, n3: 2, n2hit: 1 }, line: 2, action: "Read fib(2) = 1 from the cache and skip its repeated work." },
    { current: "n4", edge: null, stack: [4], cache: { 0: 0, 1: 1, 2: 1, 3: 2, 4: 3 }, done: ["n1", "n0", "n2", "n3", "n4"], hits: ["n1hit", "n2hit"], values: { n1: 1, n0: 0, n2: 1, n1hit: 1, n3: 2, n2hit: 1, n4: 3 }, line: 6, action: "Add 2 + 1 and cache fib(4) = 3." },
    { current: null, edge: null, stack: [], cache: { 0: 0, 1: 1, 2: 1, 3: 2, 4: 3 }, done: ["n1", "n0", "n2", "n3", "n4"], hits: ["n1hit", "n2hit"], values: { n1: 1, n0: 0, n2: 1, n1hit: 1, n3: 2, n2hit: 1, n4: 3 }, line: 6, action: "Return fib(4) = 3 after computing each distinct input once." },
];

const root = document.createElement("section");
root.style.cssText = "font:14px system-ui; max-width:720px; border:1px solid var(--background-modifier-border); border-radius:10px; padding:12px; background:var(--background-primary); color:var(--text-normal);";
root.innerHTML = `
    <style>
        .memo-graph { width:100%; height:auto; display:block; }
        .memo-state-grid { display:grid; grid-template-columns:1fr 1fr; gap:8px; margin-top:8px; }
        .memo-code { overflow-x:auto; font-size:13px; }
        .memo-controls { display:flex; align-items:center; gap:8px; flex-wrap:wrap; margin-top:10px; }
        .memo-controls button, .memo-controls select { min-height:44px; font-size:14px; touch-action:manipulation; }
        .memo-controls button { padding:6px 12px; }
        @media (max-width:520px) {
            .memo-state-grid { grid-template-columns:1fr; }
            .memo-code { font-size:12px; }
            .memo-controls { gap:6px; }
            .memo-controls button { flex:1 1 72px; }
            .memo-speed { flex:1 1 100%; display:flex; align-items:center; gap:8px; }
            .memo-speed select { flex:1; }
            .memo-counter { width:100%; margin-left:0 !important; text-align:center; }
        }
    </style>
    <svg class="memo-graph" viewBox="0 0 560 380" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Interactive memoization stepper">
        <defs>
            <marker id="memo-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="#566573"></path></marker>
            <marker id="memo-arrow-active" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="#d35400"></path></marker>
        </defs>
        <text x="280" y="24" text-anchor="middle" font-size="18" font-weight="700" fill="#17202a">fib(4) call tree</text>
        <g data-layer="edges"></g>
        <g data-layer="nodes"></g>
    </svg>
    <div style="display:flex; gap:12px; flex-wrap:wrap; margin:0 0 8px; font-size:12px;">
        <span>Orange = active call</span><span>Green = cached result</span><span>Purple dashed = cache hit</span>
    </div>
    <div data-role="status" aria-live="polite" style="min-height:22px; padding:9px 10px; border-radius:8px; background:var(--background-secondary);"></div>
    <div class="memo-state-grid">
        <div style="padding:8px; border:1px solid var(--background-modifier-border); border-radius:8px;"><strong>Call stack</strong><div data-role="stack" style="margin-top:4px; font-family:var(--font-monospace);"></div></div>
        <div style="padding:8px; border:1px solid var(--background-modifier-border); border-radius:8px;"><strong>Cache</strong><div data-role="cache" style="margin-top:4px; font-family:var(--font-monospace);"></div></div>
    </div>
    <div class="memo-code" data-role="code" style="margin-top:8px; padding:8px 10px; border-radius:8px; background:var(--background-secondary); font-family:var(--font-monospace); line-height:1.55;">
        <div data-line="1">1&nbsp; fib(n)</div>
        <div data-line="2">2&nbsp;&nbsp;&nbsp; if n in cache: return cache[n]</div>
        <div data-line="3">3&nbsp;&nbsp;&nbsp; if n &lt;= 1: cache[n] = n; return n</div>
        <div data-line="4">4&nbsp;&nbsp;&nbsp; left = fib(n - 1)</div>
        <div data-line="5">5&nbsp;&nbsp;&nbsp; right = fib(n - 2)</div>
        <div data-line="6">6&nbsp;&nbsp;&nbsp; cache[n] = left + right; return cache[n]</div>
    </div>
    <div class="memo-controls">
        <button data-action="previous" aria-label="Previous step">Previous</button>
        <button data-action="play">Play</button>
        <button data-action="next" aria-label="Next step">Next</button>
        <button data-action="reset">Reset</button>
        <label class="memo-speed" style="margin-left:8px;">Speed <select data-action="speed"><option value="1200">Slow</option><option value="700" selected>Normal</option><option value="350">Fast</option></select></label>
        <strong class="memo-counter" data-role="counter" style="margin-left:auto;"></strong>
    </div>
`;

const ns = "http://www.w3.org/2000/svg";
const svg = root.querySelector("svg");
const edgeLayer = root.querySelector('[data-layer="edges"]');
const nodeLayer = root.querySelector('[data-layer="nodes"]');

for (const edge of edges) {
    const from = positions[edge.from];
    const to = positions[edge.to];
    const dx = to.x - from.x;
    const dy = to.y - from.y;
    const length = Math.hypot(dx, dy);
    const startX = from.x + (dx / length) * 34;
    const startY = from.y + (dy / length) * 25;
    const endX = to.x - (dx / length) * 37;
    const endY = to.y - (dy / length) * 27;
    const path = document.createElementNS(ns, "path");
    path.setAttribute("d", `M ${startX} ${startY} L ${endX} ${endY}`);
    path.setAttribute("data-edge", edge.id);
    path.setAttribute("fill", "none");
    path.setAttribute("stroke", "#7f8c8d");
    path.setAttribute("stroke-width", "3");
    path.setAttribute("marker-end", "url(#memo-arrow)");
    edgeLayer.appendChild(path);
}

for (const node of nodes) {
    const group = document.createElementNS(ns, "g");
    group.setAttribute("data-node", node.id);
    group.setAttribute("transform", `translate(${node.x} ${node.y})`);
    group.innerHTML = `<rect x="-39" y="-24" width="78" height="48" rx="10" fill="#f2f3f4" stroke="#566573" stroke-width="3"></rect><text text-anchor="middle" y="-4" dominant-baseline="central" font-size="16" font-weight="700" fill="#17202a">${node.label}</text><text data-result text-anchor="middle" y="15" font-size="13" font-weight="600" fill="#17202a"></text>`;
    nodeLayer.appendChild(group);
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
    const done = new Set(step.done);
    const hits = new Set(step.hits);
    svg.setAttribute("aria-label", `Memoization step ${index + 1} of ${steps.length}: ${step.action}`);

    for (const group of nodeLayer.querySelectorAll("g[data-node]")) {
        const id = group.getAttribute("data-node");
        const rect = group.querySelector("rect");
        const result = group.querySelector("[data-result]");
        if (id === step.current) {
            rect.setAttribute("fill", "#f8c471");
            rect.setAttribute("stroke", "#b03a2e");
            rect.setAttribute("stroke-width", "5");
            rect.setAttribute("stroke-dasharray", "");
        } else if (hits.has(id)) {
            rect.setAttribute("fill", "#d7bde2");
            rect.setAttribute("stroke", "#6c3483");
            rect.setAttribute("stroke-width", "4");
            rect.setAttribute("stroke-dasharray", "7 5");
        } else if (done.has(id)) {
            rect.setAttribute("fill", "#82e0aa");
            rect.setAttribute("stroke", "#1e8449");
            rect.setAttribute("stroke-width", "3");
            rect.setAttribute("stroke-dasharray", "");
        } else {
            rect.setAttribute("fill", "#f2f3f4");
            rect.setAttribute("stroke", "#566573");
            rect.setAttribute("stroke-width", "3");
            rect.setAttribute("stroke-dasharray", "");
        }
        result.textContent = Object.hasOwn(step.values, id) ? `= ${step.values[id]}` : "";
    }

    for (const path of edgeLayer.querySelectorAll("path[data-edge]")) {
        const active = path.getAttribute("data-edge") === step.edge;
        path.setAttribute("stroke", active ? "#d35400" : "#7f8c8d");
        path.setAttribute("stroke-width", active ? "5" : "3");
        path.setAttribute("marker-end", active ? "url(#memo-arrow-active)" : "url(#memo-arrow)");
    }

    root.querySelector('[data-role="status"]').textContent = step.action;
    root.querySelector('[data-role="stack"]').textContent = step.stack.length ? step.stack.map((n) => `fib(${n})`).join(" -> ") : "empty";
    const cacheEntries = Object.entries(step.cache).map(([key, value]) => `${key}:${value}`);
    root.querySelector('[data-role="cache"]').textContent = cacheEntries.length ? `{${cacheEntries.join(", ")}}` : "empty";

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

For `fib(4)`, the cache stores each input from `0` through `4`, and the final result is `3`.

## Why it works
A plain recursive call tree for `fib(n)` branches into two children per call, so it visits `O(2^n)` nodes overall, but there are only `n + 1` distinct states (`fib(0)` through `fib(n)`). Every call asks one of those `n + 1` questions. A cache keyed on the argument stops each repeated question after the first, so total work depends on the number of distinct states instead of the number of calls. Bottom-up [[Tabulation|tabulation]] also computes each `dp[i]` once. Memoization discovers needed states through recursion instead of filling every state in advance.

## Template
```python
def fib(n: int) -> int:
    memo = {}

    def dp(i: int) -> int:
        if i in memo:
            return memo[i]
        if i <= 1:
            return i

        memo[i] = dp(i - 1) + dp(i - 2)
        return memo[i]

    return dp(n)


def solve(start_state):
    memo = {}

    def dp(state):
        if state in memo:
            return memo[state]
        if base_case(state):
            return base_value(state)

        memo[state] = combine(dp(next_state) for next_state in transitions(state))
        return memo[state]

    return dp(start_state)
```
## Complexity
Time: O(distinct states x work per state). Each state is computed once, and each later call for that state is an O(1) cache lookup.
Space: O(distinct states) for the cache, plus O(recursion depth) for the call stack. Memoization removes repeated work but does not reduce the deepest call chain.

## Common pitfalls
- Cache keys must be hashable. Convert a list or dictionary state into a tuple, frozenset, string, or smaller hashable summary.
- Keying the cache on more state than necessary causes needless growth. A cache with as many entries as calls provides no speedup.
- Write the result to the cache before returning it. A lookup without a matching write leaves the recursion exponential.
- Create the dictionary inside the outer solution function so separate test cases do not share stale entries.
- Deep recursion can still hit Python's recursion limit for large `n`. Memoization does not flatten the call stack like an iterative table does.

## NeetCode examples
- [[10.WordBreak|WordBreak]]: top-down memo on whether the substring starting at `i` can be split into words
- [[07.DecodeWays|DecodeWays]]: memo on the number of ways to decode from index `i`
- [[12.PartitionEqualSubsetSum|PartitionEqualSubsetSum]]: memo on whether `(index, remaining sum)` is reachable

## Full guide
[[Job Search/Neetcode/01. Questions/13. 1-D Dynamic Programming/0.1DDynamicProgrammingGuide|1-D DP Guide]]
