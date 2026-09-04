---
question: "[[11.ReverseNodesInK-Group|ReverseNodesInK-Group]]"
topic: ["LinkedList"]
lc_difficulty: Hard
tags: ["neetcode-150"]
attempt_date: 2026-06-14
my_difficulty: Medium
status: Done
time_min: 35
review_concepts: ["Linked List Reversal", "Dummy Head"]
---

# Reverse Nodes in k-Group

*Modularize linked list manipulations: strictly separate boundary detection from the actual reversal logic to avoid spaghetti pointers.*

## My Approach

I used an iterative approach with a dummy head to anchor the new linked list, avoiding edge cases when returning the final result. To keep the state management clean and prevent losing references, I decoupled the problem into two distinct helper functions: one to find the tail of the current $k$-group (`find_tail`), and another to execute a standard reversal between two boundaries (`reverse_group`).

The core loop tracks the tail of the previously processed group (`prev_group`) and the start of the next pending group (`next_unprocessed`). Inside the loop, I search for the $k$-th node. If found, I isolate that segment, reverse it, and update the outer pointers to bridge the newly reversed group with the rest of the list.

A critical design choice was initializing `new_head` and `new_tail` to `None` at the top of the loop. If the remaining nodes fall short of $k$, the logic gracefully bypasses the reversal, maps the untouched remainder to `new_head`, and connects it to the previous group without muddying variable states.

## Complexity

| | Complexity | Why |
|---|---|---|
| Time | $O(N)$ | Each node is visited exactly twice: once to measure the group boundary, and once during the pointer reversal. |
| Space | $O(1)$ | Reversals occur entirely in-place utilizing a constant number of pointer variables, avoiding recursion overhead. |

## Key Insight

Complex pointer manipulation becomes manageable when you isolate sub-tasks and strictly define your boundaries before executing mutations. The actual reversal of a linked list segment is trivial; the danger lies in losing the connections to the preceding and succeeding segments. By identifying four critical nodes upfront—the tail of the prior group, the head of the current group, the tail of the current group, and the head of the next group—you can execute the localized reversal safely and seamlessly patch the surrounding pointers together.

## Mistakes / Gaps

1. **Instantiation Syntax:** Assigned `dummy = ListNode` instead of `dummy = ListNode()`, which references the class itself rather than creating a new node object.
2. **Missing Return Statement:** Forgot to explicitly return `h` at the end of the `find_tail` helper after traversing.
3. **Stale State:** Neglected to update the `prev_group` pointer at the end of the main loop, which breaks the chain linkage for subsequent $k$-groups.
4. **Variable Confusion:** `new_head` and `new_tail` suffered from semantic drift during execution; fixed by explicitly resetting them to `None` at the start of each iteration to enforce a clean state.
5. **Off-by-One Error:** Used `range(k)` instead of `range(k - 1)` in `find_tail`, which incorrectly advanced the pointer one node past the intended $k$-group boundary.

## Code

```python
class Solution:
    def reverseKGroup(self, head: Optional[ListNode], k: int) -> Optional[ListNode]:
        def find_tail(h):
            for _ in range(k - 1):
                if not h:
                    return None
                h = h.next
            return h
        
        def reverse_group(h, t):
            cur = h
            new_head = None
            end = t.next

            while cur != end:
                temp = cur.next
                cur.next = new_head
                new_head = cur
                cur = temp

        dummy = ListNode()
        prev_group = dummy
        next_unprocessed = head

        while next_unprocessed:
            tail = find_tail(next_unprocessed)
            new_head = None
            new_tail = None

            if tail:
                new_head = tail
                new_tail = next_unprocessed
                next_unprocessed = tail.next
                reverse_group(new_tail, new_head)
            else:
                new_head = next_unprocessed
                next_unprocessed = None
            
            prev_group.next = new_head
            prev_group = new_tail

        return dummy.next
```
