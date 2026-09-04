---
type: concept
tags: [concept, dsa, pattern]
---

# Prim's Algorithm

**TL;DR:** Build a Minimum Spanning Tree by growing one tree outward — always adding the cheapest edge that connects the current tree to a new node, tracked with a min-heap.

## When to reach for it
- "Minimum cost to connect all nodes" (MST problem).
- The graph is **dense** (many edges) — Prim's heap-based growth beats sorting the entire edge list.
- You're already building an adjacency list rather than an explicit edge list.

## How it works
Seed the tree with A, then repeatedly take the cheapest frontier edge that reaches an unvisited node. The heap below is shown in sorted order so the next edge is easy to see.

Use **Previous** and **Next** to watch the tree grow. The graph shows the chosen edges, while the two panels show the frontier heap and the running MST cost.

```freeform
const nodes = {
    A: [75, 165],
    B: [225, 70],
    C: [225, 260],
    D: [410, 70],
    E: [525, 225],
};

const edges = [
    { id: "A-B", a: "A", b: "B", w: 2, label: [143, 105] },
    { id: "A-C", a: "A", b: "C", w: 3, label: [143, 225] },
    { id: "B-C", a: "B", b: "C", w: 1, label: [245, 165] },
    { id: "B-D", a: "B", b: "D", w: 4, label: [318, 52] },
    { id: "C-D", a: "C", b: "D", w: 5, label: [325, 170] },
    { id: "C-E", a: "C", b: "E", w: 6, label: [380, 265] },
    { id: "D-E", a: "D", b: "E", w: 2, label: [490, 130] },
];

const steps = [
    {
        visited: ["A"], current: null, mst: [], stale: [], total: 0,
        heap: ["A-B(2)", "A-C(3)"], line: 1,
        action: "Start at A and push its two outgoing edges."
    },
    {
        visited: ["A", "B"], current: "A-B", mst: ["A-B"], stale: [], total: 2,
        heap: ["B-C(1)", "A-C(3)", "B-D(4)"], line: 3,
        action: "Choose A-B(2) because B is new, then push B's frontier edges."
    },
    {
        visited: ["A", "B", "C"], current: "B-C", mst: ["A-B", "B-C"], stale: [], total: 3,
        heap: ["A-C(3)", "B-D(4)", "C-D(5)", "C-E(6)"], line: 3,
        action: "Choose B-C(1), which reaches C before the older A-C(3) entry."
    },
    {
        visited: ["A", "B", "C"], current: "A-C", mst: ["A-B", "B-C"], stale: ["A-C"], total: 3,
        heap: ["B-D(4)", "C-D(5)", "C-E(6)"], line: 2,
        action: "Skip stale A-C(3) because C is already in the tree."
    },
    {
        visited: ["A", "B", "C", "D"], current: "B-D", mst: ["A-B", "B-C", "B-D"], stale: ["A-C"], total: 7,
        heap: ["D-E(2)", "C-D(5)", "C-E(6)"], line: 3,
        action: "Choose B-D(4), making the cheaper D-E(2) edge available late."
    },
    {
        visited: ["A", "B", "C", "D", "E"], current: "D-E", mst: ["A-B", "B-C", "B-D", "D-E"], stale: ["A-C"], total: 9,
        heap: ["C-D(5) unused", "C-E(6) unused"], line: 4,
        action: "Choose D-E(2) to complete the MST and leave C-D(5) and C-E(6) unused."
    },
];

const root = document.createElement("section");
root.style.cssText = "font:14px system-ui; max-width:760px; border:1px solid var(--background-modifier-border); border-radius:10px; padding:12px; background:var(--background-primary); color:var(--text-normal);";
root.innerHTML = `
    <style>
        .prim-graph { width:100%; height:auto; display:block; }
        .prim-state { display:grid; grid-template-columns:1fr 1fr; gap:8px; margin-top:8px; }
        .prim-panel { min-width:0; padding:8px; border:1px solid var(--background-modifier-border); border-radius:8px; }
        .prim-code { margin-top:8px; padding:8px 10px; overflow-x:auto; border-radius:8px; background:var(--background-secondary); font:13px/1.55 var(--font-monospace); }
        .prim-controls { display:flex; align-items:center; gap:8px; flex-wrap:wrap; margin-top:10px; }
        .prim-controls button, .prim-controls select { min-height:44px; font-size:14px; touch-action:manipulation; }
        .prim-controls button { padding:6px 12px; }
        @media (max-width:520px) {
            .prim-state { grid-template-columns:1fr; }
            .prim-controls { gap:6px; }
            .prim-controls button { flex:1 1 72px; }
            .prim-speed { flex:1 1 100%; display:flex; align-items:center; gap:8px; }
            .prim-speed select { flex:1; }
            .prim-counter { width:100%; margin-left:0 !important; text-align:center; }
            .prim-code { font-size:12px; }
        }
    </style>
    <svg class="prim-graph" viewBox="0 0 620 330" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Interactive Prim algorithm graph">
        <g data-layer="edges"></g>
        <g data-layer="labels"></g>
        <g data-layer="nodes"></g>
    </svg>
    <div style="display:flex; gap:12px; flex-wrap:wrap; margin:0 0 8px; font-size:12px;">
        <span>Orange = current edge</span><span>Blue solid = chosen MST edge</span>
        <span>Red dashed = stale edge</span><span>* = node in the tree</span>
    </div>
    <div data-role="status" aria-live="polite" style="min-height:22px; padding:9px 10px; border-radius:8px; background:var(--background-secondary);"></div>
    <div class="prim-state">
        <div class="prim-panel"><strong>Frontier min-heap</strong><div data-role="heap" style="margin-top:4px; font-family:var(--font-monospace);"></div></div>
        <div class="prim-panel"><strong>MST and total</strong><div data-role="mst" style="margin-top:4px; font-family:var(--font-monospace);"></div></div>
    </div>
    <div class="prim-code" data-role="code">
        <div data-line="1">1&nbsp; push start node and its frontier edges</div>
        <div data-line="2">2&nbsp; pop cheapest; skip if destination visited</div>
        <div data-line="3">3&nbsp; add edge and node; push new frontier edges</div>
        <div data-line="4">4&nbsp; stop when every node is visited</div>
    </div>
    <div class="prim-controls">
        <button data-action="previous">Previous</button>
        <button data-action="play">Play</button>
        <button data-action="next">Next</button>
        <button data-action="reset">Reset</button>
        <label class="prim-speed" style="margin-left:8px;">Speed
            <select data-action="speed"><option value="1200">Slow</option><option value="700" selected>Normal</option><option value="350">Fast</option></select>
        </label>
        <strong class="prim-counter" data-role="counter" style="margin-left:auto;"></strong>
    </div>
`;

const svgNS = "http://www.w3.org/2000/svg";
const edgeLayer = root.querySelector('[data-layer="edges"]');
const labelLayer = root.querySelector('[data-layer="labels"]');
const nodeLayer = root.querySelector('[data-layer="nodes"]');

for (const edge of edges) {
    const line = document.createElementNS(svgNS, "line");
    line.dataset.edge = edge.id;
    line.setAttribute("x1", nodes[edge.a][0]);
    line.setAttribute("y1", nodes[edge.a][1]);
    line.setAttribute("x2", nodes[edge.b][0]);
    line.setAttribute("y2", nodes[edge.b][1]);
    line.setAttribute("stroke-linecap", "round");
    edgeLayer.appendChild(line);

    const label = document.createElementNS(svgNS, "text");
    label.dataset.label = edge.id;
    label.setAttribute("x", edge.label[0]);
    label.setAttribute("y", edge.label[1]);
    label.setAttribute("text-anchor", "middle");
    label.setAttribute("paint-order", "stroke");
    label.setAttribute("stroke", "#ffffff");
    label.setAttribute("stroke-width", "6");
    label.setAttribute("fill", "#17202a");
    label.setAttribute("font-size", "18");
    label.textContent = String(edge.w);
    labelLayer.appendChild(label);
}

for (const [name, [x, y]] of Object.entries(nodes)) {
    const group = document.createElementNS(svgNS, "g");
    group.dataset.node = name;
    group.innerHTML = `<circle cx="${x}" cy="${y}" r="25" stroke-width="3"></circle><text x="${x}" y="${y + 6}" text-anchor="middle" font-size="20" font-weight="700"></text>`;
    nodeLayer.appendChild(group);
}

const svg = root.querySelector("svg");
let index = 0;
let timer = null;

function stopPlaying() {
    if (timer !== null) clearInterval(timer);
    timer = null;
    root.querySelector('[data-action="play"]').textContent = "Play";
}

function render() {
    const step = steps[index];
    svg.setAttribute("aria-label", `Prim step ${index + 1} of ${steps.length}: ${step.action}`);
    for (const edge of edges) {
        const line = root.querySelector(`[data-edge="${edge.id}"]`);
        const label = root.querySelector(`[data-label="${edge.id}"]`);
        const isCurrent = step.current === edge.id;
        const isMst = step.mst.includes(edge.id);
        const isStale = step.stale.includes(edge.id);
        let color = "#7f8c8d", width = 3, dash = "";
        if (isMst) { color = "#2471a3"; width = 5; }
        if (isStale) { color = "#c0392b"; width = 4; dash = "8 6"; }
        if (isCurrent) { color = "#d35400"; width = 6; dash = isStale ? "8 6" : ""; }
        line.setAttribute("stroke", color);
        line.setAttribute("stroke-width", width);
        line.setAttribute("stroke-dasharray", dash);
        label.textContent = String(edge.w);
        label.setAttribute("fill", isCurrent ? "#d35400" : isStale ? "#c0392b" : isMst ? "#2471a3" : "#17202a");
        label.setAttribute("font-weight", isCurrent || isMst || isStale ? "700" : "400");
    }

    for (const [name] of Object.entries(nodes)) {
        const group = root.querySelector(`[data-node="${name}"]`);
        const circle = group.querySelector("circle");
        const visited = step.visited.includes(name);
        circle.setAttribute("fill", visited ? "#d5f5e3" : "#f4f6f7");
        circle.setAttribute("stroke", visited ? "#1e8449" : "#7f8c8d");
        group.querySelector("text").textContent = visited ? `${name}*` : name;
        group.querySelector("text").setAttribute("fill", visited ? "#145a32" : "#17202a");
    }

    root.querySelector('[data-role="status"]').textContent = step.action;
    root.querySelector('[data-role="heap"]').textContent = step.heap.length ? step.heap.join("  |  ") : "empty";
    root.querySelector('[data-role="mst"]').textContent = `${step.mst.length ? step.mst.join(", ") : "none"}  |  total = ${step.total}`;
    for (const line of root.querySelectorAll('[data-role="code"] [data-line]')) {
        const active = Number(line.dataset.line) === step.line;
        line.style.background = active ? "#f8c471" : "transparent";
        line.style.color = active ? "#17202a" : "inherit";
        line.style.fontWeight = active ? "700" : "400";
        if (active) line.setAttribute("aria-current", "step"); else line.removeAttribute("aria-current");
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
    timer = setInterval(() => {
        if (index === steps.length - 1) { stopPlaying(); return; }
        index += 1;
        render();
    }, Number(root.querySelector('[data-action="speed"]').value));
    render();
});
root.querySelector('[data-action="speed"]').addEventListener("change", () => {
    if (timer === null) return;
    stopPlaying();
    root.querySelector('[data-action="play"]').click();
});

render();
display(root);
```

The MST is `B-C(1), A-B(2), D-E(2), B-D(4)`, with a total cost of `9`.

## Why it works
Same **cut property** as Kruskal's, applied differently: at every step, the current visited set and the rest of the graph form a cut, and the heap's top entry is guaranteed to be the cheapest edge crossing that exact cut (every edge from a visited node to an unvisited one is in the heap, and the heap always yields the minimum). Adding that edge is therefore always safe — no cheaper way to bridge visited and unvisited exists. The tree grows by exactly one node per step because any edge to an *already*-visited node would only create a cycle, never extend the tree; those are naturally skipped by the `if u in visited: continue` guard. Stale heap entries — an old, more expensive route to a node that's since been reached more cheaply, or an edge to a node that's already visited by the time it's popped — are left in the heap and simply discarded on pop, the same lazy-deletion pattern used in [[Dijkstra]].

## Template
```python
import heapq

def prim_mst(n, adj):
    # adj[u] = [(weight, v), ...]
    visited = set()
    heap = [(0, 0)]    # (edge_weight, node); seed from node 0
    total = 0

    while heap and len(visited) < n:
        w, u = heapq.heappop(heap)
        if u in visited:
            continue
        visited.add(u)
        total += w
        for cost, v in adj[u]:
            if v not in visited:
                heapq.heappush(heap, (cost, v))

    return total if len(visited) == n else -1
```

## Complexity
Time: O(E log V) with a min-heap — each edge can be pushed once, each push/pop is O(log V) | Space: O(V + E) for the adjacency list and heap

## Common pitfalls
- Forgetting the `if u in visited: continue` guard — without it, stale heap entries get double-counted into `total`.
- Not checking `len(visited) == n` at the end — a disconnected graph leaves nodes unreachable, and returning `total` unguarded would silently report a partial forest's cost as if it were a full MST.
- Building the adjacency list without both directions for an undirected graph — an edge `(u,v)` must appear in both `adj[u]` and `adj[v]`.
- Choosing Prim's for a sparse graph given only as an edge list — building the adjacency list first adds overhead that [[Kruskal's Algorithm]] avoids by sorting the edges directly.

## vs. Kruskal's

| | Prim's | Kruskal's |
|---|---|---|
| Core operation | grow one tree outward via heap | sort all edges globally, union components |
| Data structure | min-heap | sorted edge list + [[Union-Find]] |
| Better for | dense graphs, adjacency list given | sparse graphs, edge list given |
| Interview default | [[02.MinCostToConnectAllPoints\|MinCostToConnectAllPoints]] | problems with explicit sorted edge lists |

## NeetCode examples
- [[02.MinCostToConnectAllPoints|MinCostToConnectAllPoints]] — complete graph (all pairs), Prim's preferred over Kruskal's for dense graphs

## Full guide
[[Job Search/Neetcode/01. Questions/12. Advanced Graphs/0.AdvancedGraphsGuide|Advanced Graphs Guide]]
