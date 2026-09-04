---
type: concept
tags: ["concept"]
---

# Linked List Reversal

**TL;DR:** Walk the list once, flipping each node's `.next` to point at its predecessor instead of its successor.

## When to reach for it
- The problem says "reverse," "reverse between left and right," or "reverse in groups of k."
- You need to check a palindrome-style property without O(n) extra space (reverse the second half, then compare).
- A structural rewrite must happen **in place** — no new nodes, O(1) extra space.
- It shows up as a sub-step inside a bigger composition (e.g. reverse-second-half inside Reorder List, or k-group reversal repeated across an entire list).

## How it works
Three pointers do all the work. `prev` is the reversed chain built so far, `curr` is the node being changed, and `nxt` saves the rest of the list before `curr.next` is replaced. Every loop follows the same order: save, rewire, then advance.

### Interactive walkthrough

The detached frames make the replaced link visible. `nxt` still points to the saved remainder, so no nodes are lost while `curr.next` changes direction.

```freeform
const steps = [
    { prev:"None", curr:"1", nxt:"not set", links:[[1,2],[2,3]], nexts:{1:"2",2:"3",3:"None"}, detached:null, components:"Reversed: None | Current: 1 | After current: 2->3->None", line:1, action:"Start with curr at node 1 and an empty reversed chain." },
    { prev:"None", curr:"1", nxt:"2", links:[[1,2],[2,3]], nexts:{1:"2",2:"3",3:"None"}, detached:null, components:"Reversed: None | Current: 1 | Saved remainder: 2->3->None", line:2, action:"Save node 2 in nxt before changing the link from node 1." },
    { prev:"None", curr:"1", nxt:"2", links:[[2,3]], nexts:{1:"detached",2:"3",3:"None"}, detached:[1,2], components:"Reversed: None | Detached current: 1 | Saved remainder: 2->3->None", line:3, action:"Detach the old 1->2 link while nxt keeps node 2 reachable." },
    { prev:"None", curr:"1", nxt:"2", links:[[2,3]], nexts:{1:"None",2:"3",3:"None"}, detached:null, components:"Reversed: 1->None | Saved remainder: 2->3->None", line:3, action:"Point node 1 to None to finish the first reversed link." },
    { prev:"1", curr:"2", nxt:"not set", links:[[2,3]], nexts:{1:"None",2:"3",3:"None"}, detached:null, components:"Reversed: 1->None | Current: 2 | After current: 3->None", line:4, action:"Advance prev to node 1 and curr to the saved node 2." },
    { prev:"1", curr:"2", nxt:"3", links:[[2,3]], nexts:{1:"None",2:"3",3:"None"}, detached:null, components:"Reversed: 1->None | Current: 2 | Saved remainder: 3->None", line:2, action:"Save node 3 in nxt before changing the link from node 2." },
    { prev:"1", curr:"2", nxt:"3", links:[], nexts:{1:"None",2:"detached",3:"None"}, detached:[2,3], components:"Reversed: 1->None | Detached current: 2 | Saved remainder: 3->None", line:3, action:"Detach the old 2->3 link while nxt keeps node 3 reachable." },
    { prev:"1", curr:"2", nxt:"3", links:[[2,1]], nexts:{1:"None",2:"1",3:"None"}, detached:null, components:"Reversed: 2->1->None | Saved remainder: 3->None", line:3, action:"Point node 2 to node 1 to extend the reversed chain." },
    { prev:"2", curr:"3", nxt:"not set", links:[[2,1]], nexts:{1:"None",2:"1",3:"None"}, detached:null, components:"Reversed: 2->1->None | Current: 3 | After current: None", line:4, action:"Advance prev to node 2 and curr to the saved node 3." },
    { prev:"2", curr:"3", nxt:"None", links:[[2,1]], nexts:{1:"None",2:"1",3:"None"}, detached:null, components:"Reversed: 2->1->None | Current: 3 | Saved remainder: None", line:2, action:"Save None in nxt because node 3 is the original tail." },
    { prev:"2", curr:"3", nxt:"None", links:[[3,2],[2,1]], nexts:{1:"None",2:"1",3:"2"}, detached:null, components:"Reversed: 3->2->1->None | Saved remainder: None", line:3, action:"Point node 3 to node 2 to complete the reversed links." },
    { prev:"3", curr:"None", nxt:"not set", links:[[3,2],[2,1]], nexts:{1:"None",2:"1",3:"2"}, detached:null, components:"Reversed: 3->2->1->None | Remaining: None", line:5, action:"Advance curr to None and return node 3 as the new head." }
];

const root = document.createElement("section");
root.style.cssText = "font:14px system-ui;max-width:700px;border:1px solid var(--background-modifier-border);border-radius:10px;padding:12px;background:var(--background-primary);color:var(--text-normal);";
root.innerHTML = `
<style>
  .llr-svg{width:100%;height:auto;display:block}
  .llr-state{display:grid;grid-template-columns:1fr 1fr;gap:8px;margin-top:8px}
  .llr-code{overflow-x:auto;font-size:13px}
  .llr-controls{display:flex;align-items:center;gap:8px;flex-wrap:wrap;margin-top:10px}
  .llr-controls button,.llr-controls select{min-height:44px;font-size:14px;touch-action:manipulation}
  .llr-controls button{padding:6px 12px}
  @media(max-width:520px){.llr-state{grid-template-columns:1fr}.llr-code{font-size:12px}.llr-controls button{flex:1 1 72px}.llr-speed{flex:1 1 100%;display:flex;align-items:center;gap:8px}.llr-speed select{flex:1}.llr-count{width:100%;text-align:center;margin-left:0!important}}
</style>
<svg class="llr-svg" viewBox="0 0 500 245" preserveAspectRatio="xMidYMid meet" role="img"></svg>
<div style="display:flex;gap:12px;flex-wrap:wrap;margin:2px 0 8px;font-size:12px"><span>Orange = curr</span><span>Blue = prev</span><span>Purple = nxt</span><span>Red X = old link removed</span></div>
<div data-role="status" aria-live="polite" style="min-height:22px;padding:9px 10px;border-radius:8px;background:var(--background-secondary)"></div>
<div class="llr-state">
  <div style="padding:8px;border:1px solid var(--background-modifier-border);border-radius:8px"><strong>Pointers</strong><div data-role="pointers" style="margin-top:4px;font-family:var(--font-monospace)"></div></div>
  <div style="padding:8px;border:1px solid var(--background-modifier-border);border-radius:8px"><strong>List components</strong><div data-role="components" style="margin-top:4px;font-family:var(--font-monospace)"></div></div>
</div>
<pre class="llr-code" data-role="code" style="padding:8px;border-radius:8px;background:var(--background-secondary)"><code><span data-line="1">prev = None; curr = head</span>
<span data-line="2">nxt = curr.next</span>
<span data-line="3">curr.next = prev</span>
<span data-line="4">prev, curr = curr, nxt</span>
<span data-line="5">return prev</span></code></pre>
<div class="llr-controls">
  <button type="button" data-action="previous">Previous</button><button type="button" data-action="play">Play</button><button type="button" data-action="next">Next</button><button type="button" data-action="reset">Reset</button>
  <label class="llr-speed">Speed <select data-action="speed"><option value="1400">Slow</option><option value="900" selected>Normal</option><option value="500">Fast</option></select></label>
  <span class="llr-count" data-role="counter" style="margin-left:auto"></span>
</div>`;

const svg = root.querySelector("svg");
const ns = "http://www.w3.org/2000/svg";
const positions = {1:{x:90,y:95},2:{x:250,y:95},3:{x:410,y:95}};
let index = 0;
let timer = null;

function svgNode(name, attrs, text) {
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

function renderList(step) {
    svg.replaceChildren();
    const defs = svgNode("defs", {});
    const marker = svgNode("marker", {id:"llr-arrow",viewBox:"0 0 10 10",refX:9,refY:5,markerWidth:7,markerHeight:7,orient:"auto-start-reverse"});
    marker.appendChild(svgNode("path", {d:"M 0 0 L 10 5 L 0 10 z",fill:"#34495e"}));
    defs.appendChild(marker);
    svg.appendChild(defs);
    svg.appendChild(svgNode("text", {x:250,y:23,"text-anchor":"middle","font-size":17,"font-weight":700,fill:"#17202a"}, "Links stored in each node"));

    for (const [from, to] of step.links) {
        const a = positions[from];
        const b = positions[to];
        const direction = b.x > a.x ? 1 : -1;
        svg.appendChild(svgNode("line", {x1:a.x+direction*31,y1:a.y,x2:b.x-direction*31,y2:b.y,stroke:"#34495e","stroke-width":3,"marker-end":"url(#llr-arrow)"}));
    }
    if (step.detached) {
        const a = positions[step.detached[0]];
        const b = positions[step.detached[1]];
        const x = (a.x + b.x) / 2;
        svg.appendChild(svgNode("line", {x1:x-9,y1:82,x2:x+9,y2:108,stroke:"#c0392b","stroke-width":4}));
        svg.appendChild(svgNode("line", {x1:x+9,y1:82,x2:x-9,y2:108,stroke:"#c0392b","stroke-width":4}));
    }

    for (const value of [1,2,3]) {
        const p = positions[value];
        let fill = "#f4f6f7";
        if (step.prev === String(value)) fill = "#aed6f1";
        if (step.nxt === String(value)) fill = "#d7bde2";
        if (step.curr === String(value)) fill = "#f8c471";
        svg.appendChild(svgNode("circle", {cx:p.x,cy:p.y,r:29,fill,stroke:"#2c3e50","stroke-width":2}));
        svg.appendChild(svgNode("text", {x:p.x,y:p.y+6,"text-anchor":"middle","font-size":19,"font-weight":700,fill:"#17202a"}, value));
        const labels = [];
        if (step.prev === String(value)) labels.push("prev");
        if (step.curr === String(value)) labels.push("curr");
        if (step.nxt === String(value)) labels.push("nxt");
        if (labels.length) svg.appendChild(svgNode("text", {x:p.x,y:51,"text-anchor":"middle","font-size":14,"font-weight":700,fill:"#17202a"}, labels.join(" + ")));
        svg.appendChild(svgNode("text", {x:p.x,y:145,"text-anchor":"middle","font-size":14,"font-weight":600,fill:"#34495e"}, "next: " + step.nexts[value]));
    }
    svg.appendChild(svgNode("text", {x:250,y:198,"text-anchor":"middle","font-size":14,"font-weight":600,fill:"#34495e"}, "prev=" + step.prev + " | curr=" + step.curr + " | nxt=" + step.nxt));
    svg.appendChild(svgNode("text", {x:250,y:224,"text-anchor":"middle","font-size":13,fill:"#566573"}, "The arrows show only links that exist in this frame."));
}

function render() {
    const step = steps[index];
    renderList(step);
    svg.setAttribute("aria-label", "Linked list reversal step " + (index + 1) + ". " + step.action + " " + step.components + ".");
    root.querySelector('[data-role="status"]').textContent = step.action;
    root.querySelector('[data-role="pointers"]').textContent = "prev=" + step.prev + " | curr=" + step.curr + " | nxt=" + step.nxt;
    root.querySelector('[data-role="components"]').textContent = step.components;
    for (const line of root.querySelectorAll('[data-role="code"] [data-line]')) {
        const active = Number(line.getAttribute("data-line")) === step.line;
        line.style.display = "block";
        line.style.background = active ? "#f8c471" : "transparent";
        line.style.color = active ? "#17202a" : "inherit";
        line.style.fontWeight = active ? "700" : "400";
        if (active) line.setAttribute("aria-current", "step"); else line.removeAttribute("aria-current");
    }
    root.querySelector('[data-role="counter"]').textContent = "Step " + (index + 1) + " / " + steps.length;
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

display(root);
render();
```

Result: `3 -> 2 -> 1 -> None`, and `prev` is the new head.

Reversing a **sublist** `[left, right]` is the same motion applied to a slice: walk `left - 1` steps to find the anchor node just before the range, then repeatedly move the node right after that anchor to the front of the range (head-insertion), `right - left` times.

## Why it works
Invariant maintained by the loop: **at the top of every iteration, `prev` is the head of a fully-reversed chain containing every node visited so far, and it is correctly terminated** (its tail points to whatever should come "after" the reversed part — `None` at the very first step). Because you save `nxt` before overwriting `curr.next`, you never lose the reference to the unprocessed remainder of the list. Each iteration extends the reversed prefix by exactly one node without ever revisiting a node, so after `n` iterations the entire list is reversed and `prev` holds the new head.

## Template
```python
# Reverse entire list (iterative)
def reverse_list(head):
    prev = None
    curr = head
    while curr:
        nxt = curr.next   # 1. save
        curr.next = prev  # 2. rewire
        prev = curr        # 3. advance prev
        curr = nxt         #    advance curr
    return prev            # new head

# Reverse a sublist [left, right] (1-indexed) in one pass
def reverse_between(head, left, right):
    dummy = ListNode(0, head)
    pre = dummy
    for _ in range(left - 1):
        pre = pre.next
    tail = pre.next                # becomes the tail of the reversed range
    for _ in range(right - left):
        nxt = tail.next
        tail.next = nxt.next
        nxt.next = pre.next
        pre.next = nxt
    return dummy.next
```

## Complexity
Time: O(n) — every node is visited and rewired exactly once, so work scales linearly. Space: O(1) iterative (three pointers); O(n) if written recursively, since each call holds a stack frame open until the base case unwinds.

## Common pitfalls
- Overwriting `curr.next` before saving it into `nxt` — this severs your only path to the rest of the list.
- Forgetting the dummy node when the reversed range can start at the head (`left == 1`) — without it you need a separate branch for "no node before the range."
- Returning `head` instead of `prev` — after a full reversal, the original `head` is now the *tail*, not the head.
- Choosing recursive reversal for long lists without weighing the O(n) call-stack cost and Python's recursion-depth limit.

## NeetCode examples
- [[01.ReverseLinkedList|ReverseLinkedList]] — canonical iterative reversal
- [[11.ReverseNodesInK-Group|ReverseNodesInK-Group]] — reverse k-node sublists repeatedly
- [[03.ReorderList|ReorderList]] — reverse second half, then interleave

## Full guide
[[Job Search/Neetcode/01. Questions/06. LinkedList/0.LinkedListGuide|Linked List Guide]]
