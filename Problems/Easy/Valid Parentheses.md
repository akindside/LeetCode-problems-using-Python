# Valid Parentheses

---

## Problem Statement

Given a string `s` containing only `'('`, `')'`, `'{'`, `'}'`, `'['`, `']'`, determine if the string is valid. A string is valid if every opening bracket is closed by the same type of bracket, in the correct order, with no unmatched brackets remaining.

```
"()"      →  True
"()[]{}"  →  True
"(]"      →  False
"([])"    →  True
"([)]"    →  False
```

**Constraints:** `1 <= s.length <= 10⁴`, characters are `()[]{}` only.

---

## Immediate Observations

**Odd-length strings are never valid.** Every opening bracket requires exactly one closing bracket. A valid string must always have even length. Any odd-length input is `False` immediately.

**The last opened bracket must be the first closed.** This is the defining property of the problem. The structure is strictly nested — like Russian dolls. Whatever was opened most recently must be closed before anything opened earlier can be closed. This last-in-first-out ordering is the exact definition of a stack.

**An unmatched closing bracket is immediately invalid.** If a `]` appears and the most recently opened bracket was `(`, the string is invalid on the spot — no future characters can fix it.

**A non-empty stack at the end is invalid.** If the string ends and there are still unclosed opening brackets in the stack, the string is invalid.

---

## Approach Analysis

### Approach 1 — Repeated Replacement — O(n²) time, O(n) space

Repeatedly replace every valid adjacent pair (`()`, `[]`, `{}`) with an empty string until no more replacements can be made. If the string is empty at the end, it was valid. Correct in principle, but each pass is O(n) and up to O(n) passes are needed — quadratic overall. Completely impractical for `n = 10⁴`.

### Approach 2 — Counter-Based — O(n) time, O(1) space

Use a counter that increments on `(` and decrements on `)`. Valid if the counter never goes negative and ends at zero. This works only for a single bracket type. It fails entirely for mixed types — `"([)]"` would pass a counter check but is invalid because of ordering.

### Approach 3 — Stack — O(n) time, O(n) space ✅ Optimal

Push every opening bracket onto a stack. When a closing bracket is encountered, check the top of the stack — if it matches, pop it; if it does not match or the stack is empty, return `False` immediately. After processing the full string, the string is valid if and only if the stack is empty. This is the canonical solution and directly models the nested structure of the problem.

---

## Complexity

| Approach             | Time  | Space | Notes                                |
| -------------------- | ----- | ----- | ------------------------------------ |
| Repeated replacement | O(n²) | O(n)  | Impractical at scale                 |
| Counter-based        | O(n)  | O(1)  | Fails for mixed bracket types        |
| Stack                | O(n)  | O(n)  | Optimal — models the problem exactly |

---

## Solution

```python
def isValid(s: str) -> bool:
    stack = []
    matching = {')': '(', ']': '[', '}': '{'}

    for char in s:
        if char in matching:
            if not stack or stack[-1] != matching[char]:
                return False
            stack.pop()
        else:
            stack.append(char)

    return not stack
```

---

## Algorithm Walkthrough

### `"([])"` → True

```
char='('  opening → stack: ['(']
char='['  opening → stack: ['(', '[']
char=']'  closing → matching[']'] = '['
          stack[-1] = '['  ✓  match → pop
          stack: ['(']
char=')'  closing → matching[')'] = '('
          stack[-1] = '('  ✓  match → pop
          stack: []

End: stack is empty → True ✅
```

### `"([)]"` → False

```
char='('  opening → stack: ['(']
char='['  opening → stack: ['(', '[']
char=')'  closing → matching[')'] = '('
          stack[-1] = '['  ✗  mismatch → return False ✅
```

### `"([]"` — unclosed bracket → False

```
char='('  opening → stack: ['(']
char='['  opening → stack: ['(', '[']
char=']'  closing → matching[']'] = '['
          stack[-1] = '['  ✓  match → pop
          stack: ['(']

End: stack is ['('] — not empty → False ✅
```

---

## Python Concepts Dissected

**`stack = []`**

Python has no dedicated stack data structure. A plain list is used as a stack by convention. The operations that define stack behaviour are `append()` for push and `pop()` for pop — both operate on the right end of the list, giving O(1) amortised time. The left end of the list represents the bottom of the stack, the right end represents the top.

---

**`matching = {')': '(', ']': '[', '}': '{'}`**

The mapping is defined closing-bracket → opening-bracket, not the other way around. This is a deliberate design choice. When a closing bracket is encountered, you need to immediately know what its expected partner is. Defining the map in this direction means `matching[char]` gives the required opening bracket in one lookup, eliminating any need for `if/elif` chains or a second dictionary.

---

**`if char in matching`**

`in` on a dictionary checks keys only. Since `matching` contains only the three closing brackets as keys, this condition is `True` exactly when `char` is a closing bracket and `False` when it is an opening bracket. The two branches of the loop are therefore: closing bracket (check and pop) and opening bracket (push). The full set of six characters is handled without any explicit enumeration.

---

**`stack[-1]`**

Negative indexing. Index `-1` refers to the last element of a list — the top of the stack — without needing to compute `stack[len(stack) - 1]`. This is the standard Python idiom for peeking at the top of a stack. It does not remove the element; that requires a separate `pop()`.

---

**`not stack or stack[-1] != matching[char]`**

Two failure conditions combined in one guard:

- `not stack` — the stack is empty. A closing bracket with nothing on the stack means there is no corresponding opener. Checking this first is mandatory — if the stack is empty and you evaluate `stack[-1]`, Python raises an `IndexError`. The `or` short-circuit ensures `stack[-1]` is never evaluated when the stack is empty.
- `stack[-1] != matching[char]` — the top of the stack is not the expected opener. The brackets are mismatched in type or order.

Either condition alone is sufficient to invalidate the string, so `or` is the correct operator.

---

**`stack.pop()`**

Removes and returns the last element of the list — the top of the stack. The return value is discarded here because the match was already confirmed by `stack[-1]` on the line above. `pop()` with no argument always operates on the last element, giving O(1) amortised time. `pop(0)` would remove the first element in O(n) time — never use that for a stack.

---

**`stack.append(char)`**

Adds `char` to the end of the list — the top of the stack. `append()` is O(1) amortised. The list grows dynamically as needed. Only opening brackets reach this branch, so only openers are ever stored on the stack.

---

**`return not stack`**

`not` applied to a list returns `True` if the list is empty and `False` if it contains any elements. An empty list is falsy in Python — `bool([]) == False`, `bool(['(']) == True`. So `not stack` is equivalent to `len(stack) == 0` but more idiomatic. This single line handles the case where the string was syntactically plausible throughout but left unclosed brackets at the end.

---

## Pattern to Internalize

This problem is the canonical introduction to the **stack as a matching structure**. Whenever a problem involves nested, ordered pairing — brackets, HTML tags, function call frames, undo history, expression evaluation — a stack is the natural model. The invariant is always the same: the most recently opened item must be the first to close. A stack enforces this invariant automatically because it only exposes its top element, which is always the most recently pushed item.

The secondary pattern here is **fail-fast validation**: every check is placed at the earliest possible point. The mismatch check fires the moment a closing bracket is seen, not at the end. The empty-stack check fires before any peek. The final empty-check covers everything that survived character-by-character processing. Structured this way, the algorithm returns `False` at the first sign of invalidity and never does unnecessary work.