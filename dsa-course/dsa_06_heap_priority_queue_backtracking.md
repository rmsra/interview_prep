# Heap / Priority Queue, Backtracking

**Courses:** **DSA** · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · [C++](../cpp-course/cpp_00_README_start_here.md) · [Python](../python-course/python_00_README_start_here.md)
**DSA modules:** [Index](dsa_00_README_start_here.md) · [C++ Setup](dsa_01_cpp_setup_and_best_practices.md) · [Arrays/Hashing/2Ptr/Stack](dsa_02_arrays_hashing_two_pointers_stack.md) · [Binary Search/Linked List](dsa_03_binary_search_and_linked_list.md) · [Recursion/Sorting](dsa_04_recursion_and_sorting.md) · [Trees/Tries/Union-Find](dsa_05_trees_tries_union_find.md) · **Heap/Backtracking** · [Graphs](dsa_07_graphs.md) · [DP](dsa_08_dynamic_programming.md) · [Bit Manipulation](dsa_09_bit_manipulation.md) · [Cisco Problem Bank](dsa_10_cisco_problem_bank.md) · [Intervals](dsa_11_intervals.md)

---

## Heap Properties

- Complete binary tree stored in an array: for index `i`, children are `2i+1`, `2i+2`; parent is `(i-1)/2`.
- **Max-heap:** parent ≥ children (root = max). **Min-heap:** parent ≤ children (root = min).
- `std::priority_queue<T>` defaults to **max-heap**. For min-heap: `std::priority_queue<T, std::vector<T>, std::greater<T>>`.
- Python's `heapq` module only implements a **min-heap**. For max-heap behavior, negate values on push/pop.

```cpp
std::priority_queue<int> maxHeap;
std::priority_queue<int, std::vector<int>, std::greater<int>> minHeap;
maxHeap.push(5); maxHeap.top(); maxHeap.pop();
```
```python
import heapq
min_heap = []
heapq.heappush(min_heap, 5)
smallest = min_heap[0]
heapq.heappop(min_heap)
# max-heap trick: push negated values
heapq.heappush(min_heap, -5)
largest = -min_heap[0]
```

## Push and Pop

- Push: add to end of array, "bubble up" — swap with parent while it violates the heap property. O(log n).
- Pop (removes root): move last element to root, "bubble down" — swap with the smaller/larger child while it violates the property. O(log n).

## Heapify

- Building a heap from an unsorted array by repeated single-element push is O(n log n). The **heapify** algorithm — start from the last non-leaf node and bubble down each node in reverse level order — builds a heap in **O(n)**. This is a genuinely surprising result (not O(n log n)) and a good one to state correctly if asked why `std::make_heap`/`heapq.heapify` are preferred over building via repeated insertion.

```cpp
std::vector<int> v = {5,3,8,1,9,2};
std::make_heap(v.begin(), v.end());   // O(n)
```
```python
import heapq
v = [5,3,8,1,9,2]
heapq.heapify(v)   # O(n), in place
```

**Practice signal:** "kth largest/smallest," "top k frequent," "merge k sorted lists," "median of a stream" → heap.

## Two Heaps

**Trigger:** "median of a running stream" — the canonical use case.

- Maintain a max-heap for the lower half and a min-heap for the upper half, keeping them balanced in size (differ by at most 1).
- Insert into the appropriate half, then rebalance by moving the top element across if sizes differ by more than 1.
- Median: top of the larger heap, or average of both tops if equal size.

```cpp
std::priority_queue<int> lo;                                             // max-heap, lower half
std::priority_queue<int, std::vector<int>, std::greater<int>> hi;        // min-heap, upper half

void addNum(int num) {
    if (lo.empty() || num <= lo.top()) lo.push(num); else hi.push(num);
    if (lo.size() > hi.size() + 1) { hi.push(lo.top()); lo.pop(); }
    else if (hi.size() > lo.size() + 1) { lo.push(hi.top()); hi.pop(); }
}
double findMedian() {
    if (lo.size() == hi.size()) return (lo.top() + hi.top()) / 2.0;
    return lo.size() > hi.size() ? lo.top() : hi.top();
}
```
- Each `addNum` is O(log n); `findMedian` is O(1).

---

## Backtracking

**Core shape:** choose → recurse → un-choose ("undo the choice on the way back up"). Every problem below is this same loop with a different "choose" step and a different pruning/validity check.

```cpp
void backtrack(/* state */, std::vector<T>& path, std::vector<std::vector<T>>& result) {
    if (/* base case: path is complete */) {
        result.push_back(path);
        return;
    }
    for (/* each candidate choice */) {
        if (/* choice is invalid, prune */) continue;
        path.push_back(choice);          // choose
        backtrack(/* advanced state */, path, result);  // recurse
        path.pop_back();                 // un-choose — this line is the one people forget
    }
}
```

### Tree Maze (grid backtracking)

**Trigger:** find a path through a grid with obstacles (word search, rat in a maze, number of paths).

```cpp
bool dfs(std::vector<std::vector<int>>& grid, int r, int c, std::vector<std::vector<bool>>& visited) {
    if (r < 0 || r >= (int)grid.size() || c < 0 || c >= (int)grid[0].size()) return false;
    if (grid[r][c] == 1 /* wall */ || visited[r][c]) return false;
    if (/* reached goal */ false) return true;
    visited[r][c] = true;
    static int dr[] = {-1,1,0,0}, dc[] = {0,0,-1,1};
    for (int d = 0; d < 4; d++)
        if (dfs(grid, r + dr[d], c + dc[d], visited)) return true;
    visited[r][c] = false;   // un-choose — allows this cell to be part of a different path
    return false;
}
```
- **Gotcha:** forgetting to reset `visited[r][c] = false` on the way back up means a cell that's a dead end on one path is wrongly blocked when a different path could have used it.

### Subsets

**Trigger:** "all subsets/power set" — 2ⁿ results.

```cpp
void subsets(std::vector<int>& nums, int start, std::vector<int>& path, std::vector<std::vector<int>>& result) {
    result.push_back(path);
    for (int i = start; i < (int)nums.size(); i++) {
        path.push_back(nums[i]);
        subsets(nums, i + 1, path, result);
        path.pop_back();
    }
}
```
- Note `result.push_back(path)` happens on **every** call, not just at a "full" base case — every prefix is a valid subset.

### Combinations

**Trigger:** choose k elements from n, order doesn't matter.

```cpp
void combine(int n, int k, int start, std::vector<int>& path, std::vector<std::vector<int>>& result) {
    if ((int)path.size() == k) { result.push_back(path); return; }
    for (int i = start; i <= n; i++) {
        path.push_back(i);
        combine(n, k, i + 1, path, result);   // i+1, not start+1 — never reuse earlier elements
        path.pop_back();
    }
}
```

### Permutations

**Trigger:** all orderings — n! results, order matters.

```cpp
void permute(std::vector<int>& nums, std::vector<bool>& used, std::vector<int>& path, std::vector<std::vector<int>>& result) {
    if (path.size() == nums.size()) { result.push_back(path); return; }
    for (int i = 0; i < (int)nums.size(); i++) {
        if (used[i]) continue;
        used[i] = true;
        path.push_back(nums[i]);
        permute(nums, used, path, result);
        path.pop_back();
        used[i] = false;
    }
}
```
- Key difference from combinations: the inner loop restarts from `0` every level (any unused element can go next), tracked with a `used[]` array — vs. combinations, which always advances `start` forward to avoid picking the same or an earlier element twice.

**Complexity note for the whole file:** backtracking is inherently exponential (subsets O(2ⁿ), permutations O(n!)) — that's expected and not a red flag. What interviewers actually grade is (1) correct pruning to avoid wasted branches, and (2) whether you remember the un-choose step every time.

**Practice signal:** "all possible," "every combination/permutation/subset," "find a path," a grid with "return true if a path exists" → backtracking.

---

## Cisco Practice Problems

Full methodology in `dsa_10_cisco_problem_bank.md`.

- HARD — [Word Search II](https://leetcode.com/problems/word-search-ii) — backtracking + Trie
- MEDIUM — [Word Search](https://leetcode.com/problems/word-search) — backtracking, grid
- MEDIUM — [Letter Combinations of a Phone Number](https://leetcode.com/problems/letter-combinations-of-a-phone-number) — backtracking
- MEDIUM — [Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii) — heap (full pattern in file 11)
- MEDIUM — [Permutations](https://leetcode.com/problems/permutations) — backtracking
- HARD — [Find Servers That Handled Most Number of Requests](https://leetcode.com/problems/find-servers-that-handled-most-number-of-requests) — heap + simulation
- HARD — [Design Search Autocomplete System](https://leetcode.com/problems/design-search-autocomplete-system) — heap + Trie

---

## Related Modules

- [DSA 11 — Intervals (Meeting Rooms II reuses the heap pattern)](dsa_11_intervals.md)
