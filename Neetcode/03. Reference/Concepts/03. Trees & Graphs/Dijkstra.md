---
type: concept
tags: ["concept"]
---

# Dijkstra

**TL;DR:** Single-source shortest path over non-negative weighted edges — greedily settle the cheapest unvisited node next, using a min-heap for O((V+E) log V).

## When to reach for it
- Shortest path from one source in a graph with non-negative weights.
- Minimum cost to reach a destination on a weighted grid or graph.
- "Cheapest flights," "network delay time," or any "minimum total cost" phrasing on a sparse-ish graph.
- Signal: weighted edges + non-negative + "shortest"/"cheapest"/"minimum cost" — if weights can be negative, this is [[Bellman-Ford]] territory instead.

## How it works
A min-heap always pops the cheapest known distance. Use **Previous** and **Next** to follow each heap pop and distance update from A. A red dashed edge was replaced by a cheaper route.

```freeform
const nodes = {
    A: [75, 160],
    B: [270, 70],
    C: [270, 250],
    D: [520, 160],
};

const edges = [
    { from: "A", to: "B", weight: 4, path: "M102 148 L243 82", label: [170, 105] },
    { from: "A", to: "C", weight: 1, path: "M102 172 L243 238", label: [170, 223] },
    { from: "C", to: "B", weight: 2, path: "M270 220 L270 100", label: [290, 164] },
    { from: "B", to: "D", weight: 1, path: "M298 80 L492 150", label: [399, 105] },
    { from: "C", to: "D", weight: 5, path: "M298 240 L492 170", label: [399, 225] },
];

const steps = [
    { current: null, stale: false, finalized: [], selected: [], replaced: [], edge: null, heap: ["(0,A)"], dist: { A: 0, B: "INF", C: "INF", D: "INF" }, line: 1, action: "Set A's distance to 0 and add (0,A) to the heap." },
    { current: "A", stale: false, finalized: ["A"], selected: [], replaced: [], edge: null, heap: [], dist: { A: 0, B: "INF", C: "INF", D: "INF" }, line: 5, action: "Pop (0,A); 0 is current, so finalize A." },
    { current: "A", stale: false, finalized: ["A"], selected: ["A-B"], replaced: [], edge: "A-B", heap: ["(4,B)"], dist: { A: 0, B: 4, C: "INF", D: "INF" }, line: 7, action: "Relax A -> B; update B from INF to 4." },
    { current: "A", stale: false, finalized: ["A"], selected: ["A-B", "A-C"], replaced: [], edge: "A-C", heap: ["(1,C)", "(4,B)"], dist: { A: 0, B: 4, C: 1, D: "INF" }, line: 7, action: "Relax A -> C; update C from INF to 1." },
    { current: "C", stale: false, finalized: ["A", "C"], selected: ["A-B", "A-C"], replaced: [], edge: null, heap: ["(4,B)"], dist: { A: 0, B: 4, C: 1, D: "INF" }, line: 5, action: "Pop (1,C); 1 is current, so finalize C." },
    { current: "C", stale: false, finalized: ["A", "C"], selected: ["A-C", "C-B"], replaced: ["A-B"], edge: "C-B", heap: ["(3,B)", "(4,B) stale"], dist: { A: 0, B: 3, C: 1, D: "INF" }, line: 7, action: "Relax C -> B; update B from 4 to 3." },
    { current: "C", stale: false, finalized: ["A", "C"], selected: ["A-C", "C-B", "C-D"], replaced: ["A-B"], edge: "C-D", heap: ["(3,B)", "(4,B) stale", "(6,D)"], dist: { A: 0, B: 3, C: 1, D: 6 }, line: 7, action: "Relax C -> D; update D from INF to 6." },
    { current: "B", stale: false, finalized: ["A", "C", "B"], selected: ["A-C", "C-B", "C-D"], replaced: ["A-B"], edge: null, heap: ["(4,B) stale", "(6,D)"], dist: { A: 0, B: 3, C: 1, D: 6 }, line: 5, action: "Pop (3,B); 3 is current, so finalize B." },
    { current: "B", stale: false, finalized: ["A", "C", "B"], selected: ["A-C", "C-B", "B-D"], replaced: ["A-B", "C-D"], edge: "B-D", heap: ["(4,B) stale", "(4,D)", "(6,D) stale"], dist: { A: 0, B: 3, C: 1, D: 4 }, line: 7, action: "Relax B -> D; update D from 6 to 4." },
    { current: "B", stale: true, finalized: ["A", "C", "B"], selected: ["A-C", "C-B", "B-D"], replaced: ["A-B", "C-D"], edge: null, heap: ["(4,D)", "(6,D) stale"], dist: { A: 0, B: 3, C: 1, D: 4 }, line: 4, action: "Pop stale (4,B); B's best distance is 3, so skip it." },
    { current: "D", stale: false, finalized: ["A", "C", "B", "D"], selected: ["A-C", "C-B", "B-D"], replaced: ["A-B", "C-D"], edge: null, heap: ["(6,D) stale"], dist: { A: 0, B: 3, C: 1, D: 4 }, line: 5, action: "Pop (4,D); 4 is current, so finalize D." },
    { current: "D", stale: true, finalized: ["A", "C", "B", "D"], selected: ["A-C", "C-B", "B-D"], replaced: ["A-B", "C-D"], edge: null, heap: [], dist: { A: 0, B: 3, C: 1, D: 4 }, line: 4, action: "Pop stale (6,D); D's best distance is 4, so skip it." },
];

const root = document.createElement("section");
root.style.cssText = "font:14px system-ui; max-width:760px; border:1px solid var(--background-modifier-border); border-radius:10px; padding:12px; background:var(--background-primary); color:var(--text-normal);";
root.innerHTML = `
    <style>
        .dij-controls { display:flex; align-items:center; gap:8px; flex-wrap:wrap; margin-top:10px; }
        .dij-controls button, .dij-controls select { min-height:44px; font-size:14px; touch-action:manipulation; }
        .dij-controls button { padding:6px 12px; }
        .dij-graph { width:100%; height:auto; display:block; }
        .dij-state-grid { display:grid; grid-template-columns:1fr 1fr; gap:8px; margin-top:8px; }
        .dij-code { overflow-x:auto; font-size:13px; }
        @media (max-width:520px) {
            .dij-controls { gap:6px; }
            .dij-controls button { flex:1 1 72px; min-height:44px; }
            .dij-speed { flex:1 1 100%; display:flex; align-items:center; gap:8px; }
            .dij-speed select { flex:1; min-height:44px; }
            .dij-counter { width:100%; margin-left:0 !important; text-align:center; }
            .dij-state-grid { grid-template-columns:1fr; }
            .dij-legend { gap:8px !important; font-size:11px !important; }
            .dij-code { font-size:12px; }
        }
    </style>
    <svg class="dij-graph" viewBox="0 0 610 320" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Interactive Dijkstra graph">
        <defs>
            <marker id="dij-arrow-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7f8c8d"></path></marker>
            <marker id="dij-arrow-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#2471a3"></path></marker>
            <marker id="dij-arrow-orange" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#d35400"></path></marker>
            <marker id="dij-arrow-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#c0392b"></path></marker>
        </defs>
        <g data-layer="edges"></g>
        <g data-layer="labels"></g>
        <g data-layer="nodes"></g>
    </svg>
    <div class="dij-legend" style="display:flex; gap:14px; flex-wrap:wrap; margin:0 0 8px; font-size:12px;">
        <span>Orange = current</span><span>Blue = current shortest-path edge</span><span>Green = finalized</span><span>Red dashed = replaced / stale</span><span>Gray = unchecked</span>
    </div>
    <div data-role="status" aria-live="polite" style="min-height:22px; padding:9px 10px; border-radius:8px; background:var(--background-secondary);"></div>
    <div class="dij-state-grid">
        <div style="padding:8px; border:1px solid var(--background-modifier-border); border-radius:8px;"><strong>Heap (sorted priority order)</strong><div data-role="heap" style="margin-top:4px; font-family:var(--font-monospace);"></div></div>
        <div style="padding:8px; border:1px solid var(--background-modifier-border); border-radius:8px;"><strong>Distances</strong><div data-role="dist" style="margin-top:4px; font-family:var(--font-monospace);"></div></div>
    </div>
    <div class="dij-code" data-role="code" style="margin-top:8px; padding:8px 10px; border-radius:8px; background:var(--background-secondary); font-family:var(--font-monospace); line-height:1.55;">
        <div data-line="1">1&nbsp; dist[src] = 0; push (0, src)</div>
        <div data-line="2">2&nbsp; while heap is not empty</div>
        <div data-line="3">3&nbsp;&nbsp;&nbsp; cost, u = pop minimum</div>
        <div data-line="4">4&nbsp;&nbsp;&nbsp; if cost &gt; dist[u]: skip stale entry</div>
        <div data-line="5">5&nbsp;&nbsp;&nbsp; finalize u</div>
        <div data-line="6">6&nbsp;&nbsp;&nbsp; inspect each edge u -&gt; v</div>
        <div data-line="7">7&nbsp;&nbsp;&nbsp; if cheaper: update dist[v]; push</div>
    </div>
    <div class="dij-controls">
        <button data-action="previous" aria-label="Previous step">Previous</button>
        <button data-action="play">Play</button>
        <button data-action="next" aria-label="Next step">Next</button>
        <button data-action="reset">Reset</button>
        <label class="dij-speed" style="margin-left:8px;">Speed <select data-action="speed"><option value="1200">Slow</option><option value="700" selected>Normal</option><option value="350">Fast</option></select></label>
        <strong class="dij-counter" data-role="counter" style="margin-left:auto;"></strong>
    </div>
`;

const edgeLayer = root.querySelector('[data-layer="edges"]');
const labelLayer = root.querySelector('[data-layer="labels"]');
const nodeLayer = root.querySelector('[data-layer="nodes"]');

for (const edge of edges) {
    const key = `${edge.from}-${edge.to}`;
    const path = document.createElementNS("http://www.w3.org/2000/svg", "path");
    path.setAttribute("d", edge.path); path.setAttribute("data-edge", key); path.setAttribute("fill", "none"); path.setAttribute("stroke", "#7f8c8d"); path.setAttribute("stroke-width", "2.5"); path.setAttribute("marker-end", "url(#dij-arrow-gray)");
    edgeLayer.appendChild(path);
    const text = document.createElementNS("http://www.w3.org/2000/svg", "text");
    text.setAttribute("x", edge.label[0]); text.setAttribute("y", edge.label[1]); text.setAttribute("text-anchor", "middle"); text.setAttribute("font-size", "14"); text.setAttribute("font-weight", "700"); text.setAttribute("fill", "currentColor"); text.textContent = String(edge.weight);
    labelLayer.appendChild(text);
}

for (const [name, [x, y]] of Object.entries(nodes)) {
    const group = document.createElementNS("http://www.w3.org/2000/svg", "g");
    group.setAttribute("data-node", name); group.setAttribute("transform", `translate(${x} ${y})`);
    group.innerHTML = `<circle r="30" fill="#d5d8dc" stroke="#566573" stroke-width="3"></circle><text text-anchor="middle" y="-3" dominant-baseline="central" font-size="18" font-weight="700" fill="#17202a">${name}</text><text data-distance text-anchor="middle" y="17" font-size="12" font-weight="600" fill="#17202a"></text>`;
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
    root.querySelector("svg").setAttribute("aria-label", `Dijkstra step ${index + 1} of ${steps.length}: ${step.action}`);
    const finalized = new Set(step.finalized);
    const selected = new Set(step.selected);
    const replaced = new Set(step.replaced);

    for (const group of nodeLayer.querySelectorAll("g[data-node]")) {
        const name = group.getAttribute("data-node");
        const circle = group.querySelector("circle");
        const finite = step.dist[name] !== "INF";
        circle.setAttribute("stroke-dasharray", "");
        if (name === step.current && step.stale) {
            circle.setAttribute("fill", "#f5b7b1"); circle.setAttribute("stroke", "#c0392b"); circle.setAttribute("stroke-width", "5"); circle.setAttribute("stroke-dasharray", "7 4");
        } else if (name === step.current) {
            circle.setAttribute("fill", "#f8c471"); circle.setAttribute("stroke", "#b03a2e"); circle.setAttribute("stroke-width", "5");
        } else if (finalized.has(name)) {
            circle.setAttribute("fill", "#82e0aa"); circle.setAttribute("stroke", "#1e8449"); circle.setAttribute("stroke-width", "3");
        } else if (finite) {
            circle.setAttribute("fill", "#85c1e9"); circle.setAttribute("stroke", "#1f618d"); circle.setAttribute("stroke-width", "3");
        } else {
            circle.setAttribute("fill", "#d5d8dc"); circle.setAttribute("stroke", "#566573"); circle.setAttribute("stroke-width", "3");
        }
        group.querySelector("[data-distance]").textContent = `dist ${step.dist[name]}`;
    }

    for (const path of edgeLayer.querySelectorAll("path[data-edge]")) {
        const key = path.getAttribute("data-edge");
        let color = "#7f8c8d", width = "2.5", marker = "dij-arrow-gray", dash = "";
        if (selected.has(key)) { color = "#2471a3"; width = "3.5"; marker = "dij-arrow-blue"; }
        if (replaced.has(key)) { color = "#c0392b"; marker = "dij-arrow-red"; dash = "7 5"; }
        if (key === step.edge) { color = "#d35400"; width = "5"; marker = "dij-arrow-orange"; dash = ""; }
        path.setAttribute("stroke", color); path.setAttribute("stroke-width", width); path.setAttribute("stroke-dasharray", dash); path.setAttribute("marker-end", `url(#${marker})`);
    }

    root.querySelector('[data-role="status"]').textContent = step.action;
    root.querySelector('[data-role="heap"]').textContent = step.heap.length ? `[${step.heap.join(", ")}]` : "empty";
    root.querySelector('[data-role="dist"]').textContent = Object.entries(step.dist).map(([node, value]) => `${node}:${value}`).join("  ");
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

Final distances: `A=0, C=1, B=3, D=4`.

## Why it works
The heap always pops the smallest tentative distance among unsettled nodes. Any *other* path to that node would have to go through some other unsettled node — but every unsettled node already has a tentative distance ≥ the one just popped (that's why it wasn't popped first), and edge weights are non-negative, so continuing along any other path can only add more cost, never less. That means the popped distance can't be beaten later — it's final. This argument breaks the instant a negative edge is allowed, because a longer-looking path could later subtract enough to become cheaper, which is exactly what [[Bellman-Ford]] is built to handle. The "stale entry" check (`cost > dist[u]`) is lazy deletion: rather than search the heap to remove outdated entries when a shorter path is found, just leave them there and ignore any popped entry that's worse than the best distance already recorded — cheaper than maintaining a decrease-key heap.

## Template
```python
import heapq
from collections import defaultdict

def dijkstra(n, edges, src):
    adj = defaultdict(list)
    for u, v, w in edges:
        adj[u].append((w, v))

    dist = [float('inf')] * n
    dist[src] = 0
    heap = [(0, src)]   # (cost, node)

    while heap:
        cost, u = heapq.heappop(heap)
        if cost > dist[u]:
            continue          # stale entry
        for w, v in adj[u]:
            new_cost = cost + w
            if new_cost < dist[v]:
                dist[v] = new_cost
                heapq.heappush(heap, (new_cost, v))

    return dist
```

## Complexity
Time: O((V + E) log V) — each edge can trigger one heap push, and each push/pop is O(log V); stale entries mean the heap can hold up to O(E) items, not just O(V) | Space: O(V + E) for the adjacency list, distances, and heap

## Common pitfalls
- Using Dijkstra with negative edge weights — the greedy-safety argument no longer holds; use [[Bellman-Ford]] instead.
- Forgetting the stale-entry check (`if cost > dist[u]: continue`) — without it, already-settled nodes get needlessly re-processed with outdated costs.
- Re-processing a node that's already settled as if it might still improve — once popped with its true shortest distance, nothing more can lower it.
- Assuming the heap only ever holds V entries — with lazy deletion it can grow to O(E), which matters for memory-sensitive problems.

## NeetCode examples
- [[03.NetworkDelayTime|NetworkDelayTime]] — single-source shortest path to all nodes
- [[06.CheapestFlightsWithinKStops|CheapestFlightsWithinKStops]] — modified Dijkstra with hop constraint (or Bellman-Ford)
- [[04.SwimInRisingWater|SwimInRisingWater]] — Dijkstra on a grid where weight = cell value

## Full guide
[[Job Search/Neetcode/01. Questions/12. Advanced Graphs/0.AdvancedGraphsGuide|Advanced Graphs Guide]]
