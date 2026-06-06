#easy
# Palindrome Number

---

## Problem Statement

Given an integer `x`, return `true` if `x` is a palindrome — a number that reads the same forwards and backwards — and `false` otherwise.

```
x =  121  →  True    (121 reversed is 121)
x = -121  →  False   (reversed reads 121-)
x =   10  →  False   (reversed reads 01)
```

**Constraints:** `-2³¹ <= x <= 2³¹ - 1`

---

## Immediate Observations — Edge Cases First

Before any algorithm, three categories resolve instantly:

**Negative numbers are never palindromes.** The `-` sign has no mirror. Any `x < 0` is `False` immediately.

**Numbers ending in 0 are never palindromes** (except `x == 0` itself). The first digit of any positive integer cannot be 0, so if the last digit is 0, the number can never mirror itself. `x % 10 == 0 and x != 0` → `False` immediately.

**Single-digit numbers are always palindromes.** `0` through `9` trivially read the same both ways.

---

## Approach Analysis

### Approach 1 — String Conversion — O(d) time, O(d) space

Convert the integer to a string, reverse it using slicing, compare the two. Python makes this a one-liner. Readable and correct, but allocates O(d) extra space where `d` is the number of digits. The follow-up challenge asks to avoid string conversion entirely.

### Approach 2 — Full Integer Reversal — O(d) time, O(1) space

Reconstruct the reversed integer mathematically using modulo and integer division, then compare against the original. No strings. However, reversing the entire number is more work than necessary — you only need half the digits to confirm symmetry.

### Approach 3 — Half Reversal — O(d/2) time, O(1) space ✅ Optimal

The key insight: a palindrome's second half, reversed, equals its first half. So strip digits from the right and accumulate them into a reversed half — stop when the reversed portion has grown as large as the remaining left portion. At that point you have enough information to confirm or deny the palindrome without ever processing the full number. This also handles even and odd digit counts naturally without needing to know the length upfront.

---

## Complexity

| Approach              | Time   | Space |
| --------------------- | ------ | ----- |
| String conversion     | O(d)   | O(d)  |
| Full integer reversal | O(d)   | O(1)  |
| Half reversal         | O(d/2) | O(1)  |

---

## Solution

### Approach 1 — String Conversion

```python
def isPalindrome(x: int) -> bool:
    if x < 0:
        return False
    s = str(x)
    return s == s[::-1]
```

### Approach 3 — Half Reversal (Optimal)

```python
def isPalindrome(x: int) -> bool:
    if x < 0 or (x % 10 == 0 and x != 0):
        return False

    reversed_half = 0

    while x > reversed_half:
        reversed_half = reversed_half * 10 + x % 10
        x //= 10

    return x == reversed_half or x == reversed_half // 10
```

---

## Algorithm Walkthrough — x = 1221

```
Start:  x = 1221,  reversed_half = 0

Step 1: last digit = 1221 % 10 = 1
        reversed_half = 0 * 10 + 1 = 1
        x = 1221 // 10 = 122

Step 2: last digit = 122 % 10 = 2
        reversed_half = 1 * 10 + 2 = 12
        x = 122 // 10 = 12

Loop ends: x (12) is no longer > reversed_half (12)

Check: x == reversed_half → 12 == 12 → True ✅
```

## Algorithm Walkthrough — x = 12321 (odd digits)

```
Start:  x = 12321,  reversed_half = 0

Step 1: last digit = 1  →  reversed_half = 1,    x = 1232
Step 2: last digit = 2  →  reversed_half = 12,   x = 123
Step 3: last digit = 3  →  reversed_half = 123,  x = 12

Loop ends: x (12) is no longer > reversed_half (123)

Check (odd): x == reversed_half // 10
             12 == 123 // 10
             12 == 12 → True ✅

The middle digit (3) is discarded via // 10 — it has no mirror.
```

---

## Python Concepts Dissected

**Early return with compound condition**

```python
if x < 0 or (x % 10 == 0 and x != 0):
    return False
```

`or` short-circuits — if `x < 0` is `True`, Python never evaluates the right side. The parentheses around the second condition are not strictly required but make the precedence explicit and the intent readable. `%` is the modulo operator: `x % 10` extracts the last digit of any integer.

---

**`while x > reversed_half`**

The loop condition is the termination logic of the algorithm. When `reversed_half` has grown to meet or exceed `x`, exactly half the digits have been processed. No need to count digits or know the number's length in advance.

---

**`reversed_half * 10 + x % 10`**

This is how you build an integer digit by digit. Multiplying by 10 shifts all existing digits one place left, making room for the new digit. `x % 10` extracts the rightmost digit. This pattern — `accumulator = accumulator * 10 + next_digit` — appears in many integer manipulation problems.

---

**`x //= 10`**

`//` is floor division — **integer division that discards the remainder**. `1221 // 10 = 122`. The `//=` form is compound assignment: equivalent to `x = x // 10`. This removes the rightmost digit from `x` each iteration.

---

**`return x == reversed_half or x == reversed_half // 10`**

Two cases handled in one line:

- `x == reversed_half` — even number of digits. Both halves are equal in length. Example: `1221` → left half `12`, reversed right half `12`.
- `x == reversed_half // 10` — odd number of digits. The middle digit ended up in `reversed_half`, so we strip it with `// 10` before comparing. Example: `12321` → left half `12`, reversed right half `123`, discard middle → `12`.

---

**`s[::-1]` (string approach)**

Python slice notation: `s[start:stop:step]`. When `start` and `stop` are omitted, the full string is taken. A `step` of `-1` traverses the string in reverse. This is the idiomatic Python one-liner for string reversal.

---

## Pattern to Internalize

Two patterns are exercised here. The first is **early elimination via edge cases** — identifying inputs that can be answered before any real computation, which simplifies the remaining logic significantly. The second is **digit extraction via modulo and floor division** — `x % 10` extracts the last digit, `x // 10` removes it. These two operations together let you process an integer one digit at a time without converting it to a string, a technique that recurs across many integer manipulation problems.