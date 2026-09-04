# Interview Prep Guide

## Clarify & Understand (2–3 min)

- Read the problem carefully (~30–40s).
- Rephrase the problem in your own words. Confirm inputs and outputs, then optionally walk through one simple example.
- Ask about constraints such as input size and memory limits, then identify the target time and space complexity aloud.
- List the obvious edge cases, such as empty input, negative numbers, and duplicates.
- Short pauses of 5–15 seconds are fine.

## Initial Approach & Brute Force (1 min)

- Think silently for 5–20 seconds to identify a brute-force solution.
- State the brute-force approach clearly in one sentence.
- Give its complexity, such as “That’s O(n²) time and O(1) space.”

## Optimization (5 min)

- Transition explicitly: “That’s too slow for n up to 10⁵, so let’s try something better.”
- Brainstorm more efficient approaches by discussing key ideas rather than every detail.
- If silence reaches 30 seconds, give a checkpoint such as “I’m considering whether a hash map would help here.”
- Do not stay silent for a full minute.

## Optimization Patterns

- Use the [[Job Search/Neetcode/03. Reference/Concepts/_Index|Concepts Index]] to review the patterns and algorithms that remove the brute-force bottleneck.

## Outline (3 min)

- Outline the solution with comments or pseudocode bullets.
- Summarize the outline aloud: “We’ll scan once to build a map, then use two pointers to …”

## Coding (12–14 min)

- Explain each section before writing it and connect it to the outline.
- Code in silence when necessary.
- After finishing a block, mention the current state: “Now my heap stores the first k elements; next I’ll pop as I slide the window.”

## Post-Coding (3–4 min)

- Dry-run the solution with an example and edge cases.
- State the time and space complexity, tying both back to the constraints.
- If the solution is suboptimal, acknowledge the bottleneck and possible improvement.

## When You Hit a Blocker

1. **Vocalize immediately** (don't go silent)
   - "I'm not immediately seeing the pattern here. Let me think through this..."
   - Interviewers want to hear your thought process, not see you freeze
2. **Ask clarifying questions** (even if they seem obvious)
   - "Can I clarify the constraints?"
   - "Are there any edge cases I should consider?"
   - This buys time and shows you think systematically
3. **Work through examples** (out loud)
   - "Let me trace through the first example..."
   - Talking through examples often reveals the pattern
   - Interviewers see problem-solving in action
4. **Propose a naive solution first**
   - "What if I just brute force it with X approach? That would be O(n²)..."
   - This shows you can think iteratively
   - Then: "But can we optimize...?"
5. **Fish for hints strategically**
   - Don't say "I don't know"
   - Say: "I'm thinking about X approach, but I'm wondering if there's a pattern I'm missing. What if I tried Y?"
   - This invites guidance without giving up
6. **If truly stuck after 10-15 min:**
   - "I want to make sure we use time well. Would it help if I coded a brute force solution while we talk about optimizations?"
   - Or: "Can you point me in the right direction? I want to show you I can implement once I understand the approach."
