---
type: concept
tags: ["concept"]
---

# Python Dictionaries and Sets

## dict / defaultdict / Counter

### Core dict operations — O(1) average

```python
d.get(key, default=None)      # safe lookup — never raises KeyError
d.setdefault(key, default)    # inserts default if key absent, then returns value
d.pop(key, default)           # remove and return; default avoids KeyError
key in d                      # O(1) membership
d.items() / d.keys() / d.values()  # O(1) view objects, O(n) to iterate
```

### `collections.defaultdict` — O(1)

```python
from collections import defaultdict

d = defaultdict(int)     # missing keys return 0
d = defaultdict(list)    # missing keys return []
d = defaultdict(set)     # missing keys return set()
d[key] += 1              # no KeyError on first access
```

Use over `d.get(k, 0) + 1` when you'll update the same keys repeatedly. → [[04.GroupAnagrams|GroupAnagrams]], [[05.TopKElements|TopKElements]]

### `collections.Counter` — O(n) to build

```python
from collections import Counter

c = Counter("aabbbc")        # Counter({'b': 3, 'a': 2, 'c': 1})
c = Counter([1, 1, 2, 3])

c.most_common(k)             # O(n log k) — [(elem, count), ...] top k
c.most_common()              # all, sorted by count desc

c.update(iterable)           # add counts — O(k)
c.subtract(iterable)         # subtract counts (can go negative)

# Arithmetic on Counters
c1 + c2     # add counts
c1 - c2     # subtract, keep only positives
c1 & c2     # element-wise min (intersection)
c1 | c2     # element-wise max (union)

+c          # remove zero and negative counts
```

→ [[02.ValidAnagram|ValidAnagram]], [[04.GroupAnagrams|GroupAnagrams]], [[04.PermutationInString|PermutationInString]], [[05.TopKElements|TopKElements]]

### Sorting dict by value

```python
sorted(d.items(), key=lambda x: x[1], reverse=True)
# or
sorted(d, key=d.get, reverse=True)   # sort keys by their values
```

---

## set

All core operations O(1) average.

```python
s = set()      # or literal: s = {1, 2, 3}
s.add(x)
s.remove(x)    # raises KeyError if absent
s.discard(x)   # silent if absent — prefer this
x in s         # O(1)

s & t          # intersection — O(min(len(s), len(t)))
s | t          # union
s - t          # difference (elements in s not in t)
s ^ t          # symmetric difference

s.issubset(t) / s.issuperset(t)

frozenset(iterable)   # hashable set — usable as dict key or element of another set
```

→ [[09.LongestConsecutiveSequence|LongestConsecutiveSequence]] (set membership to find sequence starts)
