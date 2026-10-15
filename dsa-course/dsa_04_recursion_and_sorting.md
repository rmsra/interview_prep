# Recursion & Sorting

**Courses:** **DSA** · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · [C++](../cpp-course/cpp_00_README_start_here.md) · [Python](../python-course/python_00_README_start_here.md)
**DSA modules:** [Index](dsa_00_README_start_here.md) · [C++ Setup](dsa_01_cpp_setup_and_best_practices.md) · [Arrays/Hashing/2Ptr/Stack](dsa_02_arrays_hashing_two_pointers_stack.md) · [Binary Search/Linked List](dsa_03_binary_search_and_linked_list.md) · **Recursion/Sorting** · [Trees/Tries/Union-Find](dsa_05_trees_tries_union_find.md) · [Heap/Backtracking](dsa_06_heap_priority_queue_backtracking.md) · [Graphs](dsa_07_graphs.md) · [DP](dsa_08_dynamic_programming.md) · [Bit Manipulation](dsa_09_bit_manipulation.md) · [Cisco Problem Bank](dsa_10_cisco_problem_bank.md) · [Intervals](dsa_11_intervals.md)

---

Recursion basics (factorial/fibonacci) are almost certainly a formality for you — included for completeness since it's foundational for everything downstream (trees, backtracking, DP). Sorting is worth being crisp on since interviewers sometimes ask you to implement one from scratch or reason about stability/complexity.

## Recursion — Factorial (base case + recurrence, skim)

```cpp
long long factorial(int n) {
    if (n <= 1) return 1;          // base case
    return n * factorial(n - 1);   // recurrence
}
```
```python
def factorial(n):
    if n <= 1:
        return 1
    return n * factorial(n - 1)
```
- Every recursive function needs: (1) a base case that terminates, (2) a recurrence that provably moves toward the base case.
- C++ default stack is ~1-8MB depending on platform; deep recursion (n > ~10⁴–10⁵ depending on frame size) risks stack overflow — mention this if asked about recursion vs. iteration trade-offs, it's a senior-signal point.

## Recursion — Fibonacci (the point is memoization, not the recurrence itself)

```cpp
// naive: O(2^n) — exponential, don't submit this as final
int fib(int n) {
    if (n <= 1) return n;
    return fib(n-1) + fib(n-2);
}

// memoized: O(n) time, O(n) space
int fibMemo(int n, std::vector<int>& memo) {
    if (n <= 1) return n;
    if (memo[n] != -1) return memo[n];
    return memo[n] = fibMemo(n-1, memo) + fibMemo(n-2, memo);
}
```
```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fib(n):
    if n <= 1:
        return n
    return fib(n-1) + fib(n-2)
```
- **This is the real teaching point:** naive recursion recomputes overlapping subproblems exponentially. Memoization (top-down DP) or an iterative bottom-up loop fixes it. This exact insight is the bridge into the DP file — call it out explicitly if an interviewer asks you to optimize a naive recursive solution.

---

## Sorting

| Algorithm | Time (avg) | Time (worst) | Space | Stable? |
|---|---|---|---|---|
| Insertion Sort | O(n²) | O(n²) | O(1) | Yes |
| Merge Sort | O(n log n) | O(n log n) | O(n) | Yes |
| Quick Sort | O(n log n) | O(n²) | O(log n) | No |
| Bucket Sort | O(n + k) | O(n²) | O(n + k) | Yes (usually) |

### Insertion Sort
- Good to know cold: it's O(n) on nearly-sorted data and is what many `std::sort` implementations fall back to for small subarrays.

```cpp
for (int i = 1; i < (int)v.size(); i++) {
    int key = v[i], j = i - 1;
    while (j >= 0 && v[j] > key) { v[j+1] = v[j]; j--; }
    v[j+1] = key;
}
```

### Merge Sort
- Divide and conquer: split in half, sort each half, merge two sorted halves in O(n). Guarantees O(n log n) even worst case — this is why it's preferred when you need a *guarantee*, e.g., "sort a linked list in O(n log n)."

```cpp
void merge(std::vector<int>& v, int l, int m, int r) {
    std::vector<int> tmp;
    int i = l, j = m + 1;
    while (i <= m && j <= r) tmp.push_back(v[i] <= v[j] ? v[i++] : v[j++]);
    while (i <= m) tmp.push_back(v[i++]);
    while (j <= r) tmp.push_back(v[j++]);
    for (int k = l; k <= r; k++) v[k] = tmp[k - l];
}
void mergeSort(std::vector<int>& v, int l, int r) {
    if (l >= r) return;
    int m = l + (r - l) / 2;
    mergeSort(v, l, m);
    mergeSort(v, m + 1, r);
    merge(v, l, m, r);
}
```

### Quick Sort
- Pick a pivot, partition into `< pivot` / `>= pivot`, recurse on each side. Average O(n log n), worst-case O(n²) on already-sorted input with a naive pivot — mention **randomized pivot selection** (or median-of-three) to avoid this; it's a standard follow-up question.

```cpp
int partition(std::vector<int>& v, int l, int r) {
    int pivot = v[r], i = l;
    for (int j = l; j < r; j++)
        if (v[j] < pivot) std::swap(v[i++], v[j]);
    std::swap(v[i], v[r]);
    return i;
}
void quickSort(std::vector<int>& v, int l, int r) {
    if (l >= r) return;
    int p = partition(v, l, r);
    quickSort(v, l, p - 1);
    quickSort(v, p + 1, r);
}
```

### Bucket Sort
- Only useful with known/bounded input distribution (e.g., floats in [0,1), or integers in a known small range). Distribute into k buckets, sort each bucket (often with insertion sort since buckets are small), concatenate. Related: **Counting Sort** for small-range integers — worth knowing the name even though it's not in your list, since interviewers sometimes conflate the two.

```cpp
// sketch for floats in [0, 1)
std::vector<std::vector<float>> buckets(n);
for (float x : v) buckets[(int)(x * n)].push_back(x);
for (auto& b : buckets) std::sort(b.begin(), b.end());
// concatenate buckets in order
```

**In practice:** for live coding, `std::sort(v.begin(), v.end())` (C++) / `v.sort()` or `sorted(v)` (Python) is what you actually call — these hand-rolled versions are for when an interviewer explicitly asks "implement X sort" or asks you to reason about why one is chosen over another (e.g., "why merge sort for linked lists, not quicksort?" → no random access, so quicksort's partition step loses its advantage, and merge sort's sequential access is a natural fit).

---

## Cisco Practice Problems

Full methodology in `dsa_10_cisco_problem_bank.md`.

- MEDIUM — [Pow(x, n)](https://leetcode.com/problems/powx-n) — recursion, fast exponentiation
- HARD — [Integer to English Words](https://leetcode.com/problems/integer-to-english-words) — recursion + string simulation, low pattern value
