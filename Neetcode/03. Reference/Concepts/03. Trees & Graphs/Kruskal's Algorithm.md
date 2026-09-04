---
type: concept
tags: [concept, dsa, pattern]
---

# Kruskal's Algorithm

**TL;DR:** Build a Minimum Spanning Tree by sorting all edges cheapest-first and greedily adding each one that doesn't create a cycle, using [[Union-Find]] to check that in near-O(1).

## When to reach for it
- "Minimum cost to connect all nodes" (MST problem).
- The graph is **sparse** (few edges relative to nodes) or edges are already given as an explicit list.
- The problem is naturally about edges rather than adjacency (e.g. a list of `(cost, u, v)` pairs).

## How it works
Sort every edge by weight, then add an edge only when it joins two different components.

Use **Previous** and **Next** to move through the sorted edge list. The graph keeps accepted and skipped edges visible, while the panels show the cursor, components, and running cost.

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

const sorted = ["B-C", "A-B", "D-E", "A-C", "B-D", "C-D", "C-E"];
const byId = Object.fromEntries(edges.map(edge => [edge.id, edge]));

const steps = [
    {
        cursor: -1, current: null, accepted: [], skipped: [], total: 0,
        components: ["{A}", "{B}", "{C}", "{D}", "{E}"], line: 1,
        action: "Sort all edges by weight while each node starts in its own component."
    },
    {
        cursor: 0, current: "B-C", accepted: ["B-C"], skipped: [], total: 1,
        components: ["{B,C}", "{A}", "{D}", "{E}"], line: 3,
        action: "Accept B-C(1) because B and C are in different components."
    },
    {
        cursor: 1, current: "A-B", accepted: ["B-C", "A-B"], skipped: [], total: 3,
        components: ["{A,B,C}", "{D}", "{E}"], line: 3,
        action: "Accept A-B(2) to add A to the component containing B and C."
    },
    {
        cursor: 2, current: "D-E", accepted: ["B-C", "A-B", "D-E"], skipped: [], total: 5,
        components: ["{A,B,C}", "{D,E}"], line: 3,
        action: "Accept D-E(2) to join D and E in a second component."
    },
    {
        cursor: 3, current: "A-C", accepted: ["B-C", "A-B", "D-E"], skipped: ["A-C"], total: 5,
        components: ["{A,B,C}", "{D,E}"], line: 4,
        action: "Skip A-C(3) because A and C already share a root."
    },
    {
        cursor: 4, current: "B-D", accepted: ["B-C", "A-B", "D-E", "B-D"], skipped: ["A-C"], total: 9,
        components: ["{A,B,C,D,E}"], line: 3,
        action: "Accept B-D(4) to join the final two components."
    },
    {
        cursor: 4, current: null, accepted: ["B-C", "A-B", "D-E", "B-D"], skipped: ["A-C"], total: 9,
        components: ["{A,B,C,D,E}"], line: 5,
        action: "Stop after four accepted edges because five nodes need only four MST edges."
    },
];

const root = document.createElement("section");
root.style.cssText = "font:14px system-ui; max-width:760px; border:1px solid var(--background-modifier-border); border-radius:10px; padding:12px; background:var(--background-primary); color:var(--text-normal);";
root.innerHTML = `
    <style>
        .kruskal-graph { width:100%; height:auto; display:block; }
        .kruskal-state { display:grid; grid-template-columns:1fr 1fr; gap:8px; margin-top:8px; }
        .kruskal-panel { min-width:0; padding:8px; border:1px solid var(--background-modifier-border); border-radius:8px; }
        .kruskal-code { margin-top:8px; padding:8px 10px; overflow-x:auto; border-radius:8px; background:var(--background-secondary); font:13px/1.55 var(--font-monospace); }
        .kruskal-controls { display:flex; align-items:center; gap:8px; flex-wrap:wrap; margin-top:10px; }
        .kruskal-controls button, .kruskal-controls select { min-height:44px; font-size:14px; touch-action:manipulation; }
        .kruskal-controls button { padding:6px 12px; }
        @media (max-width:520px) {
            .kruskal-state { grid-template-columns:1fr; }
            .kruskal-controls { gap:6px; }
            .kruskal-controls button { flex:1 1 72px; }
            .kruskal-speed { flex:1 1 100%; display:flex; align-items:center; gap:8px; }
            .kruskal-speed select { flex:1; }
            .kruskal-counter { width:100%; margin-left:0 !important; text-align:center; }
            .kruskal-code { font-size:12px; }
        }
    </style>
    <svg class="kruskal-graph" viewBox="0 0 620 330" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Interactive Kruskal algorithm graph">
        <g data-layer="edges"></g><g data-layer="labels"></g><g data-layer="nodes"></g>
    </svg>
    <div style="display:flex; gap:12px; flex-wrap:wrap; margin:0 0 8px; font-size:12px;">
        <span>Orange = edge under review</span><span>Blue solid = accepted</span>
        <span>Red dashed = rejected cycle</span><span>Gray = not processed</span>
    </div>
    <div data-role="status" aria-live="polite" style="min-height:22px; padding:9px 10px; border-radius:8px; background:var(--background-secondary);"></div>
    <div class="kruskal-state">
        <div class="kruskal-panel"><strong>Sorted edge cursor</strong><div data-role="sorted" style="margin-top:4px; font-family:var(--font-monospace); line-height:1.5;"></div></div>
        <div class="kruskal-panel"><strong>Components and result</strong><div data-role="components" style="margin-top:4px; font-family:var(--font-monospace); line-height:1.5;"></div></div>
    </div>
    <div class="kruskal-code" data-role="code">
        <div data-line="1">1&nbsp; sort edges by weight</div>
        <div data-line="2">2&nbsp; for each edge (u, v)</div>
        <div data-line="3">3&nbsp;&nbsp;&nbsp; if roots differ: union and add cost</div>
        <div data-line="4">4&nbsp;&nbsp;&nbsp; else: skip cycle edge</div>
        <div data-line="5">5&nbsp; stop after V - 1 accepted edges</div>
    </div>
    <div class="kruskal-controls">
        <button data-action="previous">Previous</button><button data-action="play">Play</button>
        <button data-action="next">Next</button><button data-action="reset">Reset</button>
        <label class="kruskal-speed" style="margin-left:8px;">Speed
            <select data-action="speed"><option value="1200">Slow</option><option value="700" selected>Normal</option><option value="350">Fast</option></select>
        </label>
        <strong class="kruskal-counter" data-role="counter" style="margin-left:auto;"></strong>
    </div>
`;

const svgNS = "http://www.w3.org/2000/svg";
const edgeLayer = root.querySelector('[data-layer="edges"]');
const labelLayer = root.querySelector('[data-layer="labels"]');
const nodeLayer = root.querySelector('[data-layer="nodes"]');

for (const edge of edges) {
    const line = document.createElementNS(svgNS, "line");
    line.dataset.edge = edge.id;
    line.setAttribute("x1", nodes[edge.a][0]); line.setAttribute("y1", nodes[edge.a][1]);
    line.setAttribute("x2", nodes[edge.b][0]); line.setAttribute("y2", nodes[edge.b][1]);
    line.setAttribute("stroke-linecap", "round");
    edgeLayer.appendChild(line);

    const label = document.createElementNS(svgNS, "text");
    label.dataset.label = edge.id;
    label.setAttribute("x", edge.label[0]); label.setAttribute("y", edge.label[1]);
    label.setAttribute("text-anchor", "middle"); label.setAttribute("paint-order", "stroke");
    label.setAttribute("stroke", "#ffffff"); label.setAttribute("stroke-width", "6");
    label.setAttribute("font-size", "18"); label.setAttribute("fill", "#17202a");
    label.textContent = String(edge.w);
    labelLayer.appendChild(label);
}

for (const [name, [x, y]] of Object.entries(nodes)) {
    const group = document.createElementNS(svgNS, "g");
    group.innerHTML = `<circle cx="${x}" cy="${y}" r="25" fill="#f4f6f7" stroke="#566573" stroke-width="3"></circle><text x="${x}" y="${y + 6}" text-anchor="middle" font-size="20" font-weight="700" fill="#17202a">${name}</text>`;
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
    svg.setAttribute("aria-label", `Kruskal step ${index + 1} of ${steps.length}: ${step.action}`);
    for (const edge of edges) {
        const line = root.querySelector(`[data-edge="${edge.id}"]`);
        const label = root.querySelector(`[data-label="${edge.id}"]`);
        const accepted = step.accepted.includes(edge.id);
        const skipped = step.skipped.includes(edge.id);
        const current = step.current === edge.id;
        let color = "#7f8c8d", width = 3, dash = "";
        if (accepted) { color = "#2471a3"; width = 5; }
        if (skipped) { color = "#c0392b"; width = 4; dash = "8 6"; }
        if (current) { color = "#d35400"; width = 6; dash = skipped ? "8 6" : ""; }
        line.setAttribute("stroke", color); line.setAttribute("stroke-width", width); line.setAttribute("stroke-dasharray", dash);
        label.textContent = String(edge.w);
        label.setAttribute("fill", current ? "#d35400" : skipped ? "#c0392b" : accepted ? "#2471a3" : "#17202a");
        label.setAttribute("font-weight", current || accepted || skipped ? "700" : "400");
    }

    root.querySelector('[data-role="status"]').textContent = step.action;
    root.querySelector('[data-role="sorted"]').innerHTML = sorted.map((id, position) => {
        const edge = byId[id];
        const prefix = position === step.cursor && step.current ? "> " : "&nbsp;&nbsp;";
        const state = step.accepted.includes(id) ? "  accepted" : step.skipped.includes(id) ? " skip" : position > step.cursor && index === steps.length - 1 ? " not needed" : "";
        return `<div>${prefix}${id}(${edge.w})${state}</div>`;
    }).join("");
    root.querySelector('[data-role="components"]').innerHTML = `<div>${step.components.join("  ")}</div><div style="margin-top:5px;">accepted = ${step.accepted.length}/4 &nbsp; total = ${step.total}</div>`;
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
        index += 1; render();
    }, Number(root.querySelector('[data-action="speed"]').value));
    render();
});
root.querySelector('[data-action="speed"]').addEventListener("change", () => {
    if (timer === null) return;
    stopPlaying(); root.querySelector('[data-action="play"]').click();
});

render();
display(root);
```

The MST is `B-C(1), A-B(2), D-E(2), B-D(4)`, with a total cost of `9`.

## Why it works
This is the **cut property**: for any way of splitting the graph's nodes into two non-empty groups, the minimum-weight edge crossing that split must be in *some* MST — including any edge already skipped as "cheaper," an even-cheaper alternative crossing the same cut would have been picked first. Kruskal's applies this one edge at a time: at the moment each edge is considered (cheapest-first order), if its endpoints are in different components, that edge is the minimum-weight edge crossing the cut between "this component" and "everything else" — so it's always safe to add. If the endpoints are already in the same component, the edge crosses no cut between components at all — it would only close a cycle, never help connectivity — so skipping it is exactly right, not just convenient. [[Union-Find]]'s `union` does double duty here: it merges components *and* reports whether they were already connected, which is precisely the check the cut property needs.

## Template
```python
def kruskal_mst(n, edges):
    # edges = [(weight, u, v)]
    edges.sort()
    uf = UnionFind(n)
    total, count = 0, 0

    for w, u, v in edges:
        if uf.union(u, v):          # True if different components (no cycle)
            total += w
            count += 1
            if count == n - 1:      # MST complete: V−1 edges
                break

    return total if count == n - 1 else -1
```

## Complexity
Time: O(E log E) to sort the edges (dominates), plus O(E · α(V)) for the union-find operations — overall O(E log E) | Space: O(V + E) for the union-find arrays and edge list

## Common pitfalls
- Forgetting to sort edges before processing — the greedy cheapest-first order is the entire algorithm; without it, the cut-property argument doesn't apply.
- Not stopping (or not checking) once V−1 edges are added — processing the full edge list wastes time and, if not counted, obscures whether a valid MST was actually found.
- Returning a total cost without verifying `count == n - 1` — a disconnected graph never reaches V−1 edges, so the "total" would silently describe only a spanning forest, not a spanning tree.
- Using union-find without path compression / union by rank, degrading the "near-O(1)" union cost to O(n) on adversarial edge orders.

## vs. Prim's

| | Kruskal's | Prim's |
|---|---|---|
| Core operation | sort all edges, union components | grow one tree via heap |
| Data structure | sorted edge list + [[Union-Find]] | min-heap |
| Better for | sparse graphs, edge list given | dense graphs, adjacency list given |
| Interview default | problems with explicit edge lists | [[02.MinCostToConnectAllPoints\|MinCostToConnectAllPoints]] |

## NeetCode examples
- [[02.MinCostToConnectAllPoints|MinCostToConnectAllPoints]] — uses Prim's but Kruskal's is a valid alternative
- [[10.RedundantConnection|RedundantConnection]] — uses Union-Find's cycle detection, the core idea behind Kruskal's

## Full guide
[[Job Search/Neetcode/01. Questions/12. Advanced Graphs/0.AdvancedGraphsGuide|Advanced Graphs Guide]]
