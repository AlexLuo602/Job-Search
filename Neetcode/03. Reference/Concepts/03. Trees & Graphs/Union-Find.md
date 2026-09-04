---
type: concept
tags: ["concept"]
---

# Union-Find

**TL;DR:** A disjoint-set structure that answers "are these connected?" and merges groups in near-O(1) amortized time — ideal for connectivity that changes incrementally.

## When to reach for it
- "Are these two nodes in the same group?" as edges are added one at a time.
- Count the number of connected components.
- Detect a cycle in an *undirected* graph while building it edge by edge.
- Merge groups incrementally — Kruskal's MST, redundant connections, accounts/emails belonging to the same person.

## How it works
Each element starts as its own component (its own parent). `union(x, y)` links the roots of x's and y's trees; `find(x)` walks up parent pointers to the root, compressing the path as it goes. Trace `union(0,1)`, `union(1,2)`, `union(3,4)`, `union(2,3)` on 5 singleton nodes (rank starts at 0 for all):

Use **Previous** and **Next** to see each union choice and the final `find(4)`. The forest shows parent pointers, and the two panels show the exact parent and rank arrays.

```freeform
const layouts = {
    singletons: { 0:[70,90], 1:[190,90], 2:[310,90], 3:[430,90], 4:[550,90] },
    zeroOne:    { 0:[120,75], 1:[120,205], 2:[300,75], 3:[430,75], 4:[550,75] },
    zeroTwo:    { 0:[175,70], 1:[115,205], 2:[235,205], 3:[420,70], 4:[550,70] },
    twoTrees:   { 0:[150,65], 1:[90,195], 2:[210,195], 3:[455,65], 4:[455,195] },
    merged:     { 0:[310,45], 1:[105,175], 2:[225,175], 3:[415,175], 4:[500,285] },
    compressed: { 0:[310,45], 1:[70,190], 2:[230,190], 3:[390,190], 4:[550,190] },
};

const steps = [
    {
        parents:[0,1,2,3,4], ranks:[0,0,0,0,0], layout:"singletons", activeNodes:[], activeLinks:[], line:1,
        action:"Initialize five components with each node as a rank-0 root."
    },
    {
        parents:[0,0,2,3,4], ranks:[1,0,0,0,0], layout:"zeroOne", activeNodes:[0,1], activeLinks:["1-0"], line:4,
        action:"Attach 1 under 0 and raise rank[0] to 1 because the roots have equal rank."
    },
    {
        parents:[0,0,0,3,4], ranks:[1,0,0,0,0], layout:"zeroTwo", activeNodes:[0,2], activeLinks:["2-0"], line:3,
        action:"Attach 2 under root 0 because find(1) returns the higher-rank root 0."
    },
    {
        parents:[0,0,0,3,3], ranks:[1,0,0,1,0], layout:"twoTrees", activeNodes:[3,4], activeLinks:["4-3"], line:4,
        action:"Attach 4 under 3 and raise rank[3] to 1 because the roots have equal rank."
    },
    {
        parents:[0,0,0,0,3], ranks:[2,0,0,1,0], layout:"merged", activeNodes:[0,3], activeLinks:["3-0"], line:4,
        action:"Attach root 3 under root 0 and raise rank[0] to 2 because the roots have equal rank."
    },
    {
        parents:[0,0,0,0,3], ranks:[2,0,0,1,0], layout:"merged", activeNodes:[4,3,0], activeLinks:["4-3","3-0"], line:5,
        action:"Follow 4 -> 3 -> 0 to find root 0 before changing parent pointers."
    },
    {
        parents:[0,0,0,0,0], ranks:[2,0,0,1,0], layout:"compressed", activeNodes:[4,0], activeLinks:["4-0"], line:6,
        action:"Set parent[4] to root 0 while leaving every rank unchanged."
    },
];

const root = document.createElement("section");
root.style.cssText = "font:14px system-ui; max-width:760px; border:1px solid var(--background-modifier-border); border-radius:10px; padding:12px; background:var(--background-primary); color:var(--text-normal);";
root.innerHTML = `
    <style>
        .uf-forest { width:100%; height:auto; display:block; }
        .uf-state { display:grid; grid-template-columns:1fr 1fr; gap:8px; margin-top:8px; }
        .uf-panel { min-width:0; padding:8px; border:1px solid var(--background-modifier-border); border-radius:8px; }
        .uf-code { margin-top:8px; padding:8px 10px; overflow-x:auto; border-radius:8px; background:var(--background-secondary); font:13px/1.55 var(--font-monospace); }
        .uf-controls { display:flex; align-items:center; gap:8px; flex-wrap:wrap; margin-top:10px; }
        .uf-controls button, .uf-controls select { min-height:44px; font-size:14px; touch-action:manipulation; }
        .uf-controls button { padding:6px 12px; }
        @media (max-width:520px) {
            .uf-state { grid-template-columns:1fr; }
            .uf-controls { gap:6px; }
            .uf-controls button { flex:1 1 72px; }
            .uf-speed { flex:1 1 100%; display:flex; align-items:center; gap:8px; }
            .uf-speed select { flex:1; }
            .uf-counter { width:100%; margin-left:0 !important; text-align:center; }
            .uf-code { font-size:12px; }
        }
    </style>
    <svg class="uf-forest" viewBox="0 0 620 340" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Interactive Union-Find parent forest">
        <defs>
            <marker id="uf-arrow-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#2471a3"></path></marker>
            <marker id="uf-arrow-orange" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#d35400"></path></marker>
        </defs>
        <g data-layer="links"></g><g data-layer="nodes"></g>
    </svg>
    <div style="display:flex; gap:12px; flex-wrap:wrap; margin:0 0 8px; font-size:12px;">
        <span>Node labeled root = component root</span><span>Blue arrow = parent pointer</span>
        <span>Orange node or arrow = current union/find change</span>
    </div>
    <div data-role="status" aria-live="polite" style="min-height:22px; padding:9px 10px; border-radius:8px; background:var(--background-secondary);"></div>
    <div class="uf-state">
        <div class="uf-panel"><strong>Parent array</strong><div data-role="parents" style="margin-top:4px; font-family:var(--font-monospace);"></div></div>
        <div class="uf-panel"><strong>Rank array</strong><div data-role="ranks" style="margin-top:4px; font-family:var(--font-monospace);"></div></div>
    </div>
    <div class="uf-code" data-role="code">
        <div data-line="1">1&nbsp; parent[i] = i; rank[i] = 0</div>
        <div data-line="2">2&nbsp; rx = find(x); ry = find(y)</div>
        <div data-line="3">3&nbsp; attach lower-rank root under higher root</div>
        <div data-line="4">4&nbsp; on tie: attach ry under rx; rank[rx]++</div>
        <div data-line="5">5&nbsp; find(x): follow parents to the root</div>
        <div data-line="6">6&nbsp; parent[x] = root to compress the path</div>
    </div>
    <div class="uf-controls">
        <button data-action="previous">Previous</button><button data-action="play">Play</button>
        <button data-action="next">Next</button><button data-action="reset">Reset</button>
        <label class="uf-speed" style="margin-left:8px;">Speed
            <select data-action="speed"><option value="1200">Slow</option><option value="700" selected>Normal</option><option value="350">Fast</option></select>
        </label>
        <strong class="uf-counter" data-role="counter" style="margin-left:auto;"></strong>
    </div>
`;

const svgNS = "http://www.w3.org/2000/svg";
const linkLayer = root.querySelector('[data-layer="links"]');
const nodeLayer = root.querySelector('[data-layer="nodes"]');
let index = 0;
let timer = null;

function stopPlaying() {
    if (timer !== null) clearInterval(timer);
    timer = null;
    root.querySelector('[data-action="play"]').textContent = "Play";
}

function drawForest(step) {
    linkLayer.replaceChildren();
    nodeLayer.replaceChildren();
    const positions = layouts[step.layout];

    for (let child = 0; child < step.parents.length; child += 1) {
        const parent = step.parents[child];
        if (child === parent) continue;
        const [cx, cy] = positions[child];
        const [px, py] = positions[parent];
        const dx = px - cx, dy = py - cy;
        const distance = Math.hypot(dx, dy);
        const ux = dx / distance, uy = dy / distance;
        const linkId = `${child}-${parent}`;
        const active = step.activeLinks.includes(linkId);
        const line = document.createElementNS(svgNS, "line");
        line.setAttribute("x1", cx + ux * 27); line.setAttribute("y1", cy + uy * 27);
        line.setAttribute("x2", px - ux * 29); line.setAttribute("y2", py - uy * 29);
        line.setAttribute("stroke", active ? "#d35400" : "#2471a3");
        line.setAttribute("stroke-width", active ? "5" : "4");
        line.setAttribute("marker-end", active ? "url(#uf-arrow-orange)" : "url(#uf-arrow-blue)");
        linkLayer.appendChild(line);
    }

    for (let node = 0; node < step.parents.length; node += 1) {
        const [x, y] = positions[node];
        const isRoot = step.parents[node] === node;
        const active = step.activeNodes.includes(node);
        const group = document.createElementNS(svgNS, "g");
        const fill = active ? "#f8c471" : isRoot ? "#d5f5e3" : "var(--background-primary)";
        const stroke = active ? "#d35400" : isRoot ? "#1e8449" : "#566573";
        const textColor = "#17202a";
        const secondLine = isRoot ? `root r${step.ranks[node]}` : `parent ${step.parents[node]}`;
        group.innerHTML = `<circle cx="${x}" cy="${y}" r="32" fill="${fill}" stroke="${stroke}" stroke-width="4"></circle><text x="${x}" y="${y - 4}" text-anchor="middle" fill="${textColor}" font-size="20" font-weight="700">${node}</text><text x="${x}" y="${y + 17}" text-anchor="middle" fill="${textColor}" font-size="15" font-weight="700">${secondLine}</text>`;
        nodeLayer.appendChild(group);
    }
}

function render() {
    const step = steps[index];
    drawForest(step);
    root.querySelector("svg").setAttribute("aria-label", `Union-Find step ${index + 1} of ${steps.length}: ${step.action}`);
    root.querySelector('[data-role="status"]').textContent = step.action;
    root.querySelector('[data-role="parents"]').textContent = `[${step.parents.join(", ")}]`;
    root.querySelector('[data-role="ranks"]').textContent = `[${step.ranks.join(", ")}]`;
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

After the four unions, every node has root `0`. Calling `find(4)` changes its path from `4 -> 3 -> 0` to the direct link `4 -> 0`.

## Why it works
**Union by rank** always attaches the shallower tree under the deeper one's root, so tree height grows only logarithmically with merges — you never accidentally build a long chain. **Path compression** flattens every node touched by a `find` to point straight at the root, so repeated queries on the same elements get cheaper over time. Combined, no single operation is ever worse than O(log n), and the amortized cost across a full sequence of operations drops to O(α(n)) — the inverse Ackermann function, which grows so slowly it's under 5 for any input size that could ever exist in practice, so it's treated as a constant. (The full proof is a standard algorithms-course result; the practical takeaway is just "effectively O(1) per operation.") The cycle-detection trick falls out for free: if `find(u) == find(v)` before you union them, they were already connected, so adding edge `(u,v)` would close a cycle.

## Template
```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank   = [0] * n
        self.components = n

    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])  # path compression
        return self.parent[x]

    def union(self, x, y):
        px, py = self.find(x), self.find(y)
        if px == py:
            return False   # already connected (cycle if adding an edge)
        # union by rank
        if self.rank[px] < self.rank[py]:
            px, py = py, px
        self.parent[py] = px
        if self.rank[px] == self.rank[py]:
            self.rank[px] += 1
        self.components -= 1
        return True
```

## Complexity
Time: O(α(n)) amortized per `find`/`union` (α = inverse Ackermann, effectively constant) — without both optimizations a naive version degrades to O(n) per operation on a skewed chain | Space: O(n) for the parent and rank arrays

## Common pitfalls
- Implementing only one of path compression / union by rank — either alone is fine, but skipping both lets trees degenerate into long chains (O(n) per find).
- Confusing what `union`'s return value means — some codebases return True for "merged," others for "already connected"; pick one convention and be consistent, since it's usually the cycle-detection signal.
- Using 1-indexed node labels without sizing the parent array to match (off-by-one → index errors or silently wrong roots).
- Comparing `x == y` instead of `find(x) == find(y)` to check connectivity — two different labels can already share a root.

## NeetCode examples
- [[11.NumberOfConnectedComponentsInAnUndirectedGraph|NumberOfConnectedComponentsInAnUndirectedGraph]] — count components via union-find
- [[10.RedundantConnection|RedundantConnection]] — first union() that returns False is the redundant edge
- [[12.GraphValidTree|GraphValidTree]] — valid tree iff no cycle and exactly n-1 edges

## Full guide
[[Job Search/Neetcode/01. Questions/11. Graphs/0.GraphsGuide|Graphs Guide]]
