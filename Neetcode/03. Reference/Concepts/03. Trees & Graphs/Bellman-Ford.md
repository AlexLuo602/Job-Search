---
type: concept
tags: [concept, dsa, pattern]
---

# Bellman-Ford

**TL;DR:** Bellman-Ford finds shortest paths when edges may have negative weights. It relaxes every edge `V - 1` times, so it is slower than [[Dijkstra]] but works in cases where Dijkstra can return a wrong answer.

## When to reach for it
- Shortest path with **negative edge weights**. Dijkstra's greedy pop is unsafe here and can return a wrong answer without an error.
- Detecting a **negative cycle** (a V-th pass that still finds an improvement proves one exists).
- A **k-hop constraint**. To find the cheapest path using at most `k` edges, run exactly `k` passes with a snapshot for each pass.

## How it works
Every pass relaxes *every* edge once.

The interactive walkthrough keeps the note's edge order: `B→D (1), A→B (4), A→C (1), C→B (-2)`. Watch the negative edge replace the first route to B, then see that a later pass is needed before D can use B's lower distance.

```freeform
const nodes = { A: [85, 150], B: [315, 65], C: [315, 235], D: [530, 65] };
const edges = [
    { key: "B-D", from: "B", to: "D", weight: "1", path: "M343 65 L502 65", lx: 423, ly: 50 },
    { key: "A-B", from: "A", to: "B", weight: "4", path: "M112 140 L288 75", lx: 195, ly: 91 },
    { key: "A-C", from: "A", to: "C", weight: "1", path: "M112 160 L288 225", lx: 195, ly: 209 },
    { key: "C-B", from: "C", to: "B", weight: "-2", path: "M315 207 L315 93", lx: 338, ly: 154 },
];

const steps = [
    { pass: "Start", edge: null, dist: { A: 0, B: "INF", C: "INF", D: "INF" }, tree: [], changed: "not started", line: 1, action: "Set A to 0 and every other distance to infinity." },
    { pass: "1 of 3", edge: "B-D", dist: { A: 0, B: "INF", C: "INF", D: "INF" }, tree: [], changed: "no, so far", line: 3, action: "Check B -> D while B is unreachable, so D stays at infinity." },
    { pass: "1 of 3", edge: "A-B", dist: { A: 0, B: 4, C: "INF", D: "INF" }, tree: ["A-B"], changed: "yes", line: 3, action: "Relax A -> B to set B to 0 + 4 = 4." },
    { pass: "1 of 3", edge: "A-C", dist: { A: 0, B: 4, C: 1, D: "INF" }, tree: ["A-B", "A-C"], changed: "yes", line: 3, action: "Relax A -> C to set C to 0 + 1 = 1." },
    { pass: "1 of 3", edge: "C-B", dist: { A: 0, B: -1, C: 1, D: "INF" }, tree: ["A-C", "C-B"], changed: "yes", line: 3, action: "Relax C -> B to lower B from 4 to 1 - 2 = -1." },
    { pass: "2 of 3", edge: "B-D", dist: { A: 0, B: -1, C: 1, D: 0 }, tree: ["A-C", "C-B", "B-D"], changed: "yes", line: 3, action: "Relax B -> D to set D to -1 + 1 = 0." },
    { pass: "2 of 3", edge: "A-B", dist: { A: 0, B: -1, C: 1, D: 0 }, tree: ["A-C", "C-B", "B-D"], changed: "yes", line: 4, action: "Keep B at -1 because the route through A costs 4." },
    { pass: "2 of 3", edge: "A-C", dist: { A: 0, B: -1, C: 1, D: 0 }, tree: ["A-C", "C-B", "B-D"], changed: "yes", line: 4, action: "Keep C at 1 because the route through A also costs 1." },
    { pass: "2 of 3", edge: "C-B", dist: { A: 0, B: -1, C: 1, D: 0 }, tree: ["A-C", "C-B", "B-D"], changed: "yes", line: 4, action: "Keep B at -1 because the route through C also costs -1." },
    { pass: "3 of 3", edge: null, dist: { A: 0, B: -1, C: 1, D: 0 }, tree: ["A-C", "C-B", "B-D"], changed: "no", line: 4, action: "Check all four edges and confirm that every saved distance stays unchanged." },
    { pass: "Cycle check", edge: null, dist: { A: 0, B: -1, C: 1, D: 0 }, tree: ["A-C", "C-B", "B-D"], changed: "no", line: 5, action: "One extra pass finds no improvement, so no reachable negative cycle exists." },
];

const root = document.createElement("section");
root.style.cssText = "font:14px system-ui;max-width:760px;border:1px solid var(--background-modifier-border);border-radius:10px;padding:12px;background:var(--background-primary);color:var(--text-normal);";
root.innerHTML = `
<style>
  .bf-graph{display:block;width:100%;height:auto}.bf-state{display:grid;grid-template-columns:2fr 1fr;gap:8px;margin-top:8px}.bf-panel{padding:8px;border:1px solid var(--background-modifier-border);border-radius:8px}.bf-code{margin-top:8px;padding:8px 10px;border-radius:8px;background:var(--background-secondary);font:13px/1.55 var(--font-monospace);overflow-x:auto}.bf-controls{display:flex;align-items:center;gap:8px;flex-wrap:wrap;margin-top:10px}.bf-controls button,.bf-controls select{min-height:44px;font-size:14px;touch-action:manipulation}.bf-controls button{padding:6px 12px}
  @media(max-width:520px){.bf-state{grid-template-columns:1fr}.bf-controls button{flex:1 1 72px}.bf-speed{display:flex;align-items:center;gap:8px;flex:1 1 100%}.bf-speed select{flex:1}.bf-counter{width:100%;margin-left:0!important;text-align:center}.bf-code{font-size:12px}}
</style>
<svg class="bf-graph" viewBox="0 0 620 300" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Interactive Bellman-Ford graph">
  <defs>
    <marker id="bf-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0L10 5L0 10z" fill="#6c757d"></path></marker>
    <marker id="bf-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0L10 5L0 10z" fill="#2471a3"></path></marker>
    <marker id="bf-orange" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0L10 5L0 10z" fill="#d35400"></path></marker>
  </defs>
  <g data-layer="edges"></g><g data-layer="labels"></g><g data-layer="nodes"></g>
</svg>
<div style="display:flex;gap:12px;flex-wrap:wrap;font-size:12px;margin-bottom:8px"><span>Orange = edge being checked</span><span>Blue = edge used by a saved shortest path</span><span>Gray = other edge</span></div>
<div data-role="status" aria-live="polite" style="min-height:22px;padding:9px 10px;border-radius:8px;background:var(--background-secondary)"></div>
<div class="bf-state">
  <div class="bf-panel"><strong>Distances from A</strong><div data-role="dist" style="margin-top:4px;font-family:var(--font-monospace)"></div></div>
  <div class="bf-panel"><strong>Pass state</strong><div data-role="pass" style="margin-top:4px;font-family:var(--font-monospace)"></div><div data-role="changed" style="margin-top:4px;font-family:var(--font-monospace)"></div></div>
</div>
<div class="bf-code" data-role="code">
  <div data-line="1">1&nbsp; dist[source] = 0; others = infinity</div>
  <div data-line="2">2&nbsp; repeat V - 1 passes</div>
  <div data-line="3">3&nbsp;&nbsp;&nbsp; if dist[u] + w &lt; dist[v]: update v</div>
  <div data-line="4">4&nbsp;&nbsp;&nbsp; otherwise keep the saved distance</div>
  <div data-line="5">5&nbsp; one more improvement means a negative cycle</div>
</div>
<div class="bf-controls">
  <button data-action="previous">Previous</button><button data-action="play">Play</button><button data-action="next">Next</button><button data-action="reset">Reset</button>
  <label class="bf-speed" style="margin-left:8px">Speed <select data-action="speed"><option value="1200">Slow</option><option value="700" selected>Normal</option><option value="350">Fast</option></select></label>
  <strong class="bf-counter" data-role="counter" style="margin-left:auto"></strong>
</div>`;

const edgeLayer = root.querySelector('[data-layer="edges"]'), labelLayer = root.querySelector('[data-layer="labels"]'), nodeLayer = root.querySelector('[data-layer="nodes"]');
for (const edge of edges) {
    const path = document.createElementNS("http://www.w3.org/2000/svg", "path");
    path.setAttribute("d", edge.path); path.setAttribute("data-edge", edge.key); path.setAttribute("fill", "none"); edgeLayer.appendChild(path);
    const label = document.createElementNS("http://www.w3.org/2000/svg", "text");
    label.setAttribute("x", edge.lx); label.setAttribute("y", edge.ly); label.setAttribute("text-anchor", "middle"); label.setAttribute("font-size", "16"); label.setAttribute("font-weight", "700"); label.setAttribute("paint-order", "stroke"); label.setAttribute("stroke", "#ffffff"); label.setAttribute("stroke-width", "5"); label.setAttribute("fill", "currentColor"); label.textContent = edge.weight; labelLayer.appendChild(label);
}
for (const [name, [x, y]] of Object.entries(nodes)) {
    const group = document.createElementNS("http://www.w3.org/2000/svg", "g"); group.setAttribute("data-node", name); group.setAttribute("transform", `translate(${x} ${y})`);
    const distanceX = name === "B" ? -55 : 0;
    const distanceY = name === "B" ? -48 : 49;
    group.innerHTML = `<circle r="28" fill="#d5d8dc" stroke="#566573" stroke-width="3"></circle><text text-anchor="middle" dominant-baseline="central" font-size="20" font-weight="700" fill="#17202a">${name}</text><text data-role="distance" x="${distanceX}" y="${distanceY}" text-anchor="middle" font-size="16" font-weight="700" paint-order="stroke" stroke="#ffffff" stroke-width="5" fill="currentColor"></text>`; nodeLayer.appendChild(group);
}

const svg = root.querySelector("svg");
let index = 0, timer = null;
function stopPlaying() { if (timer !== null) clearInterval(timer); timer = null; root.querySelector('[data-action="play"]').textContent = "Play"; }
function render() {
    const step = steps[index], tree = new Set(step.tree);
    svg.setAttribute("aria-label", `Bellman-Ford step ${index + 1} of ${steps.length}: ${step.action}`);
    for (const group of nodeLayer.querySelectorAll('g[data-node]')) {
        const name = group.getAttribute("data-node"); group.querySelector('[data-role="distance"]').textContent = `dist=${step.dist[name]}`;
        const active = step.edge && step.edge.endsWith(`-${name}`); const circle = group.querySelector("circle");
        circle.setAttribute("fill", active ? "#f8c471" : step.dist[name] !== "INF" ? "#82e0aa" : "#d5d8dc"); circle.setAttribute("stroke", active ? "#b03a2e" : step.dist[name] !== "INF" ? "#1e8449" : "#566573"); circle.setAttribute("stroke-width", active ? "5" : "3");
    }
    for (const path of edgeLayer.querySelectorAll('path[data-edge]')) {
        const key = path.getAttribute("data-edge"), current = key === step.edge, saved = tree.has(key);
        path.setAttribute("stroke", current ? "#d35400" : saved ? "#2471a3" : "#6c757d"); path.setAttribute("stroke-width", current ? "5" : saved ? "3.5" : "2.5"); path.setAttribute("marker-end", `url(#${current ? "bf-orange" : saved ? "bf-blue" : "bf-gray"})`);
    }
    root.querySelector('[data-role="status"]').textContent = step.action;
    root.querySelector('[data-role="dist"]').textContent = `A:${step.dist.A}  B:${step.dist.B}  C:${step.dist.C}  D:${step.dist.D}`;
    root.querySelector('[data-role="pass"]').textContent = `Pass: ${step.pass}`; root.querySelector('[data-role="changed"]').textContent = `Changed: ${step.changed}`;
    for (const line of root.querySelectorAll('[data-role="code"] [data-line]')) {
        const active = Number(line.getAttribute("data-line")) === step.line;
        line.style.cssText = active ? "background:#f8c471;color:#17202a;font-weight:700" : "";
        if (active) line.setAttribute("aria-current", "step"); else line.removeAttribute("aria-current");
    }
    root.querySelector('[data-role="counter"]').textContent = `Step ${index + 1} / ${steps.length}`; root.querySelector('[data-action="previous"]').disabled = index === 0; root.querySelector('[data-action="next"]').disabled = index === steps.length - 1;
}
root.querySelector('[data-action="previous"]').addEventListener("click", () => { stopPlaying(); index = Math.max(0, index - 1); render(); });
root.querySelector('[data-action="next"]').addEventListener("click", () => { stopPlaying(); index = Math.min(steps.length - 1, index + 1); render(); });
root.querySelector('[data-action="reset"]').addEventListener("click", () => { stopPlaying(); index = 0; render(); });
root.querySelector('[data-action="play"]').addEventListener("click", () => { if (timer !== null) return stopPlaying(); if (index === steps.length - 1) index = 0; root.querySelector('[data-action="play"]').textContent = "Pause"; timer = setInterval(() => { if (index === steps.length - 1) return stopPlaying(); index += 1; render(); }, Number(root.querySelector('[data-action="speed"]').value)); render(); });
root.querySelector('[data-action="speed"]').addEventListener("change", () => { if (timer === null) return; stopPlaying(); root.querySelector('[data-action="play"]').click(); });
render();
display(root);
```

The final distances are `A:0, B:-1, C:1, D:0`. One more pass with no improvement confirms that this example has no reachable negative cycle.

## Why it works

- A shortest path never needs to repeat a node unless repeating a cycle makes the path cheaper.
- If there is no reachable negative cycle, the repeated part can be removed. A shortest path can therefore be a **simple path** that visits each node at most once.
- A simple path visits at most `V` nodes, so it uses at most `V - 1` edges.
- After pass `k`, Bellman-Ford guarantees the correct cost for every shortest path that uses at most `k` edges. An update may travel farther in one pass when the edge order happens to help, but the algorithm never depends on that shortcut.
- After `V - 1` passes, every possible simple shortest path has been covered.
- If one more pass still lowers a distance, the cheaper route must repeat a node. The repeated section is a reachable negative cycle.
- Going around a negative cycle keeps lowering the path cost, so no finite shortest distance exists for nodes reachable from that cycle.

This differs from [[Dijkstra]]. Dijkstra can finalize a node early only when every edge weight is non-negative. Bellman-Ford does not finalize nodes early, so negative edges are safe.

### Why an extra pass finds a negative cycle

In this example, `B -> C -> B` has a total weight of `-2 + 1 = -1`. Every trip around the cycle lowers the saved distances by `1`.

```freeform
const ncNodes = { A: [85, 125], B: [300, 55], C: [300, 195] };
const ncEdges = [
    { key: "A-B", from: "A", to: "B", weight: "1", path: "M111 116 L273 66", lx: 188, ly: 73 },
    { key: "B-C", from: "B", to: "C", weight: "-2", path: "M288 81 C260 112 260 138 288 169", lx: 247, ly: 128 },
    { key: "C-B", from: "C", to: "B", weight: "1", path: "M312 169 C340 138 340 112 312 81", lx: 353, ly: 128 },
];

const ncSteps = [
    { label: "Start", dist: { A: 0, B: "INF", C: "INF" }, cycle: false, action: "Only the source is reachable." },
    { label: "Pass 1", dist: { A: 0, B: 0, C: -1 }, cycle: true, action: "The first trip around B -> C -> B lowers B from 1 to 0." },
    { label: "Pass 2", dist: { A: 0, B: -1, C: -2 }, cycle: true, action: "V - 1 passes are complete, but another trip lowers both cycle nodes again." },
    { label: "Extra pass", dist: { A: 0, B: -2, C: -3 }, cycle: true, negative: true, action: "The extra pass still improves a distance, which proves a reachable negative cycle exists." },
    { label: "Repeat again", dist: { A: 0, B: -3, C: -4 }, cycle: true, negative: true, action: "The cost keeps falling. There is no finite shortest distance for B or C." },
];

const ncRoot = document.createElement("section");
ncRoot.style.cssText = "font:14px system-ui;width:min(100%,420px);box-sizing:border-box;border:1px solid #c8cdd2;border-radius:10px;padding:12px;background:#ffffff;color:#17202a;";
ncRoot.innerHTML = `
<style>
  .nc-graph{display:block;width:100%;height:auto}.nc-grid{display:grid;grid-template-columns:1fr 1fr;gap:8px;margin-top:8px}.nc-panel{padding:9px 10px;border:1px solid #d5d8dc;border-radius:8px;background:#f8f9f9}.nc-controls{display:flex;align-items:center;gap:8px;flex-wrap:wrap;margin-top:10px}.nc-controls button,.nc-controls select{min-height:44px;font-size:14px}.nc-controls button{padding:6px 12px}
  @media(max-width:520px){.nc-grid{grid-template-columns:1fr}.nc-controls button{flex:1 1 72px}.nc-speed{display:flex;align-items:center;gap:8px;flex:1 1 100%}.nc-speed select{flex:1}.nc-counter{width:100%;margin-left:0!important;text-align:center}}
</style>
<svg class="nc-graph" viewBox="0 0 410 255" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Negative cycle walkthrough">
  <defs>
    <marker id="nc-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0L10 5L0 10z" fill="#6c757d"></path></marker>
    <marker id="nc-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0L10 5L0 10z" fill="#c0392b"></path></marker>
    <marker id="nc-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0L10 5L0 10z" fill="#2471a3"></path></marker>
  </defs>
  <g data-layer="edges"></g><g data-layer="labels"></g><g data-layer="nodes"></g>
</svg>
<div style="display:flex;gap:12px;flex-wrap:wrap;font-size:12px;margin-bottom:8px"><span>Blue = entry edge</span><span>Red = negative cycle</span></div>
<div data-role="status" aria-live="polite" style="min-height:22px;padding:9px 10px;border-radius:8px;background:#eef2f3"></div>
<div class="nc-grid">
  <div class="nc-panel"><strong>Distances from A</strong><div data-role="dist" style="margin-top:5px;font-family:monospace"></div></div>
  <div class="nc-panel"><strong>Cycle cost</strong><div style="margin-top:5px;font-family:monospace">-2 + 1 = -1</div></div>
</div>
<div class="nc-controls">
  <button data-action="previous">Previous</button><button data-action="play">Play</button><button data-action="next">Next</button><button data-action="reset">Reset</button>
  <label class="nc-speed" style="margin-left:8px">Speed <select data-action="speed"><option value="1200">Slow</option><option value="700" selected>Normal</option><option value="350">Fast</option></select></label>
  <strong class="nc-counter" data-role="counter" style="margin-left:auto"></strong>
</div>`;

const ncEdgeLayer = ncRoot.querySelector('[data-layer="edges"]');
const ncLabelLayer = ncRoot.querySelector('[data-layer="labels"]');
const ncNodeLayer = ncRoot.querySelector('[data-layer="nodes"]');
for (const edge of ncEdges) {
    const path = document.createElementNS("http://www.w3.org/2000/svg", "path");
    path.setAttribute("d", edge.path); path.setAttribute("data-edge", edge.key); path.setAttribute("fill", "none"); ncEdgeLayer.appendChild(path);
    const label = document.createElementNS("http://www.w3.org/2000/svg", "text");
    label.setAttribute("x", edge.lx); label.setAttribute("y", edge.ly); label.setAttribute("text-anchor", "middle"); label.setAttribute("font-size", "16"); label.setAttribute("font-weight", "700"); label.setAttribute("paint-order", "stroke"); label.setAttribute("stroke", "#ffffff"); label.setAttribute("stroke-width", "5"); label.setAttribute("fill", "#17202a"); label.textContent = edge.weight; ncLabelLayer.appendChild(label);
}
for (const [name, [x, y]] of Object.entries(ncNodes)) {
    const group = document.createElementNS("http://www.w3.org/2000/svg", "g");
    group.setAttribute("data-node", name); group.setAttribute("transform", `translate(${x} ${y})`);
    group.innerHTML = `<circle r="28" fill="#d5d8dc" stroke="#566573" stroke-width="3"></circle><text text-anchor="middle" dominant-baseline="central" font-size="20" font-weight="700" fill="#17202a">${name}</text><text data-role="distance" y="49" text-anchor="middle" font-size="16" font-weight="700" paint-order="stroke" stroke="#ffffff" stroke-width="5" fill="#17202a"></text>`;
    ncNodeLayer.appendChild(group);
}

const ncSvg = ncRoot.querySelector("svg");
let ncIndex = 0, ncTimer = null;
function ncStop() { if (ncTimer !== null) clearInterval(ncTimer); ncTimer = null; ncRoot.querySelector('[data-action="play"]').textContent = "Play"; }
function ncRender() {
    const step = ncSteps[ncIndex];
    ncSvg.setAttribute("aria-label", `Negative cycle step ${ncIndex + 1} of ${ncSteps.length}: ${step.action}`);
    for (const path of ncEdgeLayer.querySelectorAll('path[data-edge]')) {
        const key = path.getAttribute("data-edge");
        const inCycle = key === "B-C" || key === "C-B";
        const color = inCycle && step.cycle ? "#c0392b" : key === "A-B" && step.dist.B !== "INF" ? "#2471a3" : "#6c757d";
        path.setAttribute("stroke", color); path.setAttribute("stroke-width", inCycle && step.cycle ? "5" : "3"); path.setAttribute("marker-end", `url(#${inCycle && step.cycle ? "nc-red" : key === "A-B" && step.dist.B !== "INF" ? "nc-blue" : "nc-gray"})`);
    }
    for (const group of ncNodeLayer.querySelectorAll('g[data-node]')) {
        const name = group.getAttribute("data-node");
        const onCycle = name === "B" || name === "C";
        group.querySelector('[data-role="distance"]').textContent = `dist=${step.dist[name]}`;
        group.querySelector("circle").setAttribute("fill", onCycle && step.negative ? "#f5b7b1" : step.dist[name] !== "INF" ? "#aed6f1" : "#d5d8dc");
        group.querySelector("circle").setAttribute("stroke", onCycle && step.cycle ? "#c0392b" : step.dist[name] !== "INF" ? "#2471a3" : "#566573");
    }
    ncRoot.querySelector('[data-role="status"]').textContent = `${step.label}: ${step.action}`;
    ncRoot.querySelector('[data-role="dist"]').textContent = `A:${step.dist.A}  B:${step.dist.B}  C:${step.dist.C}`;
    ncRoot.querySelector('[data-role="counter"]').textContent = `Step ${ncIndex + 1} / ${ncSteps.length}`;
    ncRoot.querySelector('[data-action="previous"]').disabled = ncIndex === 0;
    ncRoot.querySelector('[data-action="next"]').disabled = ncIndex === ncSteps.length - 1;
}
ncRoot.querySelector('[data-action="previous"]').addEventListener("click", () => { ncStop(); ncIndex = Math.max(0, ncIndex - 1); ncRender(); });
ncRoot.querySelector('[data-action="next"]').addEventListener("click", () => { ncStop(); ncIndex = Math.min(ncSteps.length - 1, ncIndex + 1); ncRender(); });
ncRoot.querySelector('[data-action="reset"]').addEventListener("click", () => { ncStop(); ncIndex = 0; ncRender(); });
ncRoot.querySelector('[data-action="play"]').addEventListener("click", () => { if (ncTimer !== null) return ncStop(); if (ncIndex === ncSteps.length - 1) ncIndex = 0; ncRoot.querySelector('[data-action="play"]').textContent = "Pause"; ncTimer = setInterval(() => { if (ncIndex === ncSteps.length - 1) return ncStop(); ncIndex += 1; ncRender(); }, Number(ncRoot.querySelector('[data-action="speed"]').value)); ncRender(); });
ncRoot.querySelector('[data-action="speed"]').addEventListener("change", () => { if (ncTimer === null) return; ncStop(); ncRoot.querySelector('[data-action="play"]').click(); });
ncRender();
display(ncRoot);
```

## Template
```python
def bellman_ford(n, edges, src):
    # edges = [(u, v, weight), ...]
    dist = [float('inf')] * n
    dist[src] = 0

    for _ in range(n - 1):              # V−1 passes
        for u, v, w in edges:
            if dist[u] != float('inf') and dist[u] + w < dist[v]:
                dist[v] = dist[u] + w

    # Optional: detect negative cycle
    for u, v, w in edges:
        if dist[u] + w < dist[v]:
            return None                  # negative cycle

    return dist
```

### K-hop variant

A pass normally updates `dist` in place. That is safe for regular Bellman-Ford, but it can break an edge limit because a later edge may use a value saved earlier in the same pass.

Suppose a flight may use **at most 1 stop**, which means at most 2 edges. Process these edges in order:

1. `A -> B` costs `1`
2. `B -> C` costs `1`
3. `C -> D` costs `1`
4. `A -> D` costs `10`

Without a snapshot, the first pass can chain all three cheap edges:

- `A -> B` sets `B = 1`.
- `B -> C` immediately reads the new `B` and sets `C = 2`.
- `C -> D` immediately reads the new `C` and sets `D = 3`.
- The algorithm accepts a route with 2 stops even though the limit is 1 stop.

With a snapshot, every edge reads distances from the **start of the pass**:

- Pass 1 finds routes that use at most 1 edge.
- Pass 2 finds routes that use at most 2 edges.
- The illegal three-edge route cannot appear. The best valid route to `D` is the direct edge with cost `10`.

```freeform
const khNames = ["A", "B", "C", "D"];
const khX = { A: 105, B: 265, C: 425, D: 585 };
const khRows = [
    { key: "bad", label: "No snapshot", y: 95 },
    { key: "good", label: "With snapshot", y: 255 },
];
const khEdges = [
    { key: "A-B", from: "A", to: "B", weight: "1" },
    { key: "B-C", from: "B", to: "C", weight: "1" },
    { key: "C-D", from: "C", to: "D", weight: "1" },
    { key: "A-D", from: "A", to: "D", weight: "10", direct: true },
];
const khSteps = [
    {
        label: "Start",
        bad: { A: 0, B: "INF", C: "INF", D: "INF" },
        good: { A: 0, B: "INF", C: "INF", D: "INF" },
        badEdges: [], goodEdges: [],
        action: "Both versions start with only A reachable."
    },
    {
        label: "After pass 1",
        bad: { A: 0, B: 1, C: 2, D: 3 },
        good: { A: 0, B: 1, C: "INF", D: 10 },
        badEdges: ["A-B", "B-C", "C-D"], goodEdges: ["A-B", "A-D"],
        action: "No snapshot chains three edges in one pass. The snapshot version allows only one-edge routes."
    },
    {
        label: "After pass 2",
        bad: { A: 0, B: 1, C: 2, D: 3 },
        good: { A: 0, B: 1, C: 2, D: 10 },
        badEdges: ["A-B", "B-C", "C-D"], goodEdges: ["A-B", "B-C", "A-D"],
        action: "The snapshot version now allows up to two edges. It can reach C through B, but not D through C."
    },
    {
        label: "Result",
        bad: { A: 0, B: 1, C: 2, D: 3 },
        good: { A: 0, B: 1, C: 2, D: 10 },
        badEdges: ["A-B", "B-C", "C-D"], goodEdges: ["A-D"],
        action: "Cost 3 is illegal because it uses two stops. The correct answer is the direct route with cost 10."
    },
];

const khRoot = document.createElement("section");
khRoot.style.cssText = "font:14px system-ui;max-width:760px;border:1px solid #c8cdd2;border-radius:10px;padding:12px;background:#ffffff;color:#17202a;";
khRoot.innerHTML = `
<style>
  .kh-graph{display:block;width:100%;height:auto}.kh-results{display:grid;grid-template-columns:1fr 1fr;gap:8px;margin-top:8px}.kh-panel{padding:9px 10px;border:1px solid #d5d8dc;border-radius:8px;background:#f8f9f9}.kh-controls{display:flex;align-items:center;gap:8px;flex-wrap:wrap;margin-top:10px}.kh-controls button,.kh-controls select{min-height:44px;font-size:14px}.kh-controls button{padding:6px 12px}
  @media(max-width:520px){.kh-results{grid-template-columns:1fr}.kh-controls button{flex:1 1 72px}.kh-speed{display:flex;align-items:center;gap:8px;flex:1 1 100%}.kh-speed select{flex:1}.kh-counter{width:100%;margin-left:0!important;text-align:center}}
</style>
<svg class="kh-graph" viewBox="0 0 690 325" preserveAspectRatio="xMidYMid meet" role="img" aria-label="K-hop snapshot comparison">
  <defs>
    <marker id="kh-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0L10 5L0 10z" fill="#6c757d"></path></marker>
    <marker id="kh-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0L10 5L0 10z" fill="#c0392b"></path></marker>
    <marker id="kh-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0L10 5L0 10z" fill="#2471a3"></path></marker>
  </defs>
  <g data-layer="edges"></g><g data-layer="labels"></g><g data-layer="nodes"></g>
</svg>
<div style="display:flex;gap:12px;flex-wrap:wrap;font-size:12px;margin-bottom:8px"><span>Red = illegal route</span><span>Blue = routes allowed so far</span><span>Gray = unused edge</span></div>
<div data-role="status" aria-live="polite" style="min-height:22px;padding:9px 10px;border-radius:8px;background:#eef2f3"></div>
<div class="kh-results">
  <div class="kh-panel"><strong>No snapshot</strong><div data-role="bad-dist" style="margin-top:5px;font-family:monospace"></div></div>
  <div class="kh-panel"><strong>With snapshot</strong><div data-role="good-dist" style="margin-top:5px;font-family:monospace"></div></div>
</div>
<div class="kh-controls">
  <button data-action="previous">Previous</button><button data-action="play">Play</button><button data-action="next">Next</button><button data-action="reset">Reset</button>
  <label class="kh-speed" style="margin-left:8px">Speed <select data-action="speed"><option value="1200">Slow</option><option value="700" selected>Normal</option><option value="350">Fast</option></select></label>
  <strong class="kh-counter" data-role="counter" style="margin-left:auto"></strong>
</div>`;

const khEdgeLayer = khRoot.querySelector('[data-layer="edges"]');
const khLabelLayer = khRoot.querySelector('[data-layer="labels"]');
const khNodeLayer = khRoot.querySelector('[data-layer="nodes"]');
for (const row of khRows) {
    const title = document.createElementNS("http://www.w3.org/2000/svg", "text");
    title.setAttribute("x", "12"); title.setAttribute("y", row.y - 52); title.setAttribute("font-size", "16"); title.setAttribute("font-weight", "700"); title.setAttribute("fill", "#17202a"); title.textContent = row.label; khLabelLayer.appendChild(title);
    for (const edge of khEdges) {
        const path = document.createElementNS("http://www.w3.org/2000/svg", "path");
        const d = edge.direct
            ? `M${khX.A + 24} ${row.y - 14} C250 ${row.y - 82} 440 ${row.y - 82} ${khX.D - 24} ${row.y - 14}`
            : `M${khX[edge.from] + 28} ${row.y} L${khX[edge.to] - 28} ${row.y}`;
        path.setAttribute("d", d); path.setAttribute("data-row", row.key); path.setAttribute("data-edge", edge.key); path.setAttribute("fill", "none"); khEdgeLayer.appendChild(path);
        const label = document.createElementNS("http://www.w3.org/2000/svg", "text");
        const lx = edge.direct ? 345 : (khX[edge.from] + khX[edge.to]) / 2;
        const ly = edge.direct ? row.y - 66 : row.y - 10;
        label.setAttribute("x", lx); label.setAttribute("y", ly); label.setAttribute("text-anchor", "middle"); label.setAttribute("font-size", "14"); label.setAttribute("font-weight", "700"); label.setAttribute("paint-order", "stroke"); label.setAttribute("stroke", "#ffffff"); label.setAttribute("stroke-width", "5"); label.setAttribute("fill", "#17202a"); label.textContent = edge.weight; khLabelLayer.appendChild(label);
    }
    for (const name of khNames) {
        const group = document.createElementNS("http://www.w3.org/2000/svg", "g");
        group.setAttribute("data-row", row.key); group.setAttribute("data-node", name); group.setAttribute("transform", `translate(${khX[name]} ${row.y})`);
        group.innerHTML = `<circle r="28" fill="#d5d8dc" stroke="#566573" stroke-width="3"></circle><text text-anchor="middle" dominant-baseline="central" font-size="20" font-weight="700" fill="#17202a">${name}</text><text data-role="distance" y="49" text-anchor="middle" font-size="16" font-weight="700" paint-order="stroke" stroke="#ffffff" stroke-width="5" fill="#17202a"></text>`;
        khNodeLayer.appendChild(group);
    }
}

const khSvg = khRoot.querySelector("svg");
let khIndex = 0, khTimer = null;
function khStop() { if (khTimer !== null) clearInterval(khTimer); khTimer = null; khRoot.querySelector('[data-action="play"]').textContent = "Play"; }
function khFormat(dist) { return `A:${dist.A}  B:${dist.B}  C:${dist.C}  D:${dist.D}`; }
function khRender() {
    const step = khSteps[khIndex];
    khSvg.setAttribute("aria-label", `K-hop step ${khIndex + 1} of ${khSteps.length}: ${step.action}`);
    for (const path of khEdgeLayer.querySelectorAll('path[data-edge]')) {
        const row = path.getAttribute("data-row");
        const key = path.getAttribute("data-edge");
        const active = new Set(step[`${row}Edges`]).has(key);
        const illegal = row === "bad" && active;
        const color = active ? illegal ? "#c0392b" : "#2471a3" : "#6c757d";
        path.setAttribute("stroke", color); path.setAttribute("stroke-width", active ? "4.5" : "2.5"); path.setAttribute("marker-end", `url(#${active ? illegal ? "kh-red" : "kh-blue" : "kh-gray"})`);
    }
    for (const group of khNodeLayer.querySelectorAll('g[data-node]')) {
        const row = group.getAttribute("data-row");
        const name = group.getAttribute("data-node");
        const value = step[row][name];
        group.querySelector('[data-role="distance"]').textContent = `dist=${value}`;
        const illegal = row === "bad" && khIndex > 0 && name !== "A";
        group.querySelector("circle").setAttribute("fill", value === "INF" ? "#d5d8dc" : illegal ? "#f5b7b1" : "#aed6f1");
        group.querySelector("circle").setAttribute("stroke", value === "INF" ? "#566573" : illegal ? "#c0392b" : "#2471a3");
    }
    khRoot.querySelector('[data-role="status"]').textContent = `${step.label}: ${step.action}`;
    khRoot.querySelector('[data-role="bad-dist"]').textContent = khFormat(step.bad);
    khRoot.querySelector('[data-role="good-dist"]').textContent = khFormat(step.good);
    khRoot.querySelector('[data-role="counter"]').textContent = `Step ${khIndex + 1} / ${khSteps.length}`;
    khRoot.querySelector('[data-action="previous"]').disabled = khIndex === 0;
    khRoot.querySelector('[data-action="next"]').disabled = khIndex === khSteps.length - 1;
}
khRoot.querySelector('[data-action="previous"]').addEventListener("click", () => { khStop(); khIndex = Math.max(0, khIndex - 1); khRender(); });
khRoot.querySelector('[data-action="next"]').addEventListener("click", () => { khStop(); khIndex = Math.min(khSteps.length - 1, khIndex + 1); khRender(); });
khRoot.querySelector('[data-action="reset"]').addEventListener("click", () => { khStop(); khIndex = 0; khRender(); });
khRoot.querySelector('[data-action="play"]').addEventListener("click", () => { if (khTimer !== null) return khStop(); if (khIndex === khSteps.length - 1) khIndex = 0; khRoot.querySelector('[data-action="play"]').textContent = "Pause"; khTimer = setInterval(() => { if (khIndex === khSteps.length - 1) return khStop(); khIndex += 1; khRender(); }, Number(khRoot.querySelector('[data-action="speed"]').value)); khRender(); });
khRoot.querySelector('[data-action="speed"]').addEventListener("change", () => { if (khTimer === null) return; khStop(); khRoot.querySelector('[data-action="play"]').click(); });
khRender();
display(khRoot);
```

The code keeps two arrays for each pass:

- `dist` contains answers from the previous pass and is read-only during the current pass.
- `next_dist` receives this pass's improvements.
- At the end of the pass, `next_dist` becomes the new `dist`.

```python
for _ in range(k + 1):          # k stops means at most k + 1 edges
    next_dist = dist[:]
    for u, v, w in edges:
        if dist[u] != float('inf') and dist[u] + w < next_dist[v]:
            next_dist[v] = dist[u] + w
    dist = next_dist
```

If the limit is given as **at most `k` edges** instead of `k` stops, run exactly `k` passes.

## Complexity

- **Time:** `O(V × E)`. Bellman-Ford runs `V - 1` passes and checks every edge during each pass.
- **Space:** `O(V)` for the distance array.

## Common pitfalls
- **Stopping before all required passes:** Distances may look stable before a later pass finds another improvement. An early exit is safe only when a complete pass makes no updates.
- **Using Bellman-Ford when Dijkstra is enough:** Bellman-Ford also works on non-negative graphs, but `O(V × E)` can be much slower than `O((V + E) log V)` on large graphs.
- **Forgetting the K-hop snapshot:** A single pass may use a distance that was just updated, which can silently allow more than `k` edges.
- **Relaxing from infinity:** Check `dist[u] != float('inf')` before adding the edge weight.

## vs. Dijkstra

| | Dijkstra | Bellman-Ford |
|---|---|---|
| Negative weights | wrong (greedy fails) | correct |
| Negative cycles | undetectable | detected on V-th pass |
| Speed | O((V+E) log V) | O(V × E) |
| k-hop limit | awkward | natural with `k + 1` passes |

## NeetCode examples
- [[06.CheapestFlightsWithinKStops|CheapestFlightsWithinKStops]]: K-hop Bellman-Ford with a snapshot

## Full guide
[[Job Search/Neetcode/01. Questions/12. Advanced Graphs/0.AdvancedGraphsGuide|Advanced Graphs Guide]]
