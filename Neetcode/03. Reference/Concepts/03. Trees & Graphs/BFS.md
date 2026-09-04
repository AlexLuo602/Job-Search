---
type: concept
tags: ["concept"]
---

# BFS

**TL;DR:** Explore all nodes at distance d before any node at distance d+1, using a FIFO queue — this ordering is what guarantees shortest path in an unweighted graph.

## When to reach for it
- "Shortest path" / "fewest steps" / "minimum moves" in an unweighted graph or grid.
- Level-order traversal of a tree (process one depth at a time).
- Spreading/infection problems — rotten oranges, walls and gates, word ladder — especially with multiple simultaneous sources.
- Any "nearest X" question where all edges cost the same.

## How it works
A queue processes nodes in the order they were discovered, so it finishes every node at distance d before processing any node at distance d+1. Use **Previous** and **Next** to trace shortest distance from A. The node labels show each node's BFS level.

```freeform
const nodes = {
    A: [70, 160],
    B: [215, 70],
    C: [215, 250],
    D: [380, 70],
    E: [380, 250],
    F: [545, 160],
};

const edges = [
    { from: "A", to: "B", path: "M95 145 L190 85" },
    { from: "A", to: "C", path: "M95 175 L190 235" },
    { from: "B", to: "D", path: "M244 70 L351 70" },
    { from: "C", to: "E", path: "M244 250 L351 250" },
    { from: "C", to: "F", path: "M243 242 L517 168" },
    { from: "D", to: "B", path: "M359 49 Q298 -2 236 49" },
    { from: "D", to: "F", path: "M405 84 L520 146" },
    { from: "E", to: "D", path: "M380 221 L380 99" },
];

const steps = [
    { current: null, processed: [], discovered: ["A"], edge: null, edgeType: null, tree: [], skipped: [], queue: ["A"], order: ["A"], levels: { A: 0 }, line: 1, action: "Start at A and add it to the queue." },
    { current: "A", processed: [], discovered: ["A"], edge: null, edgeType: null, tree: [], skipped: [], queue: [], order: ["A"], levels: { A: 0 }, line: 3, action: "Remove A from the front of the queue." },
    { current: "A", processed: [], discovered: ["A", "B"], edge: "A-B", edgeType: "discover", tree: ["A-B"], skipped: [], queue: ["B"], order: ["A", "B"], levels: { A: 0, B: 1 }, line: 5, action: "Inspect A -> B; discover B at level 1." },
    { current: "A", processed: [], discovered: ["A", "B", "C"], edge: "A-C", edgeType: "discover", tree: ["A-B", "A-C"], skipped: [], queue: ["B", "C"], order: ["A", "B", "C"], levels: { A: 0, B: 1, C: 1 }, line: 5, action: "Inspect A -> C; discover C at level 1." },
    { current: "B", processed: ["A"], discovered: ["A", "B", "C"], edge: null, edgeType: null, tree: ["A-B", "A-C"], skipped: [], queue: ["C"], order: ["A", "B", "C"], levels: { A: 0, B: 1, C: 1 }, line: 3, action: "Finish A, then remove B from the queue." },
    { current: "B", processed: ["A"], discovered: ["A", "B", "C", "D"], edge: "B-D", edgeType: "discover", tree: ["A-B", "A-C", "B-D"], skipped: [], queue: ["C", "D"], order: ["A", "B", "C", "D"], levels: { A: 0, B: 1, C: 1, D: 2 }, line: 5, action: "Inspect B -> D; discover D at level 2." },
    { current: "C", processed: ["A", "B"], discovered: ["A", "B", "C", "D"], edge: null, edgeType: null, tree: ["A-B", "A-C", "B-D"], skipped: [], queue: ["D"], order: ["A", "B", "C", "D"], levels: { A: 0, B: 1, C: 1, D: 2 }, line: 3, action: "Finish B, then remove C from the queue." },
    { current: "C", processed: ["A", "B"], discovered: ["A", "B", "C", "D", "E"], edge: "C-E", edgeType: "discover", tree: ["A-B", "A-C", "B-D", "C-E"], skipped: [], queue: ["D", "E"], order: ["A", "B", "C", "D", "E"], levels: { A: 0, B: 1, C: 1, D: 2, E: 2 }, line: 5, action: "Inspect C -> E; discover E at level 2." },
    { current: "C", processed: ["A", "B"], discovered: ["A", "B", "C", "D", "E", "F"], edge: "C-F", edgeType: "discover", tree: ["A-B", "A-C", "B-D", "C-E", "C-F"], skipped: [], queue: ["D", "E", "F"], order: ["A", "B", "C", "D", "E", "F"], levels: { A: 0, B: 1, C: 1, D: 2, E: 2, F: 2 }, line: 5, action: "Inspect C -> F; discover F at level 2." },
    { current: "D", processed: ["A", "B", "C"], discovered: ["A", "B", "C", "D", "E", "F"], edge: "D-B", edgeType: "skip", tree: ["A-B", "A-C", "B-D", "C-E", "C-F"], skipped: ["D-B"], queue: ["E", "F"], order: ["A", "B", "C", "D", "E", "F"], levels: { A: 0, B: 1, C: 1, D: 2, E: 2, F: 2 }, line: 4, action: "Remove D; B is already discovered, so skip D -> B." },
    { current: "D", processed: ["A", "B", "C"], discovered: ["A", "B", "C", "D", "E", "F"], edge: "D-F", edgeType: "skip", tree: ["A-B", "A-C", "B-D", "C-E", "C-F"], skipped: ["D-B", "D-F"], queue: ["E", "F"], order: ["A", "B", "C", "D", "E", "F"], levels: { A: 0, B: 1, C: 1, D: 2, E: 2, F: 2 }, line: 4, action: "F is already discovered, so skip D -> F." },
    { current: null, processed: ["A", "B", "C", "D", "E", "F"], discovered: ["A", "B", "C", "D", "E", "F"], edge: null, edgeType: null, tree: ["A-B", "A-C", "B-D", "C-E", "C-F"], skipped: ["D-B", "D-F", "E-D"], queue: [], order: ["A", "B", "C", "D", "E", "F"], levels: { A: 0, B: 1, C: 1, D: 2, E: 2, F: 2 }, line: 2, action: "Process E and F; no new node is added, so BFS is complete." },
];

const root = document.createElement("section");
root.style.cssText = "font:14px system-ui; max-width:760px; border:1px solid var(--background-modifier-border); border-radius:10px; padding:12px; background:var(--background-primary); color:var(--text-normal);";
root.innerHTML = `
    <style>
        .bfs-controls { display:flex; align-items:center; gap:8px; flex-wrap:wrap; margin-top:10px; }
        .bfs-controls button, .bfs-controls select { min-height:44px; font-size:14px; touch-action:manipulation; }
        .bfs-controls button { padding:6px 12px; }
        .bfs-graph { width:100%; height:auto; display:block; }
        .bfs-state-grid { display:grid; grid-template-columns:1fr 1fr; gap:8px; margin-top:8px; }
        .bfs-code { overflow-x:auto; font-size:13px; }
        @media (max-width:520px) {
            .bfs-controls { gap:6px; }
            .bfs-controls button { flex:1 1 72px; min-height:44px; }
            .bfs-speed { flex:1 1 100%; display:flex; align-items:center; gap:8px; }
            .bfs-speed select { flex:1; min-height:44px; }
            .bfs-counter { width:100%; margin-left:0 !important; text-align:center; }
            .bfs-state-grid { grid-template-columns:1fr; }
            .bfs-legend { gap:8px !important; font-size:11px !important; }
            .bfs-code { font-size:12px; }
        }
    </style>
    <svg class="bfs-graph" viewBox="0 0 620 320" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Interactive BFS graph">
        <defs>
            <marker id="bfs-arrow-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7f8c8d"></path></marker>
            <marker id="bfs-arrow-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#2471a3"></path></marker>
            <marker id="bfs-arrow-orange" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#d35400"></path></marker>
            <marker id="bfs-arrow-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#c0392b"></path></marker>
        </defs>
        <g data-layer="edges"></g>
        <g data-layer="nodes"></g>
    </svg>
    <div class="bfs-legend" style="display:flex; gap:14px; flex-wrap:wrap; margin:0 0 8px; font-size:12px;">
        <span>Orange = current</span><span>Blue = discovered / BFS-tree edge</span><span>Green = processed</span><span>Red dashed = skipped edge</span><span>Gray = unseen</span>
    </div>
    <div data-role="status" aria-live="polite" style="min-height:22px; padding:9px 10px; border-radius:8px; background:var(--background-secondary);"></div>
    <div class="bfs-state-grid">
        <div style="padding:8px; border:1px solid var(--background-modifier-border); border-radius:8px;"><strong>Queue</strong><div data-role="queue" style="margin-top:4px; font-family:var(--font-monospace);"></div></div>
        <div style="padding:8px; border:1px solid var(--background-modifier-border); border-radius:8px;"><strong>Visit order</strong><div data-role="order" style="margin-top:4px; font-family:var(--font-monospace);"></div></div>
    </div>
    <div class="bfs-code" data-role="code" style="margin-top:8px; padding:8px 10px; border-radius:8px; background:var(--background-secondary); font-family:var(--font-monospace); line-height:1.55;">
        <div data-line="1">1&nbsp; mark start visited; enqueue start</div>
        <div data-line="2">2&nbsp; while queue is not empty</div>
        <div data-line="3">3&nbsp;&nbsp;&nbsp; u = dequeue()</div>
        <div data-line="4">4&nbsp;&nbsp;&nbsp; inspect each neighbor v</div>
        <div data-line="5">5&nbsp;&nbsp;&nbsp; if unseen: set level; enqueue v</div>
    </div>
    <div class="bfs-controls">
        <button data-action="previous" aria-label="Previous step">Previous</button>
        <button data-action="play">Play</button>
        <button data-action="next" aria-label="Next step">Next</button>
        <button data-action="reset">Reset</button>
        <label class="bfs-speed" style="margin-left:8px;">Speed <select data-action="speed"><option value="1200">Slow</option><option value="700" selected>Normal</option><option value="350">Fast</option></select></label>
        <strong class="bfs-counter" data-role="counter" style="margin-left:auto;"></strong>
    </div>
`;

const edgeLayer = root.querySelector('[data-layer="edges"]');
const nodeLayer = root.querySelector('[data-layer="nodes"]');
const svg = root.querySelector("svg");

for (const edge of edges) {
    const path = document.createElementNS("http://www.w3.org/2000/svg", "path");
    path.setAttribute("d", edge.path);
    path.setAttribute("data-edge", `${edge.from}-${edge.to}`);
    path.setAttribute("fill", "none");
    path.setAttribute("stroke", "#7f8c8d");
    path.setAttribute("stroke-width", "2.5");
    path.setAttribute("marker-end", "url(#bfs-arrow-gray)");
    edgeLayer.appendChild(path);
}

for (const [name, [x, y]] of Object.entries(nodes)) {
    const group = document.createElementNS("http://www.w3.org/2000/svg", "g");
    group.setAttribute("data-node", name);
    group.setAttribute("transform", `translate(${x} ${y})`);
    group.innerHTML = `<circle r="29" fill="#d5d8dc" stroke="#566573" stroke-width="3"></circle><text text-anchor="middle" y="-3" dominant-baseline="central" font-size="18" font-weight="700" fill="#17202a">${name}</text><text data-level text-anchor="middle" y="16" font-size="12" font-weight="600" fill="#17202a"></text>`;
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
    svg.setAttribute("aria-label", `BFS step ${index + 1} of ${steps.length}: ${step.action}`);
    const processed = new Set(step.processed);
    const discovered = new Set(step.discovered);
    const tree = new Set(step.tree);
    const skipped = new Set(step.skipped);

    for (const group of nodeLayer.querySelectorAll("g[data-node]")) {
        const name = group.getAttribute("data-node");
        const circle = group.querySelector("circle");
        const level = group.querySelector("[data-level]");
        if (name === step.current) {
            circle.setAttribute("fill", "#f8c471"); circle.setAttribute("stroke", "#b03a2e"); circle.setAttribute("stroke-width", "5");
        } else if (processed.has(name)) {
            circle.setAttribute("fill", "#82e0aa"); circle.setAttribute("stroke", "#1e8449"); circle.setAttribute("stroke-width", "3");
        } else if (discovered.has(name)) {
            circle.setAttribute("fill", "#85c1e9"); circle.setAttribute("stroke", "#1f618d"); circle.setAttribute("stroke-width", "3");
        } else {
            circle.setAttribute("fill", "#d5d8dc"); circle.setAttribute("stroke", "#566573"); circle.setAttribute("stroke-width", "3");
        }
        level.textContent = Object.hasOwn(step.levels, name) ? `level ${step.levels[name]}` : "unseen";
    }

    for (const path of edgeLayer.querySelectorAll("path[data-edge]")) {
        const key = path.getAttribute("data-edge");
        let color = "#7f8c8d", width = "2.5", marker = "bfs-arrow-gray", dash = "";
        if (tree.has(key)) { color = "#2471a3"; width = "3.5"; marker = "bfs-arrow-blue"; }
        if (skipped.has(key)) { color = "#c0392b"; marker = "bfs-arrow-red"; dash = "7 5"; }
        if (key === step.edge && step.edgeType === "discover") { color = "#d35400"; width = "5"; marker = "bfs-arrow-orange"; }
        if (key === step.edge && step.edgeType === "skip") { color = "#c0392b"; width = "5"; marker = "bfs-arrow-red"; dash = "7 5"; }
        path.setAttribute("stroke", color); path.setAttribute("stroke-width", width); path.setAttribute("stroke-dasharray", dash); path.setAttribute("marker-end", `url(#${marker})`);
    }

    root.querySelector('[data-role="status"]').textContent = step.action;
    root.querySelector('[data-role="queue"]').textContent = step.queue.length ? `front [${step.queue.join(", ")}] back` : "empty";
    root.querySelector('[data-role="order"]').textContent = step.order.join(" -> ");
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

The levels are `{A}`, `{B, C}`, then `{D, E, F}`. The shortest path from A to F uses two edges through C.

## Why it works
Because the queue is FIFO, all nodes enqueued at distance d are dequeued (and used to discover distance d+1 nodes) before any distance d+1 node is dequeued. By induction, the first time a node is *discovered* (enqueued), it's arrived at via the shortest possible hop count — no shorter path could exist, or that path's endpoint would already have been dequeued earlier. This only holds when every edge has equal weight; with weighted edges, a "later" node could still be cheaper, which is why weighted shortest path needs [[Dijkstra]] instead.

## Template
```python
from collections import deque

# Graph / grid BFS
def bfs(start, target, graph):
    queue = deque([(start, 0)])  # (node, distance)
    visited = {start}
    while queue:
        node, dist = queue.popleft()
        if node == target:
            return dist
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append((neighbor, dist + 1))
    return -1  # not reachable

# Tree level-order
queue = deque([root])
while queue:
    for _ in range(len(queue)):   # process one level
        node = queue.popleft()
        if node.left:  queue.append(node.left)
        if node.right: queue.append(node.right)
```

## Complexity
Time: O(V + E) — each node dequeued once, each edge examined once | Space: O(V) — the queue and visited set hold at most all nodes, worst case an entire "ring" at once

## Common pitfalls
- Marking visited on dequeue instead of enqueue → the same node gets enqueued multiple times before it's ever processed, degrading toward O(V²).
- Forgetting to seed `visited` with the start node before the loop begins.
- Using BFS on a weighted graph — it counts hops, not cost; use [[Dijkstra]] instead.
- For multi-source BFS, forgetting to enqueue *all* sources at distance 0 up front — treating them one at a time gives wrong distances for cells reachable from multiple sources.

## NeetCode examples
- [[08.BinaryTreeLevelOrderTraversal|BinaryTreeLevelOrderTraversal]] — textbook level-order BFS
- [[06.RottenOranges|RottenOranges]] — multi-source BFS from all rotten cells at once
- [[13.WordLadder|WordLadder]] — BFS over word-transformation graph for min steps
- [[08.CourseSchedule|CourseSchedule]] — Kahn's BFS-based topological sort for cycle detection

## Full guide
- [[Job Search/Neetcode/01. Questions/11. Graphs/0.GraphsGuide|Graphs Guide]]
- [[Job Search/Neetcode/01. Questions/07. Tree/0.TreeGuide|Tree Guide]]
