# Remove Duplicates from Sorted Array

---

## Problem Statement

Given an integer array `nums` sorted in non-decreasing order, remove duplicates **in-place** so each unique element appears exactly once. Return `k` — the count of unique elements. The first `k` positions of `nums` must hold the unique values in sorted order. Everything beyond index `k-1` is ignored.

```
[1,1,2]               →  k=2,  nums=[1,2,_]
[0,0,1,1,1,2,2,3,3,4] →  k=5,  nums=[0,1,2,3,4,_,_,_,_,_]
```

**Constraints:** `1 <= nums.length <= 3 * 10⁴`, `-100 <= nums[i] <= 100`, sorted in non-decreasing order.

---

## Immediate Observations

**In-place means O(1) extra space.** No auxiliary array, no set, no copy of the input. The original array must be modified directly.

**The array is already sorted.** This is the critical property. All duplicates of any value are grouped together contiguously. A duplicate is always detected by comparing a value to its immediate left neighbor — no searching required.

**The first element is always unique.** There is nothing to its left to duplicate. The result always contains at least one element, and position `0` never needs to change.

**Only the first `k` positions matter.** The judge ignores everything from index `k` onward. There is no need to zero out, truncate, or clean up the tail of the array.

---

## Approach Analysis

### Approach 1 — Set then Overwrite — O(n) time, O(n) space

Collect all unique values into a set, sort them (the input is already sorted so this is trivial), write them back into `nums` from index `0`, return the count. Correct, but uses O(n) extra space for the set. Violates the spirit of in-place.

### Approach 2 — Two Pointers (Slow/Fast) — O(n) time, O(1) space ✅ Optimal

Maintain two pointers:

- `k` — the **write pointer** (slow), always pointing to the last confirmed unique position.
- `i` — the **read pointer** (fast), scanning forward through the array.

When `nums[i]` differs from `nums[k]`, a new unique value has been found — write it to position `k+1` and advance `k`. When `nums[i]` equals `nums[k]`, it is a duplicate — advance `i` and do nothing. One pass, constant space, no extra memory.

---

## Complexity

|Approach|Time|Space|
|---|---|---|
|Set then overwrite|O(n)|O(n)|
|Two pointers|O(n)|O(1)|

---

## Solution

```python
def removeDuplicates(nums: list[int]) -> int:
    k = 0

    for i in range(1, len(nums)):
        if nums[i] != nums[k]:
            k += 1
            nums[k] = nums[i]

    return k + 1
```

---

## Algorithm Walkthrough — `[0,0,1,1,1,2,2,3,3,4]`

```
Initial:  k=0  (nums[0]=0 is the first confirmed unique)

i=1  nums[1]=0  ==  nums[0]=0  →  duplicate,  skip
i=2  nums[2]=1  !=  nums[0]=0  →  new unique,  k=1,  nums[1]=1
i=3  nums[3]=1  ==  nums[1]=1  →  duplicate,  skip
i=4  nums[4]=1  ==  nums[1]=1  →  duplicate,  skip
i=5  nums[5]=2  !=  nums[1]=1  →  new unique,  k=2,  nums[2]=2
i=6  nums[6]=2  ==  nums[2]=2  →  duplicate,  skip
i=7  nums[7]=3  !=  nums[2]=2  →  new unique,  k=3,  nums[3]=3
i=8  nums[8]=3  ==  nums[3]=3  →  duplicate,  skip
i=9  nums[9]=4  !=  nums[3]=3  →  new unique,  k=4,  nums[4]=4

Array state: [0,1,2,3,4,2,2,3,3,4]
                         ↑ tail ignored

return k+1 = 5 ✅
```

The write pointer `k` only ever moves forward when a genuinely new value is found. The read pointer `i` moves forward unconditionally every iteration. The gap between them is exactly the number of duplicates encountered so far.

---

## Python Concepts Dissected

**`k = 0` as the write pointer**

`k` serves a dual purpose: it is both the index of the most recently confirmed unique element and a running count (offset by one) of unique elements found. Initializing it to `0` anchors it at the first element, which is always unique by definition since nothing precedes it.

---

**`for i in range(1, len(nums))`**

`range(start, stop)` generates integers from `start` up to but not including `stop`. Starting at `1` instead of `0` skips the first element intentionally — it is already placed at `k=0` and comparing it with itself would always produce a false "new unique." The read pointer begins one step ahead of the write pointer.

---

**`if nums[i] != nums[k]`**

Because the array is sorted, a value different from `nums[k]` is guaranteed to be strictly greater than it, and is guaranteed to be a new unique element not yet written. This single comparison does all the work. If the array were unsorted this would fail — but sorted order means all copies of a value are adjacent, so the write pointer always holds the last unique seen and the read pointer advancing through any run of duplicates will keep triggering `==` until the run ends.

---

**`k += 1` then `nums[k] = nums[i]`**

The order matters. `k` is incremented first to move to the next available write slot, then the new unique value is written there. Writing before incrementing would overwrite the current last confirmed unique. This two-step sequence — advance the write pointer, then write — is the standard pattern for all slow/fast pointer in-place problems.

---

**`nums[k] = nums[i]`**

Direct index assignment into a Python list. This modifies the list in place — no new list is created. In Python, lists are mutable objects and index assignment changes the element at that position in the same underlying memory. This is what satisfies the in-place constraint.

---

**`return k + 1`**

`k` is a zero-based index, not a count. If `k=4`, elements are at positions `0,1,2,3,4` — five unique elements. The count is always `k + 1`. Returning `k` would undercount by one. An alternative formulation increments `k` before writing (making `k` track count directly and writing to `k-1`), but the version above is more natural to read.

---

## Pattern to Internalize

This problem is the foundational example of the **slow/fast two-pointer pattern for in-place array modification**. The pattern structure is always the same: a read pointer (`i`) scans every element unconditionally, and a write pointer (`k`) advances only when a qualifying element is found. The write pointer defines a "clean prefix" of the array that satisfies some invariant — here, all unique elements in order. Elements in the tail beyond the write pointer are in an undefined, ignored state.

This same pattern, with only the condition changed, solves a whole class of related problems: remove all instances of a specific value, remove elements not satisfying a predicate, keep only elements appearing at most twice. In every case the array structure and the two-pointer mechanic are identical — only `if nums[i] != nums[k]` changes to reflect the new rule.