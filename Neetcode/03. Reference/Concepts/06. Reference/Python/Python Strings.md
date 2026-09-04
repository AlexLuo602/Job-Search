---
type: concept
tags: ["concept"]
---

# Python Strings

Python strings are **immutable**. Any "modification" creates a new string — repeated `+=` in a loop is O(n²). Always build via a list then `"".join()`.

## Slicing `s[start:stop:step]` — O(k)

```python
s[::-1]          # reverse entire string
s[1:-1]          # strip first and last character
s[::2]           # every other character
s[i:j]           # substring [i, j)
```

→ [[01.ValidPalindrome|ValidPalindrome]]

## `s.split(sep=None, maxsplit=-1)` — O(n)

```python
"a b  c".split()          # ['a', 'b', 'c']  — splits on ANY whitespace, strips leading/trailing
"a,b,c".split(",")        # ['a', 'b', 'c']
"a,b,c".split(",", 1)     # ['a', 'b,c']     — maxsplit limits the number of splits
```

No-arg split is cleaner than `.split(" ")` — it handles multiple spaces and newlines.

## `sep.join(iterable)` — O(n)

```python
" ".join(["hello", "world"])   # "hello world"
"".join(char_list)             # rebuild string from list — O(n), not O(n²)
",".join(str(x) for x in nums)
```

**Always** use join over repeated `+=`. → [[08.EncodeAndDecodeStrings|EncodeAndDecodeStrings]]

## `s.strip(chars=None)` / `.lstrip()` / `.rstrip()` — O(n)

```python
"  hello  ".strip()        # "hello"
"xxhelloxx".strip("x")    # "hello"  — strips any char in the string, not the substring
```

## `s.find(sub, start=0, end=len(s))` — O(n)

Returns the **index** of the first occurrence, or `-1` if not found. `s.index()` does the same but raises `ValueError`. Use `find` when absence is a valid case.

```python
s.find("ab")          # first occurrence
s.find("ab", i)       # search starting at index i
```

## `s.count(sub, start=0, end=len(s))` — O(n)

Non-overlapping count of `sub` in `s`.

## `s.replace(old, new, count=-1)` — O(n)

```python
s.replace("a", "b")       # replace all
s.replace("a", "b", 2)    # replace first 2 only
```

## Character checks — O(n)

```python
s.isalpha()    # all letters (no digits, spaces, punctuation)
s.isdigit()    # all digit characters
s.isalnum()    # letters or digits — use this for palindrome filter
s.islower() / s.isupper()
s.lower() / s.upper()   # new string, O(n)
```

→ [[01.ValidPalindrome|ValidPalindrome]] (filter with `isalnum`, compare lowercased)

## `ord(c)` / `chr(i)` — O(1)

```python
ord('a')          # 97
ord(c) - ord('a') # 0-25 index for lowercase letters — use for 26-element freq arrays
chr(97)           # 'a'
```

→ [[02.ValidAnagram|ValidAnagram]], [[04.GroupAnagrams|GroupAnagrams]]

## `s.startswith(prefix)` / `s.endswith(suffix)` — O(k)

Both accept a **tuple** of options: `s.startswith(("http", "https"))`.

## `s.zfill(width)` — O(n)

Pads with leading zeros. Useful for binary representation formatting.
