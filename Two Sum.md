#easy

# Two Sum
## Problem Statement

Given an array of integers `nums` and an integer `target`, return the indices of the two numbers that add up to `target`. Exactly one solution is guaranteed. You may not use the same element twice.

```
Input:  nums = [2, 7, 11, 15],  target = 9
Output: [0, 1]
```

---

## Approach Analysis

### Brute Force — O(n²) time, O(1) space

For every element, iterate over every other element and test if they sum to `target`. Two nested loops — every pair is checked. Simple but wasteful: at n = 10,000, that's up to 100 million comparisons.

### Sort + Two Pointers — O(n log n) time, O(n) space

Sort the array, place one pointer at the smallest value and one at the largest. Shrink the window based on whether the current sum overshoots or undershoots the target. Elegant, but sorting destroys original indices — extra bookkeeping is required and the time complexity still doesn't beat the optimal.

### Hash Map — O(n) time, O(n) space

The key insight: instead of asking "does any other element pair with this one?", ask "have I already seen the complement of this element?"

For each element `x`, compute `complement = target - x`. If that complement already exists in the hash map, the pair is found. Otherwise, store `x → index` and continue. One pass. Each lookup and insert is O(1) average.

---

## Complexity

|Approach|Time|Space|
|---|---|---|
|Brute force|O(n²)|O(1)|
|Sort + two pointers|O(n log n)|O(n)|
|Hash map|O(n)|O(n)|

The hash map exchanges linear space for linear time — each lookup replaces a full inner loop.

---

## Solution

```python
def twoSum(nums: list[int], target: int) -> list[int]:
    seen = {}

    for i, num in enumerate(nums):
        complement = target - num

        if complement in seen:
            return [seen[complement], i]

        seen[num] = i

    return []
```

---

## Python Concepts Dissected

**Type hints**

```python
def twoSum(nums: list[int], target: int) -> list[int]:
```

`nums: list[int]` annotates the parameter as a list of integers. `-> list[int]` annotates the return type. Python does not enforce these at runtime — they exist for readability and static analysis tools like `mypy`.

---

**Dictionary literal**

```python
seen = {}
```

`{}` creates an empty `dict`. Python dictionaries are hash tables internally, giving O(1) average time for both key insertion and key lookup. This is the data structure doing all the heavy lifting.

---

**`enumerate()`**

```python
for i, num in enumerate(nums):
```

`enumerate()` wraps any iterable and yields `(index, value)` pairs. The `i, num` part is tuple unpacking — Python assigns both values in one statement. The alternative `for i in range(len(nums)): num = nums[i]` is functionally identical but considered non-idiomatic.

---

**Arithmetic complement**

```python
complement = target - num
```

The value we need to complete the pair. If `num = 2` and `target = 9`, then `complement = 7`. This single line encodes the core problem reduction.

---

**`in` on a dictionary**

```python
if complement in seen:
```

The `in` operator on a `dict` checks keys only, in O(1) average time. If written as `if complement in list`, the check would be O(n) — lists scan linearly. Always prefer `in dict` for existence checks.

---

**Index retrieval and return**

```python
return [seen[complement], i]
```

`seen[complement]` retrieves the previously stored index of the complement. That index is returned first because it is always earlier in the array. The result is a two-element list literal.

---

**Store after lookup**

```python
seen[num] = i
```

This line runs after the lookup, not before. The ordering is intentional: checking before storing ensures an element cannot be matched against itself.

---

**Fallback return**

```python
return []
```

Unreachable given the problem guarantee of exactly one solution. Present for completeness — every code path in a function should return a value of the declared type.

---

## Pattern to Internalize

This problem is a canonical instance of the **complement lookup pattern**: whenever a brute-force approach requires finding a matching element in the remaining array, a hash map converts that O(n) inner scan into an O(1) lookup. You will apply this exact pattern to dozens of LeetCode problems going forward.