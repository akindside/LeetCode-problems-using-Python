# Longest Common Prefix

---

## Problem Statement

Given an array of strings `strs`, return the longest string that is a prefix of every string in the array. If no common prefix exists, return `""`.

```
["flower", "flow", "flight"]  →  "fl"
["dog", "racecar", "car"]     →  ""
```

**Constraints:** `1 <= strs.length <= 200`, `0 <= strs[i].length <= 200`, lowercase English letters only.

---

## Immediate Observations

A prefix must be shared by **every** string in the array, not just a majority. This means the answer is bounded above by the shortest string — no prefix can be longer than the shortest element. It also means any single mismatch at a given position across any string ends the prefix there.

Edge cases that resolve immediately:

- Array of length 1 — the single string is its own prefix, return it directly.
- Any empty string in the array — the common prefix must be `""` immediately, since the empty string shares no characters with anything.

---

## Approach Analysis

### Approach 1 — Horizontal Scanning — O(S) time, O(1) space

Take the first string as the initial candidate prefix. Compare it against the second string, trimming the candidate from the right until it matches the start of the second string. Repeat with each subsequent string. The candidate can only shrink. If it ever becomes empty, return immediately.

`S` is the total number of characters across all strings. In the worst case (all strings identical) every character of every string is visited.

### Approach 2 — Vertical Scanning — O(S) time, O(1) space

Instead of comparing full strings, scan column by column — character at position 0 across all strings, then position 1, and so on. Stop the moment any string is exhausted or a mismatch is found. This is more efficient when the common prefix is short and the strings are long, because it exits as soon as the first mismatch is found at any position, without ever touching the rest of each string.

### Approach 3 — Sort and Compare Endpoints — O(n log n) time, O(1) space

Sort the array lexicographically. After sorting, the most dissimilar strings are the first and last — if any prefix is shared by all strings, it must be shared by these two extremes. Compare only the first and last string character by character. Simple and elegant, but the sort costs O(n log n), making it theoretically worse despite being concise in code.

### Approach 4 — Divide and Conquer — O(S) time, O(m log n) space

Split the array in half recursively, find the common prefix of each half, then find the common prefix of the two results. Correct and educational for understanding recursive decomposition, but the O(m log n) stack space (where `m` is the length of the shortest string) is a real cost with no practical benefit over vertical scanning.

---

## Complexity

|Approach|Time|Space|Notes|
|---|---|---|---|
|Horizontal scanning|O(S)|O(1)|Slow when prefix is short, strings are long|
|Vertical scanning|O(S)|O(1)|Early exit on first column mismatch|
|Sort + compare endpoints|O(n log n)|O(1)|Sort dominates; elegant but suboptimal|
|Divide and conquer|O(S)|O(m log n)|Recursive stack overhead, no practical gain|

---

## Solution

### Approach 1 — Horizontal Scanning

```python
def longestCommonPrefix(strs: list[str]) -> str:
    prefix = strs[0]

    for s in strs[1:]:
        while not s.startswith(prefix):
            prefix = prefix[:-1]
            if not prefix:
                return ""

    return prefix
```

### Approach 2 — Vertical Scanning (Optimal)

```python
def longestCommonPrefix(strs: list[str]) -> str:
    for i in range(len(strs[0])):
        char = strs[0][i]

        for s in strs[1:]:
            if i == len(s) or s[i] != char:
                return strs[0][:i]

    return strs[0]
```

### Approach 3 — Sort and Compare Endpoints

```python
def longestCommonPrefix(strs: list[str]) -> str:
    strs.sort()
    first, last = strs[0], strs[-1]

    for i, char in enumerate(first):
        if char != last[i]:
            return first[:i]

    return first
```

---

## Algorithm Walkthrough — `["flower", "flow", "flight"]`

### Vertical Scanning

```
Reference string: "flower"

i=0  char='f'
     "flow"[0]   = 'f'  ✓
     "flight"[0] = 'f'  ✓

i=1  char='l'
     "flow"[1]   = 'l'  ✓
     "flight"[1] = 'l'  ✓

i=2  char='o'
     "flow"[2]   = 'o'  ✓
     "flight"[2] = 'i'  ✗  → return strs[0][:2] = "fl"

Output: "fl" ✅
```

### Horizontal Scanning

```
prefix = "flower"

Compare with "flow":
  "flow".startswith("flower")? No
  prefix = "flowe"
  "flow".startswith("flowe")? No
  prefix = "flow"
  "flow".startswith("flow")? Yes ✓

Compare with "flight":
  "flight".startswith("flow")? No
  prefix = "flo"
  "flight".startswith("flo")? No
  prefix = "fl"
  "flight".startswith("fl")? Yes ✓

Output: "fl" ✅
```

---

## Python Concepts Dissected

**`strs[1:]` — list slicing**

```python
for s in strs[1:]:
```

Slice notation `[start:stop]` returns a new list from index `start` up to (not including) `stop`. Omitting `stop` goes to the end. `strs[1:]` is therefore every element except the first. This is how you skip the reference element when iterating, without needing an index or a counter. Slicing creates a shallow copy — the strings themselves are not duplicated, only the list structure.

---

**`s.startswith(prefix)` — string method**

`startswith()` returns `True` if the string begins with the given argument, `False` otherwise. It is equivalent to `s[:len(prefix)] == prefix` but more readable and marginally faster since it is implemented in C internally. It also accepts a tuple of prefixes to check multiple at once, though that feature is not needed here.

---

**`prefix[:-1]` — negative indexing in slices**

Negative indices count from the end of a sequence. Index `-1` refers to the last element. In a slice, `[:-1]` means "from the start up to but not including the last character" — effectively trimming one character from the right. This is the standard Python idiom for shrinking a string one character at a time from its tail.

---

**`range(len(strs[0]))` — iterating by index**

In vertical scanning the outer loop must iterate by position, not by value, because the same index `i` is applied to every string simultaneously. `range(len(strs[0]))` generates integers from `0` up to the length of the reference string minus one. The reference string is `strs[0]` — its length is the upper bound since the prefix cannot be longer than any single string in the array.

---

**`i == len(s)` — exhaustion check**

```python
if i == len(s) or s[i] != char:
```

When `i` equals the length of string `s`, index `i` is out of bounds — `s[i]` would raise an `IndexError`. The exhaustion check `i == len(s)` must appear before `s[i]` in the condition. Python evaluates `or` conditions left to right and **short-circuits** — if the left side is `True`, the right side is never evaluated. This ordering is not stylistic preference; reversing the two conditions would cause a runtime error on strings shorter than the reference.

---

**`strs[0][:i]` — prefix extraction**

```python
return strs[0][:i]
```

`[:i]` returns all characters from index `0` up to but not including index `i`. When a mismatch is found at column `i`, the common prefix is everything before that column — characters at indices `0` through `i-1`. When `i=0`, this returns `""`, the empty string, which correctly handles the case where the very first column has no match.

---

**`strs.sort()` — in-place sort**

`list.sort()` sorts the list in place and returns `None`. This is distinct from `sorted(strs)`, which returns a new sorted list without modifying the original. Lexicographic (alphabetical) sort is the default for strings — `"apple"` < `"banana"` because `'a'` < `'b'` in Unicode. For the sort-and-compare approach, lexicographic order guarantees that the maximum character-level distance between any two strings in the array exists between the first and last element after sorting.

---

**`enumerate(first)` — index and value together**

```python
for i, char in enumerate(first):
```

`enumerate()` yields `(index, value)` pairs. Used here to iterate over characters of the first string while simultaneously having the index `i` available for `last[i]`. Without `enumerate`, you would write `for i in range(len(first)): char = first[i]` — functionally identical but less idiomatic.

---

## Pattern to Internalize

This problem is a direct application of **prefix invariant maintenance** — the answer is initialized to a candidate and then progressively restricted by each new piece of evidence. Every string in the array is a constraint that can only shrink the candidate, never grow it. The moment the candidate becomes empty, no further work is needed. This pattern — initialize to the most optimistic answer, tighten it with each constraint, exit early when the bound is exhausted — appears across a wide class of problems involving shared properties across a collection.

The secondary insight is the **column-first vs row-first traversal tradeoff**. Horizontal scanning processes one full string at a time (row-first). Vertical scanning processes one column at a time. When the answer is short, vertical scanning exits in the first few columns without ever examining the bulk of any string. Choosing the right traversal order based on where the answer is likely to be found is a recurring optimization consideration.