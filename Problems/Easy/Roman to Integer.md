#easy 
# Roman to Integer

---

## Problem Statement

Given a string `s` representing a valid Roman numeral, convert it to an integer.

```
"III"     →    3
"LVIII"   →   58     (L=50, V=5, III=3)
"MCMXCIV" → 1994     (M=1000, CM=900, XC=90, IV=4)
```

**Constraints:** `1 <= s.length <= 15`, valid Roman numeral in range `[1, 3999]`

---

## The Core Rule

Roman numerals are written largest to smallest left to right — you add each symbol's value. The only exception is the six subtractive cases:

|Subtractive Pair|Value|
|---|---|
|IV|4|
|IX|9|
|XL|40|
|XC|90|
|CD|400|
|CM|900|

The rule that governs all six cases is identical: **when a smaller value appears directly before a larger value, subtract the smaller one instead of adding it.**

---

## Approach Analysis

### Approach 1 — Subtractive Pair Replacement — O(1) time, O(1) space

Replace all six subtractive pairs in the string before processing (`"IV"` → `"IIII"`, `"IX"` → `"VIIII"`, etc.), eliminating the exception entirely. Then simply sum every character. Clever but brittle — invents non-standard representations and only works because the input range is bounded and fixed.

### Approach 2 — Left-to-Right with Lookahead — O(n) time, O(1) space

Scan left to right. At each character, peek at the next one. If the current value is less than the next value, subtract the current. Otherwise add it. Clean, readable, directly models the rule as stated.

### Approach 3 — Right-to-Left Accumulation — O(n) time, O(1) space ✅ Optimal

Scan right to left, keeping track of the previous (rightmost processed) value. If the current symbol's value is less than the previous, it is a subtractive prefix — subtract it. Otherwise add it. Eliminates the lookahead entirely and handles the rule with a single comparison per character. One pass, no index arithmetic.

---

## Complexity

|Approach|Time|Space|
|---|---|---|
|Pair replacement|O(1)*|O(1)|
|Left-to-right lookahead|O(n)|O(1)|
|Right-to-left accumulation|O(n)|O(1)|

*O(1) only because the input is bounded to 15 characters and 6 fixed pairs.

---

## Solution

### Approach 2 — Left-to-Right with Lookahead

```python
def romanToInt(s: str) -> int:
    values = {
        'I': 1, 'V': 5, 'X': 10, 'L': 50,
        'C': 100, 'D': 500, 'M': 1000
    }

    result = 0

    for i in range(len(s)):
        curr = values[s[i]]
        next_val = values[s[i + 1]] if i + 1 < len(s) else 0

        if curr < next_val:
            result -= curr
        else:
            result += curr

    return result
```

### Approach 3 — Right-to-Left Accumulation (Optimal)

```python
def romanToInt(s: str) -> int:
    values = {
        'I': 1, 'V': 5, 'X': 10, 'L': 50,
        'C': 100, 'D': 500, 'M': 1000
    }

    result = 0
    prev = 0

    for char in reversed(s):
        curr = values[char]

        if curr < prev:
            result -= curr
        else:
            result += curr

        prev = curr

    return result
```

---

## Algorithm Walkthrough — `"MCMXCIV"` → 1994

### Right-to-Left Pass

```
String reversed: V I C X M C M

char=V  curr=5    prev=0    5 >= 0   → add   result=5     prev=5
char=I  curr=1    prev=5    1 <  5   → sub   result=4     prev=1
char=C  curr=100  prev=1    100 >= 1 → add   result=104   prev=100
char=X  curr=10   prev=100  10 < 100 → sub   result=94    prev=10
char=M  curr=1000 prev=10   1000>=10 → add   result=1094  prev=1000
char=C  curr=100  prev=1000 100<1000 → sub   result=994   prev=100
char=M  curr=1000 prev=100  1000>=100→ add   result=1994  prev=1000

Output: 1994 ✅
```

The subtractive pairs resolve themselves: when `I` is encountered with `V` already processed (`prev=5`), the condition `1 < 5` fires and `I` is subtracted. No special-case logic required.

---

## Python Concepts Dissected

**Dictionary literal with multiple entries**

```python
values = {
    'I': 1, 'V': 5, 'X': 10, 'L': 50,
    'C': 100, 'D': 500, 'M': 1000
}
```

A dictionary maps keys to values. Here each Roman symbol (a string of length 1) maps to its integer value. Lookup `values['X']` returns `10` in O(1) average time. The dictionary is defined once outside the loop — building it inside the loop would recreate it on every iteration unnecessarily.

---

**`reversed(s)`**

`reversed()` is a built-in that returns an iterator traversing a sequence from right to left. For strings, it yields one character at a time in reverse order. It does not create a reversed copy of the string — it is lazy, producing each character only when requested by the `for` loop. This makes it O(1) in space.

Note: `s[::-1]` also reverses a string but creates a full copy in memory. `reversed(s)` is the idiomatic choice when you only need to iterate.

---

**`for char in reversed(s)`**

Iterating directly over the reversed iterator. `char` receives one character per iteration, right to left. No index needed, which is cleaner than `for i in range(len(s) - 1, -1, -1)`.

---

**`curr = values[char]`**

Dictionary lookup by key. `char` is a single character string like `'M'` or `'I'`. This returns its integer value from the map. If `char` were not a key in `values`, Python would raise a `KeyError` — but the problem guarantees only valid Roman characters, so this is safe.

---

**`if curr < prev`**

The entire subtractive rule in one comparison. Because we scan right to left, `prev` always holds the value of the symbol to the right of the current one. If the current symbol is smaller than what came after it, it must be a subtractive prefix. This single condition replaces any need to check specific pairs like `"IV"` or `"CM"`.

---

**`result -= curr` / `result += curr`**

Compound assignment operators. `result -= curr` is shorthand for `result = result - curr`. Python evaluates the right side fully before assigning, so this is safe even when `result` and `curr` are the same variable.

---

**`prev = curr`**

State carried between iterations. After processing the current character, it becomes the "previous" for the next iteration (which moves one step further left). This single variable replaces the need for index-based lookahead.

---

**Lookahead variant — conditional expression**

```python
next_val = values[s[i + 1]] if i + 1 < len(s) else 0
```

A conditional expression (Python's ternary operator): `value_if_true if condition else value_if_false`. When `i` is the last index, `i + 1` would be out of bounds — defaulting `next_val` to `0` handles this gracefully. Since no Roman symbol has value `0`, the condition `curr < 0` can never trigger, making the last character always get added. This is boundary guard logic expressed in one line.

---

## Pattern to Internalize

This problem introduces two reusable ideas. The first is **encoding rules as a lookup table** — instead of a chain of `if/elif` statements for each symbol, a dictionary maps every case in O(1). Whenever a problem involves translating a fixed set of symbols or tokens to values, a dictionary is the right tool.

The second is **right-to-left scanning to eliminate lookahead**. In the left-to-right pass, you must always peek at the next character to decide what to do with the current one. Reversing the scan direction converts that forward dependency into a backward one — `prev` is already known at every step. This technique of changing scan direction to simplify state management appears in other problems involving pairs, dependencies, or context from adjacent elements.