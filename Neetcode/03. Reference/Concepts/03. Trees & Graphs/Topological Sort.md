---
type: concept
tags: ["concept"]
---

# Topological Sort

**TL;DR:** Order the nodes of a directed acyclic graph (DAG) so every edge points from earlier to later — the only valid ordering when work has prerequisites.

## When to reach for it
- Scheduling / dependency ordering: "must A happen before B?", course prerequisites, build systems.
- Cycle detection in a *directed* graph: if a valid ordering can't include every node, a cycle exists.
- Shortest or longest path on a DAG: relax edges in topological order and no heap is needed.
- Deriving an ordering from pairwise constraints (e.g. lexicographic word comparisons → letter order).

## How it works
**Kahn's algorithm (BFS on in-degrees).** Repeatedly output a node whose in-degree is 0 (no unmet prerequisites), then decrement its neighbors' in-degrees — some of them now hit 0 and become eligible. Trace on `A→B, A→C, B→D, C→D`:

The interactive walkthrough shows when a node enters the zero-indegree queue, which edge lowers an indegree, and how the final output length detects a cycle.

```freeform
const nodes = {
    A: [90, 145],
    B: [275, 65],
    C: [275, 225],
    D: [500, 145],
};

const edges = [
    { key: "A-B", from: "A", to: "B", path: "M116 134 L249 76" },
    { key: "A-C", from: "A", to: "C", path: "M116 156 L249 214" },
    { key: "B-D", from: "B", to: "D", path: "M301 76 L474 134" },
    { key: "C-D", from: "C", to: "D", path: "M301 214 L474 156" },
];

const steps = [
    { node: null, edge: null, queue: ["A"], output: [], indegree: { A: 0, B: 1, C: 1, D: 2 }, processed: [], line: 1, action: "Initialize the queue with A because its indegree is 0." },
    { node: "A", edge: null, queue: [], output: ["A"], indegree: { A: 0, B: 1, C: 1, D: 2 }, processed: [], line: 2, action: "Remove A from the queue and add it to the output." },
    { node: "A", edge: "A-B", queue: ["B"], output: ["A"], indegree: { A: 0, B: 0, C: 1, D: 2 }, processed: ["A-B"], line: 3, action: "Remove A -> B and add B to the queue when its indegree reaches 0." },
    { node: "A", edge: "A-C", queue: ["B", "C"], output: ["A"], indegree: { A: 0, B: 0, C: 0, D: 2 }, processed: ["A-B", "A-C"], line: 3, action: "Remove A -> C and add C to the queue when its indegree reaches 0." },
    { node: "B", edge: null, queue: ["C"], output: ["A", "B"], indegree: { A: 0, B: 0, C: 0, D: 2 }, processed: ["A-B", "A-C"], line: 2, action: "Remove B from the queue and add it to the output." },
    { node: "B", edge: "B-D", queue: ["C"], output: ["A", "B"], indegree: { A: 0, B: 0, C: 0, D: 1 }, processed: ["A-B", "A-C", "B-D"], line: 3, action: "Remove B -> D while keeping D out of the queue at indegree 1." },
    { node: "C", edge: null, queue: [], output: ["A", "B", "C"], indegree: { A: 0, B: 0, C: 0, D: 1 }, processed: ["A-B", "A-C", "B-D"], line: 2, action: "Remove C from the queue and add it to the output." },
    { node: "C", edge: "C-D", queue: ["D"], output: ["A", "B", "C"], indegree: { A: 0, B: 0, C: 0, D: 0 }, processed: ["A-B", "A-C", "B-D", "C-D"], line: 3, action: "Remove C -> D and add D to the queue when its indegree reaches 0." },
    { node: "D", edge: null, queue: [], output: ["A", "B", "C", "D"], indegree: { A: 0, B: 0, C: 0, D: 0 }, processed: ["A-B", "A-C", "B-D", "C-D"], line: 2, action: "Remove D from the queue and add it to the output." },
    { node: null, edge: null, queue: [], output: ["A", "B", "C", "D"], indegree: { A: 0, B: 0, C: 0, D: 0 }, processed: ["A-B", "A-C", "B-D", "C-D"], line: 5, action: "The output has all 4 nodes, so the graph has no cycle." },
];

const root = document.createElement("section");
root.style.cssText = "font:14px system-ui;max-width:760px;border:1px solid var(--background-modifier-border);border-radius:10px;padding:12px;background:var(--background-primary);color:var(--text-normal);";
root.innerHTML = `
<style>
  .topo-graph{display:block;width:100%;height:auto}.topo-state{display:grid;grid-template-columns:1fr 2fr;gap:8px;margin-top:8px}.topo-panel{padding:8px;border:1px solid var(--background-modifier-border);border-radius:8px}.topo-code{margin-top:8px;padding:8px 10px;border-radius:8px;background:var(--background-secondary);font:13px/1.55 var(--font-monospace);overflow-x:auto}.topo-controls{display:flex;align-items:center;gap:8px;flex-wrap:wrap;margin-top:10px}.topo-controls button,.topo-controls select{min-height:44px;font-size:14px;touch-action:manipulation}.topo-controls button{padding:6px 12px}
  @media(max-width:520px){.topo-state{grid-template-columns:1fr}.topo-controls button{flex:1 1 72px}.topo-speed{display:flex;align-items:center;gap:8px;flex:1 1 100%}.topo-speed select{flex:1}.topo-counter{width:100%;margin-left:0!important;text-align:center}.topo-code{font-size:12px}}
</style>
<svg class="topo-graph" viewBox="0 0 600 300" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Interactive topological sort graph">
  <defs>
    <marker id="topo-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0L10 5L0 10z" fill="#6c757d"></path></marker>
    <marker id="topo-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0L10 5L0 10z" fill="#2471a3"></path></marker>
    <marker id="topo-orange" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0L10 5L0 10z" fill="#d35400"></path></marker>
  </defs>
  <g data-layer="edges"></g><g data-layer="nodes"></g>
</svg>
<div style="display:flex;gap:12px;flex-wrap:wrap;font-size:12px;margin-bottom:8px"><span>Orange = current</span><span>Blue dashed = removed edge</span><span>Green + done = processed node</span><span>Blue + queued = ready node</span></div>
<div data-role="status" aria-live="polite" style="min-height:22px;padding:9px 10px;border-radius:8px;background:var(--background-secondary)"></div>
<div class="topo-state">
  <div class="topo-panel"><strong>Zero-indegree queue</strong><div data-role="queue" style="margin-top:4px;font-family:var(--font-monospace)"></div></div>
  <div class="topo-panel"><strong>Output and indegrees</strong><div data-role="output" style="margin-top:4px;font-family:var(--font-monospace)"></div><div data-role="indegree" style="margin-top:4px;font-family:var(--font-monospace)"></div></div>
</div>
<div class="topo-code" data-role="code">
  <div data-line="1">1&nbsp; queue every node with indegree 0</div>
  <div data-line="2">2&nbsp; remove u; append u to output</div>
  <div data-line="3">3&nbsp; lower each neighbor's indegree; enqueue at 0</div>
  <div data-line="4">4&nbsp; repeat while the queue is not empty</div>
  <div data-line="5">5&nbsp; cycle exists if output size &lt; node count</div>
</div>
<div class="topo-controls">
  <button data-action="previous">Previous</button><button data-action="play">Play</button><button data-action="next">Next</button><button data-action="reset">Reset</button>
  <label class="topo-speed" style="margin-left:8px">Speed <select data-action="speed"><option value="1200">Slow</option><option value="700" selected>Normal</option><option value="350">Fast</option></select></label>
  <strong class="topo-counter" data-role="counter" style="margin-left:auto"></strong>
</div>`;

const svg = root.querySelector("svg");
const edgeLayer = root.querySelector('[data-layer="edges"]');
const nodeLayer = root.querySelector('[data-layer="nodes"]');

for (const edge of edges) {
    const path = document.createElementNS("http://www.w3.org/2000/svg", "path");
    path.setAttribute("d", edge.path); path.setAttribute("data-edge", edge.key); path.setAttribute("fill", "none");
    edgeLayer.appendChild(path);
}
for (const [name, [x, y]] of Object.entries(nodes)) {
    const group = document.createElementNS("http://www.w3.org/2000/svg", "g");
    group.setAttribute("data-node", name); group.setAttribute("transform", `translate(${x} ${y})`);
    group.innerHTML = `<circle r="28"></circle><text text-anchor="middle" dominant-baseline="central" font-size="20" font-weight="700" fill="#17202a">${name}</text><text data-role="node-state" y="49" text-anchor="middle" font-size="15" font-weight="700" paint-order="stroke" stroke="#ffffff" stroke-width="5" fill="currentColor"></text>`;
    nodeLayer.appendChild(group);
}

let index = 0;
let timer = null;
function stopPlaying() { if (timer !== null) clearInterval(timer); timer = null; root.querySelector('[data-action="play"]').textContent = "Play"; }
function render() {
    const step = steps[index];
    svg.setAttribute("aria-label", `Topological sort step ${index + 1} of ${steps.length}: ${step.action}`);
    const queued = new Set(step.queue), output = new Set(step.output), processed = new Set(step.processed);
    for (const group of nodeLayer.querySelectorAll('g[data-node]')) {
        const name = group.getAttribute("data-node"), circle = group.querySelector("circle"), label = group.querySelector('[data-role="node-state"]');
        let fill = "#d5d8dc", stroke = "#566573", state = `in:${step.indegree[name]}`;
        if (output.has(name)) { fill = "#82e0aa"; stroke = "#1e8449"; state += " done"; }
        if (queued.has(name)) { fill = "#85c1e9"; stroke = "#1f618d"; state += " queued"; }
        if (name === step.node) { fill = "#f8c471"; stroke = "#b03a2e"; state += " current"; }
        circle.setAttribute("fill", fill); circle.setAttribute("stroke", stroke); circle.setAttribute("stroke-width", name === step.node ? "5" : "3"); label.textContent = state;
    }
    for (const path of edgeLayer.querySelectorAll('path[data-edge]')) {
        const key = path.getAttribute("data-edge"), current = key === step.edge, done = processed.has(key);
        path.setAttribute("stroke", current ? "#d35400" : done ? "#2471a3" : "#6c757d");
        path.setAttribute("stroke-width", current ? "5" : done ? "3.5" : "2.5");
        path.setAttribute("stroke-dasharray", done && !current ? "7 5" : "");
        path.setAttribute("marker-end", `url(#${current ? "topo-orange" : done ? "topo-blue" : "topo-gray"})`);
    }
    root.querySelector('[data-role="status"]').textContent = step.action;
    root.querySelector('[data-role="queue"]').textContent = step.queue.length ? `[${step.queue.join(", ")}]` : "empty";
    root.querySelector('[data-role="output"]').textContent = `output = [${step.output.join(", ")}]`;
    root.querySelector('[data-role="indegree"]').textContent = `in = {A:${step.indegree.A}, B:${step.indegree.B}, C:${step.indegree.C}, D:${step.indegree.D}}`;
    for (const line of root.querySelectorAll('[data-role="code"] [data-line]')) {
        const active = Number(line.getAttribute("data-line")) === step.line;
        line.style.cssText = active ? "background:#f8c471;color:#17202a;font-weight:700" : "";
        if (active) line.setAttribute("aria-current", "step"); else line.removeAttribute("aria-current");
    }
    root.querySelector('[data-role="counter"]').textContent = `Step ${index + 1} / ${steps.length}`;
    root.querySelector('[data-action="previous"]').disabled = index === 0; root.querySelector('[data-action="next"]').disabled = index === steps.length - 1;
}
root.querySelector('[data-action="previous"]').addEventListener("click", () => { stopPlaying(); index = Math.max(0, index - 1); render(); });
root.querySelector('[data-action="next"]').addEventListener("click", () => { stopPlaying(); index = Math.min(steps.length - 1, index + 1); render(); });
root.querySelector('[data-action="reset"]').addEventListener("click", () => { stopPlaying(); index = 0; render(); });
root.querySelector('[data-action="play"]').addEventListener("click", () => { if (timer !== null) return stopPlaying(); if (index === steps.length - 1) index = 0; root.querySelector('[data-action="play"]').textContent = "Pause"; timer = setInterval(() => { if (index === steps.length - 1) return stopPlaying(); index += 1; render(); }, Number(root.querySelector('[data-action="speed"]').value)); render(); });
root.querySelector('[data-action="speed"]').addEventListener("change", () => { if (timer === null) return; stopPlaying(); root.querySelector('[data-action="play"]').click(); });
render();
display(root);
```

Valid order: `A, B, C, D` (or `A, C, B, D`). If instead `D→A` existed too, A's in-degree would never start at 0, nothing would ever enter the queue, and `len(order) < n` would flag the cycle.

**DFS post-order (alternative).** Recurse into all of a node's descendants first, *then* append the node itself. Since a node is only appended after everything reachable from it, reversing the final list puts every prerequisite before its dependent.

## Why it works
Kahn's invariant: a node with in-degree 0 has every prerequisite already output, so it's always safe to output next. Decrementing a processed node's neighbors' in-degrees means "this prerequisite is now satisfied for you," propagating readiness forward. Cycle members mutually depend on each other, so none of their in-degrees can reach 0 without one going first — they stay stuck outside the queue forever, which is why `len(order) < n` is a complete cycle check.

DFS post-order works differently: appending `node` only after all its reachable descendants guarantees every node appears *after* everything it depends on in the raw list — reversing flips that to "before," a valid order. A plain `visited` boolean avoids re-exploring nodes but **cannot** detect cycles, since a node visited elsewhere isn't necessarily a back edge. Cycle detection needs 3-color state: `0` unvisited, `1` on the current path, `2` done. Reaching a `1`-state node means a back edge to an ancestor (see [[DFS]] for the full template).

## Template
```python
from collections import deque

# Kahn's algorithm (BFS on in-degrees)
def topo_sort_kahn(n, edges):
    adj = [[] for _ in range(n)]
    indegree = [0] * n
    for pre, dep in edges:           # pre must come before dep
        adj[pre].append(dep)
        indegree[dep] += 1

    queue = deque(i for i in range(n) if indegree[i] == 0)
    order = []
    while queue:
        node = queue.popleft()
        order.append(node)
        for nbr in adj[node]:
            indegree[nbr] -= 1
            if indegree[nbr] == 0:
                queue.append(nbr)

    return order if len(order) == n else []   # [] means a cycle exists

# DFS post-order (alternative)
def topo_sort_dfs(n, edges):
    adj = [[] for _ in range(n)]
    for pre, dep in edges:
        adj[pre].append(dep)

    visited = [False] * n
    result = []

    def dfs(node):
        visited[node] = True
        for nbr in adj[node]:
            if not visited[nbr]:
                dfs(nbr)
        result.append(node)   # post-order: added after all reachable descendants

    for i in range(n):
        if not visited[i]:
            dfs(i)

    return result[::-1]
```

**Kahn's vs DFS — when to reach for each:**

| | Kahn's (BFS) | DFS post-order |
|---|---|---|
| Cycle detection | `len(order) < n` — no extra state | needs 3-color "in-stack" flag |
| Output order | correct directly on dequeue | must reverse the post-order list |
| Prefer when | need order + cycle check in one clean pass | already doing DFS for other reasons, or need to locate the actual cycle |

## Complexity
Time: O(V + E) — each node and edge processed a constant number of times either way | Space: O(V + E) for the adjacency list, indegree/visited arrays, and output

## Common pitfalls
- Building the edge direction backwards (prerequisite → dependent vs. dependent → prerequisite) — get this straight before coding, or the whole ordering comes out inverted.
- Forgetting the cycle check (`len(order) == n` for Kahn's) — without it, a cyclic graph silently returns a partial, invalid ordering.
- Isolated nodes with no edges: make sure the in-degree array and output loop still account for every node, not just ones that appear in an edge.
- Using a binary `visited` flag for DFS cycle detection instead of 3-color state — "already visited" can mean either a finished branch reached by a cross edge (no cycle) or an ancestor on the current path (cycle); a single boolean can't tell the two apart.

## NeetCode examples
- [[08.CourseSchedule|CourseSchedule]] — cycle detection via topo sort
- [[09.CourseScheduleII|CourseScheduleII]] — return the actual ordering
- [[05.AlienDictionary|AlienDictionary]] — derive edges from adjacent word pairs, then topo sort

## Full guide
[[Job Search/Neetcode/01. Questions/11. Graphs/0.GraphsGuide|Graphs Guide]] · [[Job Search/Neetcode/01. Questions/12. Advanced Graphs/0.AdvancedGraphsGuide|Advanced Graphs Guide]]
