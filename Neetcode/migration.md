# NeetCode Vault Migration

## Goal

Make the vault easier to study in editing, reading, desktop, and mobile views without forcing every note into the same format.

The migration should:

- Put useful problem visuals beside the examples.
- Use Freeform for changing algorithm state when interaction helps.
- Remove duplicate diagrams and walkthroughs.
- Prevent Mermaid, tables, SVGs, and code blocks from widening the entire note.
- Keep guides, concepts, and question notes easy to navigate.

## Writing Rules

- Start with the `amazon-writing` skill as the sentence-level clarity baseline.
- Then apply the local `alex-writing` Codex CLI skill at `/local/home/alexlluo/.codex/skills/alex-writing/SKILL.md` for Alex's voice and context-specific writing rules.
- Use clear adult language. Do not use childish or patronizing ELI5 phrasing, and do not use overly technical prose when a direct explanation is clearer.
- Keep precise technical terms when they are needed to understand or verify the algorithm. Explain unfamiliar terms in plain language when they first appear.
- State what happens and why before naming an implementation detail.
- Prefer active voice, concrete nouns, precise verbs, and everyday words.
- Do not oversimplify distinctions that affect correctness, complexity, or implementation.
- Do not use analogies.
- Avoid em dashes and semicolons in new or revised text.
- Do not add explanation that repeats a self-explanatory visual.
- Break proof-style reasoning into short ordered bullets when each step establishes a separate claim.
- State algorithm guarantees precisely. Distinguish what a pass guarantees from work that may finish early because of a favorable processing order.
- Explain subtle implementation variants with a concrete failure case before presenting the corrected code.

## Example Section Standard

Use this order:

````markdown
**Example 1:**

![[problem-example.svg]]

```text
Input: ...
Output: ...
Explanation: ...
```

**Example 2:**

```text
Input: ...
Output: ...
```
````

Rules:

- Bold every example label.
- Place the static visual immediately after the relevant example label.
- Put Input, Output, and Explanation after the visual.
- Do not add headings such as `Visual Example` or `Winning container`.
- Do not explain axes, colors, or scale when the visual is already clear.
- Do not repeat the result below the visual when the example already states it.
- Show only information contained in the problem's example input and output.
- Do not reveal the solution method in the problem description.
- Give related examples their own visuals when comparing parameter changes, valid routes, or rejected routes improves understanding, even if each example is simple by itself.
- Reuse the same geometry across related examples. Change only the route, range, node, or state that the example is meant to compare.

## Static Visual Standard

Use a static visual only when the problem itself is easier to understand spatially or structurally.

Good candidates:

- Trees
- Graphs
- Grids and matrices
- Linked lists
- Intervals and timelines
- Geometry
- Histograms and heights
- Pointer layouts
- DP matrices when the complete state helps explain the example

Usually unnecessary:

- Simple scalar inputs
- Direct string examples
- Basic hash-map or counting problems
- Examples already clear from a short array
- Problems where a visual would expose the algorithm instead of the problem

Preferred format:

- Use standalone SVG attachments for custom diagrams.
- Store SVG files under `Alex_Personal/attachments` with unique lowercase filenames.
- Embed by basename, for example `![[container-with-most-water.svg]]`.
- Avoid raw inline SVG because Obsidian may sanitize part of it in reading mode.
- Use compact Mermaid only when it does not widen the note.
- Use a matrix, Markdown table, or text drawing when that representation is clearer.

Default visual style:

- White background
- Black or dark-gray neutral axes and structures
- Red selected, conflicting, rejected, target, or answer-defining elements
- Transparent blue highlighted regions, paths, ranges, or water
- Extra colors only when the problem requires more states
- Responsive width with no whole-document horizontal scrolling
- No text placed under lines or bars without a readable background
- Offset every edge-weight label from its arrow with visible whitespace. A white text outline improves contrast but does not replace proper spacing.
- Preserve accepted visual regions during feedback. Make targeted changes only to the diagram or element the feedback identifies.

## Freeform Standard

Freeform belongs in the Solution section and explains changing algorithm state.

Use Freeform when it materially helps show:

- Queue, stack, heap, or recursion changes
- Pointer movement and rewiring
- Backtracking and restoration
- Edge acceptance, rejection, or relaxation
- DP fill order
- Greedy windows or boundaries
- State transitions that are difficult to follow in a static table

Do not add Freeform merely to add controls.

Use separate steppers when they teach different failure mechanisms or state transitions. Do not avoid Freeform because a note already contains another diagram. Use as many focused steppers as the material needs, provided each one adds a distinct view and remains responsive. The limit is instructional value and readability, not a fixed diagram count. Avoid only diagrams that repeat the same state or explanation.

When diagrams in one note share a visual language:

- Select one diagram as the sizing and typography reference.
- Match rendered node size, label size, and edge spacing, not only the raw SVG radius and font values.
- Account for both the SVG `viewBox` and the card width because they determine the final scale.
- Use an explicit responsive card width such as `width:min(100%,420px)` with `box-sizing:border-box` when a compact diagram should not fill the note.
- Recheck the rendered result after changing a card width. Equal source dimensions can still render at different sizes.

### Dynamic Programming approach standard

For DP questions where memoization and tabulation are both natural:

- Present **Approach 1: Top-down memoization** to derive the recurrence, base cases, and cache key.
- Use an explicit dictionary for memoization. Do not use `@cache` or `@lru_cache` in study solutions because decorators hide the cache key, lookup, and write.
- Present **Approach 2: Bottom-up tabulation** to explain base-state initialization, fill order, and space optimization.
- Give each approach a Freeform when it teaches a different mechanism. Memoization should show recursive calls, cache writes, cache hits, and skipped repeated work. Tabulation should show active state transitions and fill order.
- Share the state definition, recurrence, correctness explanation, and complexity comparison instead of repeating them under both approaches.
- Number approaches only when both are actually present. Use an unnumbered `Approach:` heading when the note intentionally teaches one primary method.
- Do not force both forms when one is unnatural. Center expansion, rolling maximum/minimum, patience sorting, and DFS with memoization may remain single-primary approaches when the alternative obscures the core idea.

Before marking a DP guide section complete:

- Name the hierarchy explicitly as **pattern**, **subpattern**, and **example**. A problem name must not stand in for a reusable pattern.
- Give each subpattern a clear parent. Use an action-oriented name that tells the reader how states connect, then define the exact state and recurrence.
- Add a wikilink when a concrete problem is introduced as an example.
- Check whether top-down memoization and bottom-up tabulation are both natural for the example. Include both when they expose the same state in useful ways.
- When omitting a natural approach, state why the omission improves the lesson. Do not let an approach disappear by oversight.
- Label a faster problem-specific algorithm as an alternative for that problem, not as part of the DP subpattern.
- Verify that memoization and tabulation use equivalent state meanings, base cases, transitions, final-answer extraction, and complexity claims.

Every stepper should include:

- One concise current-action sentence
- The underlying problem input remains visible whenever it is needed to understand the active state. Highlight the array element, string index, grid cell, graph node, interval, or other input used by the current step.
- No more than two state panels
- Short pseudocode with the active line marked
- Previous, Play/Pause, Next, Reset, speed, and step counter at the bottom
- Controls at least 44px high
- Portrait and desktop layouts without clipping
- Offline JavaScript with no external imports
- ASCII-only Freeform source because the plugin uses `btoa`
- Explicit SVG colors rather than Obsidian CSS variables inside an iframe

## Fallback Policy

Freeform is the primary solution walkthrough.

Keep a static visual when it provides a different view:

- Complete problem example before the solution
- Full board, tree, graph, matrix, or timeline
- Final state at a glance
- Compact formula or summary
- Printable/exportable view

Remove a static fallback when it merely repeats the same graph and steps as Freeform.

Do not keep full duplicate walkthrough tables. Keep only a short result or final-state summary when useful.

## Guides and Concepts

One-to-one topics live directly in their guide:

- Two Pointers
- Sliding Window
- Binary Search
- Trie
- Backtracking
- Greedy
- Bit Manipulation

Shared concepts remain separate reusable notes:

- DFS
- BFS
- Heap
- Dynamic Programming
- Memoization
- Union-Find
- Dijkstra
- Bellman-Ford
- Prim
- Kruskal
- Monotonic Stack
- Two Heaps
- Quickselect
- Fast & Slow Pointers
- Linked List Reversal

Pending structural decision:

- [x] Added a standalone `Tabulation` concept note to match `Memoization`. Both notes link to the shared `Dynamic Programming` concept and the detailed DP guides.

## Completed Work

- [x] Created `Starting Point.md` as the vault entry point.
- [x] Added complete problem-link coverage to all topic guides.
- [x] Reorganized concept files into topic folders.
- [x] Installed and enabled Dataview, Automatically Reveal Active File, Editor Width Slider, and Freeform.
- [x] Added interactive steppers selectively across concepts and questions.
- [x] Added guide links to interactive walkthroughs.
- [x] Removed redundant imported difficulty and `Description` labels.
- [x] Protected literal nested arrays from Obsidian wikilink parsing.
- [x] Split the Python cheatsheet into focused topic notes.
- [x] Added iterator documentation and Study Metrics.
- [x] Consolidated seven one-to-one concept/guide pairs.
- [x] Migrated links from deleted concept notes to their guides.
- [x] Applied the lean fallback policy to current Freeform notes.
- [x] Finalized Container With Most Water's LeetCode-style example asset.
- [x] Improved Bellman-Ford with a bullet-based proof, a negative-cycle stepper, a concrete K-hop failure example, a snapshot comparison stepper, and consistent rendered diagram sizing.
- [x] Completed the Cheapest Flights Within K Stops pilot with one visual per example, consistent comparison geometry, separate Freeforms for edge budget and snapshot isolation, precise Dijkstra wording, and a concrete in-place-update failure case.
- [x] Migrated both Dynamic Programming guides and all 23 DP questions. The automated source gates validated 47 Freeforms across 25 files at that time. Later manual review found taxonomy, wikilink, and approach-coverage omissions, so this result is not semantic or rendered approval. The follow-up QA below remains open.
- [x] Completed the source migration across all 150 numbered question notes and 18 topic guides. The automated source scan validated 85 Freeforms, 103 SVG embeds, 68 retained Mermaid blocks, attachments, fences, Python question implementations, table aliases, accessibility metadata, responsive controls, and offline JavaScript at that time. The stronger semantic, algorithm, and rendered-review gates below still require a separate audit.

## Pending Vault-Wide Migration

The previous broad pass was stopped. Resume from this checklist rather than restarting without review.

### Phase 1: Inventory

- [x] Scan all 150 numbered question notes.
- [x] Record which notes already have a useful visual before `## Solution`.
- [x] Record which notes have a problem-level visual inside the solution that should move to Examples.
- [x] Record wide Mermaid diagrams or tables that widen the whole document.
- [x] Record notes where no visual is needed.

### Phase 2: Example Structure

- [x] Bold all `Example 1`, `Example 2`, and later example labels.
- [x] Move retained problem-level visuals directly under the matching example label.
- [x] Remove extra visual headings and redundant explanatory paragraphs.
- [x] Keep algorithm-specific diagrams inside the solution.
- [x] Remove duplicate copies after moving visuals.

### Phase 3: Visual Restyling

- [x] Replace unreliable inline SVG with standalone SVG attachments.
- [x] Replace wide Mermaid when it causes desktop or mobile document overflow.
- [x] Convert stateful wide diagrams to Freeform when interaction helps.
- [x] Convert static wide diagrams to responsive SVG, matrix, timeline, table, or text.
- [x] Apply the white/black/red/blue visual style where it fits naturally.
- [x] Do not force this style when a problem requires different semantic colors.

### Phase 4: Selective Missing Visuals

- [x] Add visuals to spatial or structural examples that currently lack them.
- [x] Leave simple and self-explanatory examples text-only.
- [x] Verify every new visual shows only example information and not the solution.

### Phase 5: Cleanup

- [x] Remove duplicate Mermaid diagrams and walkthrough tables after Freeform is present.
- [x] Keep concise full-state summaries where they add a different view.
- [x] Remove stale attachments that are no longer embedded.
- [x] Update `TODO.md` and the rollout report after completion.

## QA System

A zero-finding source scan proves only that the checks encoded in that scan passed. It does not prove that the explanation is complete, the hierarchy is clear, or the selected approaches teach the intended pattern. Do not mark a scope complete until all applicable gates below pass.

### Gate 1: Deterministic source validation

Run this gate across every changed file and report each check separately:

- Parse every Python block with `ast.parse`.
- Parse every Freeform JavaScript block with Node `new Function`.
- Check balanced Markdown and Freeform fences.
- Resolve wikilinks and embedded attachments.
- Check table aliases, ASCII-only Freeform source, offline JavaScript, responsive widths, control heights, and accessibility metadata.
- Check repository rules such as explicit memoization dictionaries and prohibited decorators.

### Gate 2: Semantic structure review

Review every changed guide and concept section manually. Confirm that a fresh reader can identify:

1. The reusable pattern.
2. Any subpattern and its parent.
3. The concrete linked example.
4. The state meaning.
5. The base cases and transition.
6. How to extract the final answer.
7. Which alternatives are general and which are problem-specific.

Reject a section when a problem name is used as the pattern name, a subpattern has no named parent, or a heading does not explain the decision being made.

### Gate 3: Algorithm completeness and consistency

For every algorithm example:

- Compare prose, recurrence, code, Freeform steps, and complexity claims against each other.
- Record whether memoization, tabulation, space optimization, and a non-DP alternative are natural, present, or intentionally omitted.
- Require a written reason for each intentional omission when the missing approach would otherwise be expected from the guide standard.
- Test edge cases that affect initialization or final-answer extraction, including empty input when the problem contract allows it.
- Confirm that optimizations preserve the original state meaning or clearly explain why they use a different invariant.

### Gate 4: Rendered and interaction review

Source checks cannot approve visual rendering. Review each changed visual and Freeform in Obsidian at desktop and narrow widths when a compatible renderer is available. Check clipping, whole-note overflow, label overlap, control size, color meaning, keyboard access, and step behavior. Record the renderer and viewport used. If rendering is blocked, mark this gate **not run** instead of treating source validation as visual approval.

### Completion and sampling policy

- Apply Gates 2 and 3 to 100 percent of changed guides and shared concepts. Do not sample these files.
- Apply the full checklist to every changed question note. A vault-wide aggregate script may supplement this review but cannot replace it.
- Use a fresh-reader pass after the authoring pass. The reviewer should receive the section without prior discussion and identify the pattern hierarchy and recurrence from the text alone.
- Report results by gate and file count. Do not use an unqualified statement such as `zero findings`.
- Keep unresolved or unrun gates in a follow-up checklist with file scope and reason.

## Follow-up QA after the completed migration

Completed 2026-09-02. Full results by file in the QA tracker (`/local/home/alexlluo/.meshclaw/workspace/neetcode_qa/tracker.md`) and the completion report appended to the rollout report.

- [x] Re-audit all 18 topic guides and shared algorithm concepts with Gates 2 and 3. (18 guides + 32 migrated concepts + 16 reference files. 14 guide/concept fixes applied. Five Misc-DSA cards with direct migrated replacements were deleted on 2026-09-02. KMP, Segment Tree, Fenwick Tree, and Fenwick Tree 2D were removed on 2026-09-03 because they fall outside the NeetCode 150 study scope. String Operations remains as the only supplemental note.)
- [x] Build a DP approach matrix for both DP guides and all 23 DP questions. Record memoization, tabulation, optimization, alternative algorithms, and intentional omissions. (All 23 entries in the tracker. All @cache uses converted to explicit dictionaries. 6 of 11 2-D DP notes had fabricated stepper values, all hand-verified and fixed.)
- [x] Verify that every guide distinguishes patterns, subpatterns, linked problem examples, and problem-specific alternatives. (1-D guide cheatsheet re-synced to sections; LIS patience sorting labeled problem-specific in guide and note.)
- [x] Run a fresh-reader pass on each guide and fix headings that do not communicate the decision or transition. (Fixed: 1-D DP palindrome heading, Tree guide hierarchy, 2-D guide pattern count.)
- [x] Reissue the completion summary with separate source, semantic, algorithm, and rendered-review results. (Appended to the rollout report. Gate 4 remains **not run**: Chromium blocked by bundled libstdc++ missing GLIBCXX_3.4.29.)

## Next-session handoff

Start a fresh session for the final vault-wide QA re-audit. This is a review and targeted-fix pass, not a request to regenerate every note or repeat the visual migration from the beginning.

### Objective

Apply the four QA gates to the migrated vault, find content mistakes that source scripts cannot detect, make targeted corrections, and issue a completion report that separates source, semantic, algorithm, and rendered-review results.

### Startup order

1. Read this file in full before editing any note.
2. Load `amazon-writing`, then `/local/home/alexlluo/.codex/skills/alex-writing/SKILL.md`.
3. Inventory the current files and compare the inventory with the counts recorded here. Do not assume the old counts are still exact after later edits.
4. Audit all 18 topic guides and shared algorithm concepts with Gates 2 and 3 before reviewing individual question notes.
5. Audit both DP guides and all 23 DP questions with the DP approach matrix.
6. Audit the 150 numbered question notes in topic-sized batches with all applicable gates.
7. Run the vault-wide deterministic source checks after each batch and once more at the end.

### Batch contract

For each batch:

1. Read the full guide or question note before changing it.
2. Record semantic and algorithm findings before editing so a rewrite does not hide the original failure.
3. Make targeted changes. Preserve accepted prose, visuals, and Freeforms outside the identified issue.
4. Show a unified diff with absolute paths after every file change.
5. Run targeted Python, JavaScript, fence, link, attachment, and layout-rule validation.
6. Mark rendered review as **not run** when it cannot be performed. Do not convert source checks into visual approval.
7. Update the follow-up checklist only after the applicable gate has passed for the full stated scope.

### Current DP baseline

The 1-D DP guide now uses this hierarchy:

- **Take or skip** is a general decision pattern.
- **Boolean 0/1 knapsack** is a take-or-skip subpattern.
- [[12.PartitionEqualSubsetSum|Partition Equal Subset Sum]] is an example of boolean 0/1 knapsack.
- **Sequence DP** is a separate pattern.
- **Extend from a compatible earlier position** is a Sequence DP subpattern.
- [[11.LongestIncreasingSubsequence|Longest Increasing Subsequence]] is an example with explicit-dictionary memoization, tabulation, and a separately labeled patience-sorting alternative.
- **Build from an earlier valid boundary** is another Sequence DP subpattern.
- [[10.WordBreak|Word Break]] is its linked example.

Use this section as a corrected reference for pattern, subpattern, example, and problem-specific-alternative labeling. Do not assume the rest of the vault already follows it.

### Environment limitation

Playwright Chromium cannot currently run because the bundled `libstdc++.so.6` lacks `GLIBCXX_3.4.29` and `CXXABI_1.3.13`. Source and layout-rule checks remain available. Until Chromium or another compatible rendered-view path works, report Gate 4 as **not run** and do not claim pixel-level approval.

### Completion criteria

The re-audit is complete only when:

- Every guide, shared concept, and changed question note has a recorded result for each applicable gate.
- Every DP example has an approach-matrix entry and a reason for each intentional omission.
- All source validators pass after the final edits.
- Every unresolved or unrun item is listed with its file scope and reason.
- `migration.md`, `TODO.md`, and the rollout report agree on counts, completed gates, open gates, and renderer limitations.

## Validation Checklist

For every changed note:

- [ ] Example labels are bold.
- [ ] Static visual is before `## Solution`.
- [ ] Input, Output, and Explanation follow the visual.
- [ ] No extra visual heading or redundant explanation remains.
- [ ] No solution-specific hint appears in the problem description.
- [ ] Attachment exists and resolves in both vault configurations.
- [ ] Reading mode renders the full visual.
- [ ] Editing and reading modes do not create whole-document horizontal overflow.
- [ ] SVG labels do not overlap bars, edges, or selected regions.
- [ ] Edge-weight labels have visible whitespace from their arrow paths and arrowheads.
- [ ] Related example visuals preserve the same node positions, scale, and base geometry unless the input structure changes.
- [ ] Mermaid is compact or contained.
- [ ] Freeform remains offline, responsive, and ASCII-safe.
- [ ] Related Freeform diagrams use consistent rendered node sizes, typography, and edge spacing.
- [ ] Compact Freeform cards are explicitly width-capped without clipping controls or state panels.
- [ ] Diagram sizing has been checked in the rendered view, not inferred only from SVG source values.
- [ ] Markdown and Freeform fences are balanced.
- [ ] Wikilinks resolve.
- [ ] Table aliases do not contain unescaped pipes.
- [ ] No new em dashes or analogies were introduced.

## Reference Implementation

Use Container With Most Water as the static example reference:

- Question note: `01. Questions/02. Two Pointer/04.ContainerWithMostWater.md`
- Attachment: `attachments/container-with-most-water.svg`

Use Cheapest Flights Within K Stops as the multi-example question reference:

- Question note: `01. Questions/12. Advanced Graphs/06.CheapestFlightsWithinKStops.md`
- Each related example has its own static visual.
- Related examples reuse graph geometry and change only route validity and selection.
- Separate Freeforms explain the edge budget and snapshot isolation because they are different mechanisms.

Use DFS as the general interactive reference:

- `03. Reference/Concepts/03. Trees & Graphs/DFS.md`

Use Bellman-Ford as the multi-diagram concept reference:

- `03. Reference/Concepts/03. Trees & Graphs/Bellman-Ford.md`
- The main walkthrough defines the shared node and label style.
- The negative-cycle stepper shows how to keep a simpler diagram compact.
- The K-hop stepper shows when a separate comparison diagram teaches a different failure mechanism.

Detailed prior rollout report:

- `/local/home/alexlluo/NeetCode Interactive Rollout Report.md`
