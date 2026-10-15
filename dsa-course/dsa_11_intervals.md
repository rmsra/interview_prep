# Intervals

**Courses:** **DSA** · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · [C++](../cpp-course/cpp_00_README_start_here.md) · [Python](../python-course/python_00_README_start_here.md)
**DSA modules:** [Index](dsa_00_README_start_here.md) · [C++ Setup](dsa_01_cpp_setup_and_best_practices.md) · [Arrays/Hashing/2Ptr/Stack](dsa_02_arrays_hashing_two_pointers_stack.md) · [Binary Search/Linked List](dsa_03_binary_search_and_linked_list.md) · [Recursion/Sorting](dsa_04_recursion_and_sorting.md) · [Trees/Tries/Union-Find](dsa_05_trees_tries_union_find.md) · [Heap/Backtracking](dsa_06_heap_priority_queue_backtracking.md) · [Graphs](dsa_07_graphs.md) · [DP](dsa_08_dynamic_programming.md) · [Bit Manipulation](dsa_09_bit_manipulation.md) · [Cisco Problem Bank](dsa_10_cisco_problem_bank.md) · **Intervals**

---

Added after the Cisco tag data showed this pattern recurring (Merge Intervals, Meeting Rooms II) — it wasn't in your original two topic lists or explicitly prioritized on the roadmap image, but it's common enough across companies generally that it's worth having cold.

**Core idea for almost every interval problem:** sort by start time first. Once sorted, most of these become a single linear pass — the hard part is almost never the algorithm, it's correctly defining what "overlap" means for the specific problem (`<` vs `<=` at the boundary trips people up more than anything else in this file).

## Merge Intervals

**Trigger:** "merge all overlapping intervals," given a list of `[start, end]` pairs.

```cpp
std::vector<std::vector<int>> merge(std::vector<std::vector<int>>& intervals) {
    if (intervals.empty()) return {};
    std::sort(intervals.begin(), intervals.end());   // sorts by start (first element) by default
    std::vector<std::vector<int>> result;
    result.push_back(intervals[0]);
    for (int i = 1; i < (int)intervals.size(); i++) {
        if (intervals[i][0] <= result.back()[1]) {
            result.back()[1] = std::max(result.back()[1], intervals[i][1]);   // extend the last merged interval
        } else {
            result.push_back(intervals[i]);   // no overlap — start a new interval
        }
    }
    return result;
}
```
```python
def merge(intervals):
    if not intervals:
        return []
    intervals.sort()
    result = [intervals[0]]
    for start, end in intervals[1:]:
        if start <= result[-1][1]:
            result[-1][1] = max(result[-1][1], end)
        else:
            result.append([start, end])
    return result
```
- O(n log n), dominated by the sort. The merge pass itself is O(n).
- **Gotcha:** `intervals[i][0] <= result.back()[1]` — whether it's `<=` or `<` depends on whether touching endpoints count as overlapping (e.g., `[1,3]` and `[3,5]` — does merging happen?). Read the problem statement for this exact edge case before coding; it's the single most common off-by-one in this file.

## Insert Interval

**Trigger:** given an already-sorted, non-overlapping list of intervals, insert a new one and merge as needed.

- Three linear passes (can be done in one, but three is easier to get right live): (1) add all intervals ending before the new one starts, unchanged; (2) merge all intervals that overlap the new one into a single expanded interval; (3) add all remaining intervals, unchanged.
```cpp
std::vector<std::vector<int>> insert(std::vector<std::vector<int>>& intervals, std::vector<int> newInterval) {
    std::vector<std::vector<int>> result;
    int i = 0, n = intervals.size();
    while (i < n && intervals[i][1] < newInterval[0]) result.push_back(intervals[i++]);       // before, no overlap
    while (i < n && intervals[i][0] <= newInterval[1]) {                                       // overlaps — merge in
        newInterval[0] = std::min(newInterval[0], intervals[i][0]);
        newInterval[1] = std::max(newInterval[1], intervals[i][1]);
        i++;
    }
    result.push_back(newInterval);
    while (i < n) result.push_back(intervals[i++]);                                            // after, no overlap
    return result;
}
```
- Because the input is already sorted and non-overlapping, this is O(n) — no sort needed. If you find yourself reaching for a sort here, you're missing the given invariant.

## Non-overlapping Intervals ("minimum removals to make non-overlapping")

**Trigger:** "minimum number of intervals to remove so the rest don't overlap" — this is an interval scheduling / greedy problem, not the same shape as merge.

- Sort by **end time**, not start time. Greedily keep an interval if it starts at or after the end of the last kept interval; otherwise it must be removed (it's the one causing the conflict).
```cpp
int eraseOverlapIntervals(std::vector<std::vector<int>>& intervals) {
    std::sort(intervals.begin(), intervals.end(),
              [](auto& a, auto& b) { return a[1] < b[1]; });   // sort by END time
    int count = 0, lastEnd = INT_MIN;
    for (auto& iv : intervals) {
        if (iv[0] >= lastEnd) lastEnd = iv[1];        // keep it
        else count++;                                  // remove it (overlaps the last kept one)
    }
    return count;
}
```
- **Why end time, not start time:** sorting by end time and greedily keeping the earliest-ending option leaves the most room for future intervals — this is the classic "activity selection" greedy proof. Worth stating that reasoning out loud if asked why the sort key differs from Merge Intervals above.

## Meeting Rooms II ("minimum number of rooms needed")

**Trigger:** given meeting intervals, find the max number that overlap at any single point in time — i.e., minimum rooms required.

**Approach 1 — two sorted arrays (starts and ends):**
```cpp
int minMeetingRooms(std::vector<std::vector<int>>& intervals) {
    std::vector<int> starts, ends;
    for (auto& iv : intervals) { starts.push_back(iv[0]); ends.push_back(iv[1]); }
    std::sort(starts.begin(), starts.end());
    std::sort(ends.begin(), ends.end());
    int rooms = 0, maxRooms = 0, s = 0, e = 0;
    while (s < (int)starts.size()) {
        if (starts[s] < ends[e]) { rooms++; s++; }        // a meeting starts before the earliest one ends
        else { rooms--; e++; }                             // a meeting ends — free up a room
        maxRooms = std::max(maxRooms, rooms);
    }
    return maxRooms;
}
```
**Approach 2 — min-heap of end times** (more intuitive to explain live, same O(n log n)):
```cpp
int minMeetingRooms(std::vector<std::vector<int>>& intervals) {
    std::sort(intervals.begin(), intervals.end());  // by start time
    std::priority_queue<int, std::vector<int>, std::greater<int>> endTimes;  // min-heap
    for (auto& iv : intervals) {
        if (!endTimes.empty() && endTimes.top() <= iv[0]) endTimes.pop();   // room freed up — reuse it
        endTimes.push(iv[1]);
    }
    return endTimes.size();   // number of rooms simultaneously in use at peak
}
```
- The heap version is the one worth leading with live — it reuses the heap pattern from file 06, and "sort by start, use a min-heap of end times to track the earliest-freeing room" is a clean sentence to say out loud before coding.

**Practice signal for this file:** "overlapping," "merge," "meeting rooms," "minimum removals to avoid conflicts," "insert into a sorted schedule" → intervals. Default move: sort (by start, unless the problem is really about greedy removal, then by end), then a single linear pass.

---

## Cisco Practice Problems

Full methodology in `dsa_10_cisco_problem_bank.md`.

- MEDIUM — [Merge Intervals](https://leetcode.com/problems/merge-intervals) — the core pattern above
- MEDIUM — [Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii) — min-heap of end times

---

## Related Modules

- [DSA 02 — Arrays/Sorting (the merge pattern's home)](dsa_02_arrays_hashing_two_pointers_stack.md)
- [DSA 06 — Heap (Meeting Rooms II)](dsa_06_heap_priority_queue_backtracking.md)
