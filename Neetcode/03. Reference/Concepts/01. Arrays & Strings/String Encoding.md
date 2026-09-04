---
type: concept
tags: [concept, dsa]
---

# String Encoding

**TL;DR:** Serialize structured data into one unambiguous string — length-prefix each piece so the decoder always knows exactly where it ends, even if the payload contains any character.

## When to reach for it
- Need to pack a list of strings (or a tree/graph) into a single string for transmission or storage, then reconstruct the original exactly.
- The payload can contain *any* character, including whatever you'd naturally pick as a delimiter (comma, `#`, null byte) — a plain-delimiter scheme would be ambiguous.
- "Serialize and deserialize" appears directly in the problem statement.

## How it works
The standard trick is length-prefixed framing: before each chunk of payload, write its length followed by a separator that can't be mistaken for part of a number, e.g. `<len>#<payload>`. Trace encoding `["ab", "c#d", ""]`:

| string | length | encoded chunk |
|---|---|---|
| `"ab"` | 2 | `2#ab` |
| `"c#d"` | 3 | `3#c#d` |
| `""` | 0 | `0#` |

Concatenated: `"2#ab3#c#d0#"`. Decoding reverses it: read digits until `#`, that number tells you exactly how many characters to consume next as the payload — regardless of what those characters are (even more `#` or digits), because the length was fixed *before* reading them. For **serializing a binary tree**, the same idea applies structurally instead of character-by-character: preorder-traverse the tree, emitting a sentinel (e.g. `"N"`) for every `None` child, so the sequence of values and sentinels alone is enough to reconstruct the exact shape on decode.

## Why it works
The core problem with a plain delimiter (like splitting on `,`) is that the delimiter isn't reserved — if the payload can contain a comma, you can no longer tell whether a comma is a separator or data. Length-prefixing sidesteps this entirely by never *searching* for the end of a chunk inside untrusted content: the length is read first, from a small, constrained alphabet (digits plus one fixed separator), and then exactly that many raw characters are consumed unconditionally. Nothing about the payload's content is ever interpreted as structural, so it can be arbitrary. For tree serialization, the invariant is that preorder traversal plus explicit "null" markers records *both* the value at each node *and* the shape of the tree (which children are missing) — together sufficient to rebuild the tree uniquely, whereas a preorder sequence without null markers is ambiguous about shape.

## Template
```python
# Length-prefixed encode/decode for a list of strings
def encode(strs):
    return "".join(f"{len(s)}#{s}" for s in strs)

def decode(s):
    result, i = [], 0
    while i < len(s):
        j = i
        while s[j] != "#":
            j += 1
        length = int(s[i:j])
        result.append(s[j + 1 : j + 1 + length])
        i = j + 1 + length
    return result

# Tree serialization (preorder + null sentinels)
def serialize(root):
    vals = []
    def dfs(node):
        if not node:
            vals.append("N")
            return
        vals.append(str(node.val))
        dfs(node.left)
        dfs(node.right)
    dfs(root)
    return ",".join(vals)
```

## Complexity
Time: O(n) to encode or decode — every character (or node) is visited a constant number of times. Space: O(n) for the encoded string (or the traversal's recursion stack for trees).

## Common pitfalls
- Using a plain delimiter (comma, space) without escaping when the payload can legally contain that character — silently corrupts round-tripping.
- Off-by-one when slicing out the payload after reading the length (forgetting to skip the separator character itself).
- Forgetting null sentinels when serializing a tree — without them, the shape (which side a child is missing on) can't be recovered from values alone.
- Assuming a fixed-width length field is enough — payloads longer than that width silently truncate; a delimited length (`#`) has no such cap.

## NeetCode examples
- [[08.EncodeAndDecodeStrings|EncodeAndDecodeStrings]] — canonical length-prefixed framing
- [[15.SerializeAndDeserializeBinaryTree|SerializeAndDeserializeBinaryTree]] — preorder traversal with null sentinels

## Full guide
- [[Job Search/Neetcode/01. Questions/01. Arrays & Hashing/0.ArraysAndHashingGuide|Arrays & Hashing Guide]]
- [[Job Search/Neetcode/01. Questions/07. Tree/0.TreeGuide|Tree Guide]]
