#easy 
# Merge Two Sorted Lists

---

## Problem Statement

Given the heads of two sorted linked lists `list1` and `list2`, merge them into one sorted linked list by splicing the original nodes together. Return the head of the merged list.

```
list1 = [1,2,4],  list2 = [1,3,4]  →  [1,1,2,3,4,4]
list1 = [],       list2 = []        →  []
list1 = [],       list2 = [0]       →  [0]
```

**Constraints:** `0 to 50` nodes per list, `-100 <= Node.val <= 100`, both lists sorted in non-decreasing order.

---

## The Data Structure — Linked List

Before any algorithm, the structure must be understood. A linked list is a chain of nodes where each node holds a value and a pointer to the next node. There is no indexing — the only way to reach a node is by following pointers from the head.

```
list1:  1  →  2  →  4  →  None
list2:  1  →  3  →  4  →  None
```

In Python this is represented as a class:

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next
```

Each node owns two attributes: `val` (the stored integer) and `next` (a reference to the next node, or `None` at the tail). The problem guarantees this class exists — it does not need to be redefined in the solution.

---

## Immediate Observations

**If either list is empty, return the other.** An empty list has nothing to merge. The non-empty list is already sorted and is the answer on its own. This resolves three of the given examples immediately.

**The merged list uses the same nodes.** The problem says "splicing together the nodes" — no new nodes are created. Pointers are rewired, not values copied. The output is a reorganized view of the same memory.

**The smallest unprocessed value always comes from one of the two current heads.** Because both lists are sorted, the minimum remaining element is always at the front of one of the two lists. Comparing just the two heads at each step is sufficient — no lookahead required.

---

## Approach Analysis

### Approach 1 — Collect and Sort — O((n+m) log(n+m)) time, O(n+m) space

Collect all values into an array, sort, rebuild a new linked list. Correct but ignores the sorted property entirely. The sort costs O((n+m) log(n+m)) when an O(n+m) solution exists. Wasteful and not what the problem intends.

### Approach 2 — Iterative with Dummy Head — O(n+m) time, O(1) space ✅ Optimal

Use a dummy head node as a fixed anchor for the result list. Maintain a `current` pointer that always points to the last node appended. At each step, compare `list1.val` and `list2.val`, attach the smaller node to `current.next`, advance that list's pointer, and advance `current`. When one list is exhausted, attach the remainder of the other directly — no further iteration needed since it is already sorted. Return `dummy.next` as the head of the merged list.

### Approach 3 — Recursive — O(n+m) time, O(n+m) space

Define the merge recursively: the merged list is whichever head is smaller, with its `next` set to the recursive merge of the remainder. Elegant and concise, but each recursive call consumes a stack frame — O(n+m) call stack space. For `n+m = 100` nodes this is inconsequential in practice, but the iterative approach is strictly superior in space.

---

## Complexity

|Approach|Time|Space|Notes|
|---|---|---|---|
|Collect and sort|O((n+m) log(n+m))|O(n+m)|Ignores sorted property|
|Iterative dummy head|O(n+m)|O(1)|Optimal|
|Recursive|O(n+m)|O(n+m)|Stack frames accumulate|

---

## Solution

### Approach 2 — Iterative with Dummy Head (Optimal)

```python
def mergeTwoLists(list1: ListNode, list2: ListNode) -> ListNode:
    dummy = ListNode(0)
    current = dummy

    while list1 and list2:
        if list1.val <= list2.val:
            current.next = list1
            list1 = list1.next
        else:
            current.next = list2
            list2 = list2.next
        current = current.next

    current.next = list1 if list1 else list2

    cc
```

### Approach 3 — Recursive

```python
def mergeTwoLists(list1: ListNode, list2: ListNode) -> ListNode:
    if not list1:
        return list2
    if not list2:
        return list1

    if list1.val <= list2.val:
        list1.next = mergeTwoLists(list1.next, list2)
        return list1
    else:
        list2.next = mergeTwoLists(list1, list2.next)
        return list2
```

---

## Algorithm Walkthrough — `list1 = [1,2,4]`, `list2 = [1,3,4]`

```
dummy → [0]
current = dummy

list1=[1,2,4]  list2=[1,3,4]
  1 <= 1 → attach list1's 1,  list1 advances to [2,4]
  current = node(1)

list1=[2,4]    list2=[1,3,4]
  2 >  1 → attach list2's 1,  list2 advances to [3,4]
  current = node(1)

list1=[2,4]    list2=[3,4]
  2 <= 3 → attach list1's 2,  list1 advances to [4]
  current = node(2)

list1=[4]      list2=[3,4]
  4 >  3 → attach list2's 3,  list2 advances to [4]
  current = node(3)

list1=[4]      list2=[4]
  4 <= 4 → attach list1's 4,  list1 advances to None
  current = node(4)

list1=None → loop exits

current.next = list2 = [4]   (remainder attached directly)

return dummy.next

Result: [1,1,2,3,4,4] ✅
```

---

## Python Concepts Dissected

**`dummy = ListNode(0)`**

A sentinel node — a placeholder with an arbitrary value (`0`) that will never appear in the final result. Its sole purpose is to give `current` something to start from. Without it, the first node attached would require special-case logic: "if the result list is empty, set its head; otherwise append." The dummy node eliminates that branch entirely — the first append is identical to every subsequent one. `dummy.next` is the true head of the merged list, returned at the end.

---

**`current = dummy`**

`current` is a pointer that tracks the tail of the merged list being built. It starts at the dummy node and advances one step forward with every node appended. Maintaining a tail pointer is what makes each append O(1) — without it, every append would require traversing the entire built list to find the end.

---

**`while list1 and list2`**

A linked list node object is truthy in Python — it exists and is not `None`. `None` is falsy. So `while list1 and list2` continues as long as both pointers are pointing at a valid node, and exits the moment either list is exhausted. This is the idiomatic way to check whether a linked list pointer is non-null.

---

**`current.next = list1`**

This line does not copy a value — it rewires a pointer. `current.next` previously pointed to `None` (or wherever the dummy pointed). Now it points directly to the node `list1` is referencing. The node is not duplicated; it is spliced into the merged list by changing where `current.next` points.

---

**`list1 = list1.next`**

Advances the `list1` pointer to the next node in its original chain. The node just appended to the merged list is no longer accessible via `list1` — the pointer has moved on. The node itself still exists in memory, now part of the merged list via `current.next`.

---

**`current = current.next`**

Advances the tail pointer. After appending a node, `current` must move to that newly appended node so the next append connects to the correct place. Without this step, every node would overwrite `dummy.next` rather than chaining forward.

---

**`current.next = list1 if list1 else list2`**

After the loop, one list is exhausted and the other may still have nodes. Because the remaining list is already sorted, it can be attached wholesale — no further comparison needed. The conditional expression selects whichever list is non-empty. If both are `None` (both lists exhausted simultaneously), `None` is assigned, which is exactly what a tail node's `next` should be.

---

**`return dummy.next`**

The dummy node was never part of the real merged list — it was only a convenient starting anchor. `dummy.next` is the first real node appended, which is the head of the merged result. Returning `dummy` itself would incorrectly include the sentinel.

---

**Recursive approach — `list1.next = mergeTwoLists(list1.next, list2)`**

The recursive call returns the head of the merged list formed from `list1.next` and `list2`. Assigning that result to `list1.next` splices it in as the continuation of the current node. Then `list1` (the current, smaller node) is returned as the head of this level's result. The call stack unwinds from the deepest pair upward, stitching the list together from the tail back to the head.

---

## Pattern to Internalize

This problem introduces three ideas that persist throughout linked list problems.

The first is the **dummy head pattern**. Any time a linked list is being constructed incrementally — merged, filtered, partitioned, reversed in segments — a dummy head eliminates the special case of initializing the first node. The real result is always `dummy.next`.

The second is **pointer manipulation as the core operation**. Linked list problems are not about values — they are about rewiring references. Reading `current.next = node` as "attach this node here" and `ptr = ptr.next` as "advance one step" is the mental model that makes every linked list algorithm readable.

The third is the **two-pointer exhaustion pattern**: run two pointers in parallel, consuming from whichever is locally smaller, and attach the remainder of the surviving pointer when the other is exhausted. This same structure appears in merge sort's merge step, in finding intersections of sorted arrays, and in k-way merge problems.