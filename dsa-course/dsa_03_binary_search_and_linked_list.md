# Tier 2 — Binary Search, Linked Lists

**Courses:** **DSA** · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · [C++](../cpp-course/cpp_00_README_start_here.md) · [Python](../python-course/python_00_README_start_here.md)
**DSA modules:** [Index](dsa_00_README_start_here.md) · [C++ Setup](dsa_01_cpp_setup_and_best_practices.md) · [Arrays/Hashing/2Ptr/Stack](dsa_02_arrays_hashing_two_pointers_stack.md) · **Binary Search/Linked List** · [Recursion/Sorting](dsa_04_recursion_and_sorting.md) · [Trees/Tries/Union-Find](dsa_05_trees_tries_union_find.md) · [Heap/Backtracking](dsa_06_heap_priority_queue_backtracking.md) · [Graphs](dsa_07_graphs.md) · [DP](dsa_08_dynamic_programming.md) · [Bit Manipulation](dsa_09_bit_manipulation.md) · [Cisco Problem Bank](dsa_10_cisco_problem_bank.md) · [Intervals](dsa_11_intervals.md)

---

## Binary Search — Search Array

**Trigger:** sorted array, looking for O(log n) instead of O(n).

```cpp
int lo = 0, hi = (int)nums.size() - 1;
while (lo <= hi) {
    int mid = lo + (hi - lo) / 2;   // avoids overflow vs (lo+hi)/2
    if (nums[mid] == target) return mid;
    else if (nums[mid] < target) lo = mid + 1;
    else hi = mid - 1;
}
return -1;
```
```python
lo, hi = 0, len(nums) - 1
while lo <= hi:
    mid = (lo + hi) // 2
    if nums[mid] == target:
        return mid
    elif nums[mid] < target:
        lo = mid + 1
    else:
        hi = mid - 1
return -1
```

**Gotcha:** `mid = (lo + hi) / 2` can overflow `int` if `lo + hi` exceeds `INT_MAX` on huge inputs. `lo + (hi - lo) / 2` is the safe idiom — worth typing out of habit even when the input size makes it moot; it reads as deliberate.

**STL shortcut:** `std::lower_bound(v.begin(), v.end(), target)` returns iterator to first element `>= target`; `std::upper_bound` for first `> target`. Subtract `v.begin()` to get an index. Use these when you just need the boundary, not a hand-rolled loop.

## Binary Search — Search Range ("Find First and Last Position")

**Trigger:** find the first/last occurrence of a target in a sorted array with duplicates.

- Run binary search twice: once biased to find the **leftmost** match (on `nums[mid] == target`, keep searching left: `hi = mid - 1`), once for the **rightmost** (`lo = mid + 1`).
- Or use `lower_bound`/`upper_bound` directly — `upper_bound - lower_bound` gives the count, and the two iterators give the range.

```cpp
auto first = std::lower_bound(nums.begin(), nums.end(), target);
auto last  = std::upper_bound(nums.begin(), nums.end(), target);
// range is [first, last); empty if first == last
```

---

## Linked Lists

### Singly Linked List

```cpp
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};
```
```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next
```

- **Always write the raw struct/class in interviews** — this is what's being tested, not `std::list`.
- Dummy head node pattern: for problems that might modify the head (remove Nth node, merge lists), create `ListNode dummy(0); dummy.next = head;` and operate relative to `dummy` — avoids special-casing "what if we remove the actual head."

### Doubly Linked List

```cpp
struct DListNode {
    int val;
    DListNode* prev;
    DListNode* next;
};
```
- Used for O(1) arbitrary-position deletion when you already hold the pointer (e.g., LRU cache implementation — a very common "implement a data structure" onsite question: doubly linked list + hash map of key → node).

### Queues (as linked-list-adjacent)

```cpp
std::queue<int> q;   // wraps deque by default
q.push(x); q.front(); q.pop();
```
```python
from collections import deque
q = deque()
q.append(x)      # push
q.popleft()       # pop from front — O(1), unlike list.pop(0) which is O(n)
```

**Gotcha (Python-specific):** never use a plain `list` as a queue with `.pop(0)` — that's O(n) per pop. Always `collections.deque`.

### Fast & Slow Pointers (Advanced list — grouped here since it's the same data structure)

**Trigger:** cycle detection, finding the middle of a list, "Nth node from the end."

```cpp
ListNode *slow = head, *fast = head;
while (fast && fast->next) {
    slow = slow->next;
    fast = fast->next->next;
}
// slow is now the middle node (or the meeting point if there's a cycle)
```
```python
slow = fast = head
while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
```

- Cycle detection (Floyd's): if `fast` ever equals `slow` inside the loop (after the first step), there's a cycle. To find the cycle's *start*, reset one pointer to `head` and advance both one step at a time — they meet at the start node. This is a genuinely non-obvious result; know it as a fact rather than trying to re-derive it live.
- **Gotcha:** check `fast && fast->next` in that order — `fast->next` on a null `fast` is a null-pointer dereference.

**Practice signal:** "detect a cycle," "middle of the list," "remove Nth from end," "is palindrome linked list" (find middle, reverse second half, compare) → fast/slow pointers.

---

## Cisco Practice Problems

Full methodology in `dsa_10_cisco_problem_bank.md`. 🔥 = asked in the last 3–6 months.

- MEDIUM — [Implement Router](https://leetcode.com/problems/implement-router) — binary search + design
- MEDIUM — [LRU Cache](https://leetcode.com/problems/lru-cache) — doubly linked list + hashmap
- EASY — [Linked List Cycle](https://leetcode.com/problems/linked-list-cycle) — fast/slow
- EASY — [Happy Number](https://leetcode.com/problems/happy-number) — fast/slow variant
- MEDIUM — [Add Two Numbers](https://leetcode.com/problems/add-two-numbers) — linked list
- EASY — [Reverse Linked List](https://leetcode.com/problems/reverse-linked-list)
- HARD — [Reverse Nodes in k-Group](https://leetcode.com/problems/reverse-nodes-in-k-group)
- MEDIUM — [Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array) — binary search
- MEDIUM — [Search in Rotated Sorted Array II](https://leetcode.com/problems/search-in-rotated-sorted-array-ii) — binary search
- MEDIUM — [Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number) — binary search / fast-slow
- EASY — [Convert Binary Number in a Linked List to Integer](https://leetcode.com/problems/convert-binary-number-in-a-linked-list-to-integer)
- MEDIUM — [Single Element in a Sorted Array](https://leetcode.com/problems/single-element-in-a-sorted-array) — binary search
- MEDIUM — [Time Based Key-Value Store](https://leetcode.com/problems/time-based-key-value-store) — binary search + design
