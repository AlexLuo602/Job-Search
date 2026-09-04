---
type: concept
tags: ["concept"]
---

# Bit Manipulation Tricks

**TL;DR:** Use bitwise operators to perform arithmetic and set operations in O(1) with no extra space.

## When to use
- Find the single non-duplicate element (XOR cancellation).
- Count set bits or check power-of-two.
- Generate all subsets of a set (bitmask enumeration).
- Space/time-efficient manipulation of boolean flags.

## Template
```python
# Common tricks
x & (x - 1)          # clear lowest set bit (0 if x is power of 2)
x & (-x)             # isolate lowest set bit
x ^ x == 0           # XOR self → 0  (use to find single number)
bin(x).count('1')    # count set bits (or use bit_count() in Python 3.10+)
x >> k               # divide by 2^k
x << k               # multiply by 2^k
~x                   # bitwise NOT (= -x - 1 in Python two's complement)

# XOR to find single number
result = 0
for n in nums:
    result ^= n
return result         # all pairs cancel; unpaired value remains

# Enumerate all subsets of [0..n-1]
for mask in range(1 << n):
    subset = [i for i in range(n) if mask & (1 << i)]
```

## Key idea / invariant
XOR is its own inverse: `a ^ a == 0` and `a ^ 0 == a`. So XOR-ing all elements cancels every even-occurrence value, leaving the odd-occurrence one. Bit tricks work because integers are stored in binary — operations that look O(1) at the integer level process all bits simultaneously.

## Complexity
Time: O(1) per bitwise operation on fixed-width integers | Space: O(1)

## Common pitfalls
- Python integers are arbitrary-precision — bit NOT (`~x`) gives `-(x+1)`, not a 32-bit flip. Use `x ^ 0xFFFFFFFF` for 32-bit NOT.
- Right-shift on negative numbers is arithmetic (sign-extending) in Python; use `& 0xFFFFFFFF` to mask to 32 bits.
- Operator precedence: unlike C, Python's `&`/`^`/`|` bind *tighter* than `==`, so `x & 1 == 0` parses as `(x & 1) == 0`. The real Python trap is shifts vs. arithmetic: `+`/`-` bind tighter than `<<`, so `1 << n - 1` means `1 << (n - 1)` — write `(1 << n) - 1`.

## NeetCode examples
- [[01.SingleNumber|SingleNumber]] — XOR all elements; pairs cancel
- [[02.NumberOf1Bits|NumberOf1Bits]] — count set bits with `n & (n-1)` loop
- [[05.MissingNumber|MissingNumber]] — XOR indices and values; duplicate cancels, missing remains
- [[06.SumOfTwoIntegers|SumOfTwoIntegers]] — add without `+` using XOR (sum) and AND (carry)
- [[03.CountingBits|CountingBits]] — `dp[i] = dp[i >> 1] + (i & 1)`

## Full guide
[[Job Search/Neetcode/01. Questions/18. Bit Manipulation/0.BitManipulationGuide|Bit Manipulation Guide]]
