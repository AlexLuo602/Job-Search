# NeetCode TODO

## Writing Standard

- Use the `alex-writing` skill whenever creating or revising study notes.
- Prefer plain ELI5 words over formal or overly technical wording.
- Explain what happens and why directly. Do not use analogies.
- Use standard technical terms when they are the clearest wording. Briefly define only terms that could block understanding.

## Content and Tooling

- [x] Split the Python syntax reference into smaller files for topics such as strings.
- [x] Add an iterator section to the Python reference and link it from the Serialize and Deserialize Binary Tree attempt.
  - [x] The attempt already explains `iter()` and `next()`.
  - [x] The Python reference now has a dedicated iterator section and link target.
- [x] Scan prose and tables for literal nested arrays such as `[[1,2], [3,4]]`; wrap them in inline code or escape the opening brackets so Obsidian does not parse them as wikilinks.
  - [x] Fixed the known occurrences in Meeting Rooms and Meeting Rooms II.
- [x] Remove redundant imported body text from question notes, especially standalone `Easy` / `Medium` / `Hard` lines and `Description` labels already represented by frontmatter and the page structure.
  - [x] Cleaned Meeting Rooms and Meeting Rooms II.
- [x] Add metrics that identify weak study areas.

## Visual Cleanup

### Replace Mermaid with a Better Visual Medium

- [x] Replace [[05.MeetingRoomsII|Meeting Rooms II]] box diagrams with a scaled room timeline.
- [x] Replace box diagrams in the other five Interval questions with scaled timelines:
  - [[01.InsertInterval|Insert Interval]]
  - [[02.MergeInterval|Merge Intervals]]
  - [[03.Non-OverlappingIntervals|Non-overlapping Intervals]]
  - [[04.MeetingRooms|Meeting Rooms]]
  - [[06.MinimumIntervalToIncludeEachQuery|Minimum Interval to Include Each Query]]
- [x] Add the actual histogram and maximal rectangle to [[07.LargestRectangleInHistogram|Largest Rectangle in Histogram]].
- [x] Add a Pacific-only / Atlantic-only / shared-cell grid overlay to [[04.PacificAtlanticWaterFlow|Pacific Atlantic Water Flow]].
- [x] Replace [[06.RottenOranges|Rotten Oranges]] flowchart-like grids with fixed-width minute snapshots.
- [x] Add aligned grade-school multiplication, result positions, and carries to [[07.MultiplyStrings|Multiply Strings]].
- [x] Replace generic bit flowcharts with aligned binary rows:
  - [[06.SumOfTwoIntegers|Sum of Two Integers]]
  - [[04.ReverseBits|Reverse Bits]]
  - [[05.MissingNumber|Missing Number]]
  - [[01.SingleNumber|Single Number]]
- [x] Replace graph-shaped matrices with fixed-width matrix snapshots:
  - [[03.SetMatrixZeroes|Set Matrix Zeroes]]
  - [[02.SpiralMatrix|Spiral Matrix]]
  - [[01.RotateImage|Rotate Image]]
  - [[07.WallsAndGates|Walls and Gates]]
- [x] Add triangular interval-DP fill order and the selected final balloon to [[10.BurstBalloons|Burst Balloons]].
- [x] Align preorder and inorder slices beside each resulting subtree in [[13.ConstructBinaryTreeFromPreorderAndInorderTraversal|Construct Binary Tree from Preorder and Inorder Traversal]].
- [x] Show task frames and idle slots in [[05.TaskScheduler|Task Scheduler]].

### Remove or Simplify Redundant Mermaid

- [x] Remove the duplicative sweep diagram from [[06.ProductOfArrayExceptSelf|Product of Array Except Self]].
- [x] Keep the deque trace table and remove redundant Mermaid from [[06.SlidingWindowMaximum|Sliding Window Maximum]].
- [x] Annotate the DP matrix directly and remove the recurrence-box Mermaid from [[02.LongestCommonSubsequence|Longest Common Subsequence]].
- [x] Remove or replace the prose-heavy second diagram in [[01.ReconstructItinerary|Reconstruct Itinerary]].
- [x] Keep the grid snapshots and remove the process flowchart from [[05.SurroundedRegions|Surrounded Regions]].
- [x] Remove duplicate example/walkthrough diagrams from:
  - [[01.ReverseLinkedList|Reverse Linked List]]
  - [[01.InvertBinaryTree|Invert Binary Tree]]
  - [[05.SameTree|Same Tree]]

## Vault-Wide Migration

- [x] Complete the Dynamic Programming migration across both guides and all 23 questions.
  - [x] Add reusable memoization, tabulation, grid, sequence, interval, and rolling-state Freeforms to the two DP guides.
  - [x] Validate 47 DP Freeforms across 25 files with zero source findings.
- [x] Scan and migrate all 150 numbered question notes across 18 topic sections.
- [x] Normalize example labels, visual placement, and text fences.
- [x] Preserve useful problem visuals and move algorithm-specific state into Solution sections.
- [x] Replace wide or duplicate walkthroughs with compact text, responsive SVG, or focused Freeform where interaction adds value.
- [x] Verify all 103 SVG embeds resolve and confirm that no SVG attachment is stale.
- [x] Validate all 85 Freeforms for JavaScript syntax, ASCII safety, explicit colors, labeled regions, responsive controls, and offline execution.
- [x] Run the final source and structure gate across 150 questions and 18 guides with zero findings.

## Vault-Wide QA Re-Audit (2026-09-02)

- [x] Four-gate review pass over the full vault: 18 guides, 32 concepts, 16 reference files, 150 question notes, concept index.
  - [x] Gate 1 source checks: final vault-wide scan passes 12/12 checks with zero findings.
  - [x] Gates 2 and 3 at 100% coverage: roughly 60 targeted fixes across 47 files, including fabricated stepper values in 6 of 11 2-D DP notes, an inverted SurroundedRegions diagram state, and cancelling Prim distance errors in MinCostToConnectAllPoints. All 23 DP notes converted to explicit dictionary memoization with complete approach-matrix entries.
  - [ ] Gate 4 rendered review: **not run**, Chromium blocked by the bundled `libstdc++` (missing GLIBCXX_3.4.29).
  - [x] Misc-DSA cleanup: removed 5 cards with direct migrated replacements, then removed KMP, Segment Tree, Fenwick Tree, and Fenwick Tree 2D because they fall outside the NeetCode 150 study scope. String Operations remains as the only supplemental note.
- Details: QA tracker at `/local/home/alexlluo/.meshclaw/workspace/neetcode_qa/tracker.md` and the completion report in the rollout report.

Pixel-level Obsidian review remains manual because Chromium cannot start on this host with the bundled `libstdc++` version.
