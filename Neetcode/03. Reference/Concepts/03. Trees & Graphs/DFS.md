---
type: concept
tags: ["concept"]
---

# DFS

**TL;DR:** Explore as deep as possible down one path before backtracking — recursion (or an explicit stack) remembers the way back for free.

## When to reach for it
- "Does a path exist?" or "explore this entire region" — connected components, flood-fill.
- Tree problems where a parent needs a combined result from its children (height, diameter, LCA, path sums).
- Cycle detection in a directed graph (needs 3-color state, not a plain visited set).
- Any prompt implying "explore every branch fully before moving on" — as opposed to "find the shortest way," which is [[BFS]]'s job.

## How it works
Recursion (or an explicit stack) commits to one neighbor completely — descending as far as possible — and only tries the next neighbor after that branch is fully exhausted. Trace it starting at A:

The interactive graph below runs locally with the Freeform plugin. Use **Previous** and **Next** to follow the recursive call stack, including the cycle at `D -> B` and the later cross edges into finished branches.

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
    { from: "A", to: "B", path: "M91 147 L194 83" },
    { from: "A", to: "C", path: "M91 173 L194 237" },
    { from: "B", to: "D", path: "M240 70 L355 70" },
    { from: "C", to: "E", path: "M240 250 L355 250" },
    { from: "C", to: "F", path: "M239 243 L521 167" },
    { from: "D", to: "B", path: "M362 52 Q298 0 233 52" },
    { from: "D", to: "F", path: "M402 82 L523 148" },
    { from: "E", to: "D", path: "M380 225 L380 95" },
];

const steps = [
    { active: ["A"], finished: [], edge: null, edgeType: null, tree: [], skipped: [], order: ["A"], line: 1, action: "Visit A and mark it active." },
    { active: ["A", "B"], finished: [], edge: "A-B", edgeType: "tree", tree: ["A-B"], skipped: [], order: ["A", "B"], line: 3, action: "Traverse A -> B because B is unvisited." },
    { active: ["A", "B", "D"], finished: [], edge: "B-D", edgeType: "tree", tree: ["A-B", "B-D"], skipped: [], order: ["A", "B", "D"], line: 3, action: "Traverse B -> D because D is unvisited." },
    { active: ["A", "B", "D"], finished: [], edge: "D-B", edgeType: "skip", tree: ["A-B", "B-D"], skipped: ["D-B"], order: ["A", "B", "D"], line: 4, action: "Skip D -> B because B is already active." },
    { active: ["A", "B", "D", "F"], finished: [], edge: "D-F", edgeType: "tree", tree: ["A-B", "B-D", "D-F"], skipped: ["D-B"], order: ["A", "B", "D", "F"], line: 3, action: "Traverse D -> F because F is unvisited." },
    { active: ["A"], finished: ["F", "D", "B"], edge: null, edgeType: null, tree: ["A-B", "B-D", "D-F"], skipped: ["D-B"], order: ["A", "B", "D", "F"], line: 5, action: "Finish F, D, and B, then return to A." },
    { active: ["A", "C"], finished: ["F", "D", "B"], edge: "A-C", edgeType: "tree", tree: ["A-B", "B-D", "D-F", "A-C"], skipped: ["D-B"], order: ["A", "B", "D", "F", "C"], line: 3, action: "Traverse A -> C because C is unvisited." },
    { active: ["A", "C", "E"], finished: ["F", "D", "B"], edge: "C-E", edgeType: "tree", tree: ["A-B", "B-D", "D-F", "A-C", "C-E"], skipped: ["D-B"], order: ["A", "B", "D", "F", "C", "E"], line: 3, action: "Traverse C -> E because E is unvisited." },
    { active: ["A", "C", "E"], finished: ["F", "D", "B"], edge: "E-D", edgeType: "skip", tree: ["A-B", "B-D", "D-F", "A-C", "C-E"], skipped: ["D-B", "E-D"], order: ["A", "B", "D", "F", "C", "E"], line: 4, action: "Skip E -> D because D is already finished." },
    { active: ["A", "C"], finished: ["F", "D", "B", "E"], edge: "C-F", edgeType: "skip", tree: ["A-B", "B-D", "D-F", "A-C", "C-E"], skipped: ["D-B", "E-D", "C-F"], order: ["A", "B", "D", "F", "C", "E"], line: 4, action: "Skip C -> F because F is already finished." },
    { active: [], finished: ["A", "B", "C", "D", "E", "F"], edge: null, edgeType: null, tree: ["A-B", "B-D", "D-F", "A-C", "C-E"], skipped: ["D-B", "E-D", "C-F"], order: ["A", "B", "D", "F", "C", "E"], line: 5, action: "Finish DFS after all reachable nodes are complete." },
];

const root = document.createElement("section");
root.style.cssText = "font: 14px system-ui; max-width: 760px; border: 1px solid var(--background-modifier-border); border-radius: 10px; padding: 12px; background: var(--background-primary); color: var(--text-normal);";
root.innerHTML = `
    <style>
        .dfs-controls { display:flex; align-items:center; gap:8px; flex-wrap:wrap; margin-top:10px; }
        .dfs-controls button, .dfs-controls select { min-height:44px; font-size:14px; touch-action:manipulation; }
        .dfs-controls button { padding:6px 12px; }
        .dfs-graph-scroll { width:100%; overflow:visible; }
        .dfs-graph { width:100%; height:auto; display:block; }
        .dfs-state-grid { display:grid; grid-template-columns:1fr 1fr; gap:8px; margin-top:8px; }
        .dfs-code { overflow-x:auto; font-size:13px; }
        @media (max-width:520px) {
            .dfs-controls { gap:6px; }
            .dfs-controls button { flex:1 1 72px; min-height:44px; }
            .dfs-speed { flex:1 1 100%; display:flex; align-items:center; gap:8px; }
            .dfs-speed select { flex:1; min-height:44px; }
            .dfs-counter { width:100%; margin-left:0 !important; text-align:center; }
            .dfs-graph { width:100%; min-width:0; }
            .dfs-state-grid { grid-template-columns:1fr; }
            .dfs-legend { gap:8px !important; font-size:11px !important; }
            .dfs-code { font-size:12px; }
        }
    </style>
    <div class="dfs-graph-scroll" aria-label="Graph area">
    <svg class="dfs-graph" viewBox="0 0 620 320" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Interactive DFS graph">
        <defs>
            <marker id="arrow-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto">
                <path d="M 0 0 L 10 5 L 0 10 z" fill="#7f8c8d"></path>
            </marker>
            <marker id="arrow-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto">
                <path d="M 0 0 L 10 5 L 0 10 z" fill="#2471a3"></path>
            </marker>
            <marker id="arrow-orange" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto">
                <path d="M 0 0 L 10 5 L 0 10 z" fill="#d35400"></path>
            </marker>
            <marker id="arrow-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto">
                <path d="M 0 0 L 10 5 L 0 10 z" fill="#c0392b"></path>
            </marker>
        </defs>
        <g data-layer="edges"></g>
        <g data-layer="nodes"></g>
    </svg>
    </div>
    <div class="dfs-legend" data-role="legend" style="display:flex; gap:14px; flex-wrap:wrap; margin:0 0 8px; font-size:12px;">
        <span>Orange = current</span>
        <span>Blue = DFS-tree edge / active call</span>
        <span>Green = finished node</span>
        <span>Red dashed = skipped edge</span>
        <span>Gray = unchecked</span>
    </div>
    <div data-role="status" aria-live="polite" style="min-height:22px; padding:9px 10px; border-radius:8px; background:var(--background-secondary);"></div>
    <div class="dfs-state-grid">
        <div style="padding:8px; border:1px solid var(--background-modifier-border); border-radius:8px;">
            <strong>Call stack</strong><div data-role="stack" style="margin-top:4px; font-family:var(--font-monospace);"></div>
        </div>
        <div style="padding:8px; border:1px solid var(--background-modifier-border); border-radius:8px;">
            <strong>Visit order</strong><div data-role="order" style="margin-top:4px; font-family:var(--font-monospace);"></div>
        </div>
    </div>
    <div class="dfs-code" data-role="code" style="margin-top:8px; padding:8px 10px; border-radius:8px; background:var(--background-secondary); font-family:var(--font-monospace); line-height:1.55;">
        <div data-line="1">1&nbsp; mark u active</div>
        <div data-line="2">2&nbsp; for each v in adj[u]</div>
        <div data-line="3">3&nbsp;&nbsp;&nbsp; if v is unvisited: dfs(v)</div>
        <div data-line="4">4&nbsp;&nbsp;&nbsp; else: skip v</div>
        <div data-line="5">5&nbsp; mark u finished; return</div>
    </div>
    <div class="dfs-controls">
        <button data-action="previous" aria-label="Previous step">Previous</button>
        <button data-action="play">Play</button>
        <button data-action="next" aria-label="Next step">Next</button>
        <button data-action="reset">Reset</button>
        <label class="dfs-speed" style="margin-left:8px;">Speed
            <select data-action="speed">
                <option value="1200">Slow</option>
                <option value="700" selected>Normal</option>
                <option value="350">Fast</option>
            </select>
        </label>
        <strong class="dfs-counter" data-role="counter" style="margin-left:auto;"></strong>
    </div>
`;

const svg = root.querySelector("svg");
const edgeLayer = root.querySelector('[data-layer="edges"]');
const nodeLayer = root.querySelector('[data-layer="nodes"]');

for (const edge of edges) {
    const path = document.createElementNS("http://www.w3.org/2000/svg", "path");
    path.setAttribute("d", edge.path);
    path.setAttribute("data-edge", `${edge.from}-${edge.to}`);
    path.setAttribute("fill", "none");
    path.setAttribute("stroke", "#7f8c8d");
    path.setAttribute("stroke-width", "2.5");
    path.setAttribute("marker-end", "url(#arrow-gray)");
    edgeLayer.appendChild(path);
}

for (const [name, [x, y]] of Object.entries(nodes)) {
    const group = document.createElementNS("http://www.w3.org/2000/svg", "g");
    group.setAttribute("data-node", name);
    group.setAttribute("transform", `translate(${x} ${y})`);
    group.innerHTML = `
        <circle r="25" fill="#d5d8dc" stroke="#566573" stroke-width="3"></circle>
        <text text-anchor="middle" dominant-baseline="central" font-size="18" font-weight="700" fill="#17202a">${name}</text>
    `;
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
    svg.setAttribute("aria-label", `DFS step ${index + 1} of ${steps.length}: ${step.action}`);
    const active = new Set(step.active);
    const finished = new Set(step.finished);
    const current = step.active.length ? step.active[step.active.length - 1] : null;
    const treeEdges = new Set(step.tree);
    const skippedEdges = new Set(step.skipped);

    for (const group of nodeLayer.querySelectorAll("g[data-node]")) {
        const name = group.getAttribute("data-node");
        const circle = group.querySelector("circle");
        if (name === current) {
            circle.setAttribute("fill", "#f8c471");
            circle.setAttribute("stroke", "#b03a2e");
            circle.setAttribute("stroke-width", "5");
        } else if (active.has(name)) {
            circle.setAttribute("fill", "#85c1e9");
            circle.setAttribute("stroke", "#1f618d");
            circle.setAttribute("stroke-width", "3");
        } else if (finished.has(name)) {
            circle.setAttribute("fill", "#82e0aa");
            circle.setAttribute("stroke", "#1e8449");
            circle.setAttribute("stroke-width", "3");
        } else {
            circle.setAttribute("fill", "#d5d8dc");
            circle.setAttribute("stroke", "#566573");
            circle.setAttribute("stroke-width", "3");
        }
    }

    for (const path of edgeLayer.querySelectorAll("path[data-edge]")) {
        const key = path.getAttribute("data-edge");
        const isCurrent = key === step.edge;
        let color = "#7f8c8d";
        let width = "2.5";
        let marker = "arrow-gray";
        let dash = "";

        if (treeEdges.has(key)) {
            color = "#2471a3";
            marker = "arrow-blue";
            width = "3.5";
        }
        if (skippedEdges.has(key)) {
            color = "#c0392b";
            marker = "arrow-red";
            dash = "7 5";
        }
        if (isCurrent && step.edgeType === "tree") {
            color = "#d35400";
            marker = "arrow-orange";
            width = "5";
        }
        if (isCurrent && step.edgeType === "skip") {
            color = "#c0392b";
            marker = "arrow-red";
            width = "5";
            dash = "7 5";
        }

        path.setAttribute("stroke", color);
        path.setAttribute("stroke-width", width);
        path.setAttribute("stroke-dasharray", dash);
        path.setAttribute("marker-end", `url(#${marker})`);
    }

    root.querySelector('[data-role="status"]').textContent = step.action;
    root.querySelector('[data-role="stack"]').textContent = step.active.length
        ? step.active.join(" -> ")
        : "empty";
    root.querySelector('[data-role="order"]').textContent = step.order.join(", ");
    for (const line of root.querySelectorAll('[data-role="code"] [data-line]')) {
        const isActive = Number(line.getAttribute("data-line")) === step.line;
        line.style.background = isActive ? "#f8c471" : "transparent";
        line.style.color = isActive ? "#17202a" : "inherit";
        line.style.fontWeight = isActive ? "700" : "400";
        if (isActive) line.setAttribute("aria-current", "step");
        else line.removeAttribute("aria-current");
    }
    root.querySelector('[data-role="counter"]').textContent = `Step ${index + 1} / ${steps.length}`;
    root.querySelector('[data-action="previous"]').disabled = index === 0;
    root.querySelector('[data-action="next"]').disabled = index === steps.length - 1;
}

root.querySelector('[data-action="previous"]').addEventListener("click", () => {
    stopPlaying();
    index = Math.max(0, index - 1);
    render();
});
root.querySelector('[data-action="next"]').addEventListener("click", () => {
    stopPlaying();
    index = Math.min(steps.length - 1, index + 1);
    render();
});
root.querySelector('[data-action="reset"]').addEventListener("click", () => {
    stopPlaying();
    index = 0;
    render();
});
root.querySelector('[data-action="play"]').addEventListener("click", () => {
    if (timer !== null) {
        stopPlaying();
        return;
    }
    if (index === steps.length - 1) index = 0;
    root.querySelector('[data-action="play"]').textContent = "Pause";
    const delay = Number(root.querySelector('[data-action="speed"]').value);
    timer = setInterval(() => {
        if (index === steps.length - 1) {
            stopPlaying();
            return;
        }
        index += 1;
        render();
    }, delay);
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

Visit order: `A, B, D, F, C, E`.

## Why it works
The call stack *is* the path from the start node to wherever you currently are — "backtracking" is nothing more than a function returning, which automatically restores you to the previous node with its remaining neighbors intact. Marking a node visited **before** recursing into its neighbors guarantees every node is expanded exactly once, which is what bounds the work to O(V+E) instead of spinning forever around a cycle. For trees this is moot (no cycles), which is why tree DFS skips the visited set entirely.

## Template
```python
# Tree DFS (recursive)
def dfs(node):
    if not node:
        return base_case
    left  = dfs(node.left)
    right = dfs(node.right)
    return combine(node.val, left, right)

# Grid DFS (iterative with explicit stack)
def dfs_grid(grid, r, c):
    stack = [(r, c)]
    visited = set()
    while stack:
        row, col = stack.pop()
        if (row, col) in visited:
            continue
        visited.add((row, col))
        for dr, dc in [(-1,0),(1,0),(0,-1),(0,1)]:
            nr, nc = row + dr, col + dc
            if 0 <= nr < len(grid) and 0 <= nc < len(grid[0]):
                stack.append((nr, nc))
```

## Complexity
Time: O(V + E) for graphs, O(n) for trees — each node is expanded once and each edge examined once thanks to the visited check | Space: O(h) — recursion depth equals the longest active path (tree height, or up to V nodes in a graph)

## Common pitfalls
- Not marking nodes visited before recursing → infinite loops on cyclic graphs.
- Stack overflow on very deep recursion (e.g. a long skewed path) — switch to iterative DFS with an explicit stack.
- Losing accumulated state across branches — forgetting to return/merge a child's result into the parent's.
- Using a binary visited set for directed-graph cycle detection — a node finished by an earlier, unrelated call isn't a cycle. Need 3-color state (`unvisited` / `on current path` / `done`) — see [[Topological Sort]].

## NeetCode examples
- [[02.MaxDepthOfBinaryTree|MaxDepthOfBinaryTree]] — classic recursive tree DFS
- [[01.NumberOfIslands|NumberOfIslands]] — flood-fill DFS on a grid
- [[04.PacificAtlanticWaterFlow|PacificAtlanticWaterFlow]] — reverse DFS from boundaries
- [[02.CloneGraph|CloneGraph]] — DFS with a visited map for node cloning

## Full guide
- [[Job Search/Neetcode/01. Questions/07. Tree/0.TreeGuide|Tree Guide]]
- [[Job Search/Neetcode/01. Questions/11. Graphs/0.GraphsGuide|Graphs Guide]]
