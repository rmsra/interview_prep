# Tier 1 — Arrays & Hashing, Two Pointers, Stack

**Courses:** **DSA** · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · [C++](../cpp-course/cpp_00_README_start_here.md) · [Python](../python-course/python_00_README_start_here.md)
**DSA modules:** [Index](dsa_00_README_start_here.md) · [C++ Setup](dsa_01_cpp_setup_and_best_practices.md) · **Arrays/Hashing/2Ptr/Stack** · [Binary Search/Linked List](dsa_03_binary_search_and_linked_list.md) · [Recursion/Sorting](dsa_04_recursion_and_sorting.md) · [Trees/Tries/Union-Find](dsa_05_trees_tries_union_find.md) · [Heap/Backtracking](dsa_06_heap_priority_queue_backtracking.md) · [Graphs](dsa_07_graphs.md) · [DP](dsa_08_dynamic_programming.md) · [Bit Manipulation](dsa_09_bit_manipulation.md) · [Cisco Problem Bank](dsa_10_cisco_problem_bank.md) · [Intervals](dsa_11_intervals.md)

---

This tier is the foundation of almost every onsite problem. Given your background, the "what is RAM/an array" parts are a formality — skim those, spend your time on the pattern sections (Kadane's, sliding window, two pointers, prefix sums).

---

## RAM & Static Arrays (skim)

- Contiguous block of memory; index access is O(1) because address = `base + index * sizeof(T)`.
- Static array: fixed size at compile time, stack-allocated (`int arr[10];`), no resize.
- **Gotcha:** in C++, a raw array decays to a pointer when passed to a function — you lose `sizeof` info. Pass size explicitly or use `std::array`/`std::vector`.

## Dynamic Arrays

- `std::vector<T>` — amortized O(1) `push_back` via geometric growth (usually doubling); O(n) worst case on the resize call itself.
- Python: `list` — same amortized-doubling behavior under the hood.

```cpp
std::vector<int> v;
v.reserve(100);      // avoid repeated reallocation if size is known
v.push_back(5);
v.emplace_back(6);   // constructs in place — prefer for non-trivial types
```
```python
v = []
v.append(5)
```

**Gotcha:** `vector::erase(v.begin() + i)` is O(n) — shifts every element after `i`. If you're erasing a lot, consider swapping with the last element and popping (O(1)) when order doesn't matter.

## Stacks

- LIFO. `std::stack<T>` wraps a `deque` by default; many interviewers prefer you just use `std::vector<T>` directly (`.back()`, `.pop_back()`) since it's faster to write and equally correct.

```cpp
std::vector<int> st;
st.push_back(x);
int top = st.back();
st.pop_back();
bool empty = st.empty();
```
```python
st = []
st.append(x)
top = st[-1]
st.pop()
```

**Practice signal:** "matching pairs," "next greater/smaller element," "valid parentheses," "evaluate expression" → stack.

---

## Hashing

### Hash usage
- `unordered_map`/`unordered_set` (C++), `dict`/`set` (Python) — O(1) average lookup/insert, O(n) worst case (adversarial collisions, rare in interviews).
- Use for: existence checks, frequency counting, "have I seen this before," grouping (e.g., group anagrams by sorted-string key).

```cpp
std::unordered_map<int,int> freq;
for (int x : nums) freq[x]++;          // fine here — [] insert-on-miss is *intended*
if (freq.count(target)) { /* exists */ }
```
```python
from collections import Counter, defaultdict
freq = Counter(nums)
seen = set()
```

**Gotcha (repeated from file 1 because it's the #1 live-coding bug):** only use `map[key]` when you *want* insert-on-miss (like counting). For a pure existence check, use `.find()`/`.count()` so you don't silently mutate the map.

### Hash implementation (conceptual — sometimes asked directly)
- Array of buckets; `index = hash(key) % capacity`.
- Collisions resolved by **chaining** (each bucket holds a linked list/vector) or **open addressing** (probe to next slot).
- Resize (rehash) when load factor exceeds ~0.7 to keep O(1) average.
- If asked to implement one live: chaining with a `vector<vector<pair<K,V>>>` is the fastest to code correctly under pressure.

---

## Two Pointers

**Trigger:** sorted array/string, looking for a pair/triplet meeting a condition, or comparing from both ends.

```cpp
int l = 0, r = (int)nums.size() - 1;
while (l < r) {
    int sum = nums[l] + nums[r];
    if (sum == target) { /* found */ break; }
    else if (sum < target) l++;
    else r--;
}
```
```python
l, r = 0, len(nums) - 1
while l < r:
    s = nums[l] + nums[r]
    if s == target:
        break
    elif s < target:
        l += 1
    else:
        r -= 1
```

- Complexity: O(n) after an O(n log n) sort if not already sorted.
- **Gotcha:** cast `.size()` to `int` before subtracting, or compute `r = nums.size() - 1` carefully — subtracting 1 from an unsigned 0 wraps around.

## Prefix Sums

**Trigger:** many range-sum queries, or "subarray sum equals K" style problems.

```cpp
std::vector<int> prefix(n + 1, 0);
for (int i = 0; i < n; i++) prefix[i+1] = prefix[i] + nums[i];
// sum of nums[i..j] inclusive = prefix[j+1] - prefix[i]
```
```python
prefix = [0] * (n + 1)
for i in range(n):
    prefix[i+1] = prefix[i] + nums[i]
```

- For "subarray sum equals K" (not just a fixed range), combine prefix sum with a hash map of `{prefix_value: count_seen}` — this is the single most-reused combo trick in this tier.

## Kadane's Algorithm

**Trigger:** "maximum subarray sum" (contiguous).

```cpp
int best = nums[0], cur = nums[0];
for (int i = 1; i < (int)nums.size(); i++) {
    cur = std::max(nums[i], cur + nums[i]);
    best = std::max(best, cur);
}
```
```python
best = cur = nums[0]
for x in nums[1:]:
    cur = max(x, cur + x)
    best = max(best, cur)
```

- Core idea: at each index, decide whether extending the previous subarray or starting fresh gives a bigger sum. O(n), O(1) space.
- Variant to know: track start/end indices too if asked to return the subarray, not just the sum.

## Sliding Window — Fixed Size

**Trigger:** "window of size k," e.g., max sum of any k consecutive elements.

```cpp
int windowSum = 0;
for (int i = 0; i < k; i++) windowSum += nums[i];
int best = windowSum;
for (int i = k; i < (int)nums.size(); i++) {
    windowSum += nums[i] - nums[i - k];
    best = std::max(best, windowSum);
}
```
```python
window_sum = sum(nums[:k])
best = window_sum
for i in range(k, len(nums)):
    window_sum += nums[i] - nums[i - k]
    best = max(best, window_sum)
```

## Sliding Window — Variable Size

**Trigger:** "longest/shortest substring/subarray satisfying condition X."

```cpp
int l = 0, best = 0;
std::unordered_map<char,int> count;
for (int r = 0; r < (int)s.size(); r++) {
    count[s[r]]++;
    while (/* window invalid, e.g. count.size() > k */ false) {
        count[s[l]]--;
        if (count[s[l]] == 0) count.erase(s[l]);
        l++;
    }
    best = std::max(best, r - l + 1);
}
```
```python
l, best = 0, 0
count = {}
for r, ch in enumerate(s):
    count[ch] = count.get(ch, 0) + 1
    while False:  # window invalid condition
        count[s[l]] -= 1
        if count[s[l]] == 0:
            del count[s[l]]
        l += 1
    best = max(best, r - l + 1)
```

- Core idea: expand `r` every iteration; shrink `l` only while the window is invalid. Each pointer moves at most n times → O(n) total, not O(n²).
- **Gotcha:** the shrink condition is where bugs live — write out on paper what "invalid" means for the specific problem before coding.

**Practice signal for this whole file:** if the problem says "contiguous," "subarray," "substring," or gives you a sorted input and asks for pairs — one of these five patterns is almost always the intended solution.

---

## Cisco Practice Problems

Pulled from Cisco's LeetCode tag list, full methodology and cross-links in `dsa_10_cisco_problem_bank.md`. 🔥 = asked in the last 3–6 months, do these first.

- 🔥🔥 MEDIUM — [Contiguous Array](https://leetcode.com/problems/contiguous-array) — prefix sum + hashmap — asked in the last 3-6 months per both sources
- 🔥🔥 MEDIUM — [3Sum](https://leetcode.com/problems/3sum) — two pointers — asked in the last 6 months per both sources
- 🔥 EASY — [Two Sum](https://leetcode.com/problems/two-sum) — hashing — asked in the last 6 months per one source
- MEDIUM — [Maximum Subarray](https://leetcode.com/problems/maximum-subarray) — Kadane's
- MEDIUM — [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters) — sliding window
- EASY — [Valid Parentheses](https://leetcode.com/problems/valid-parentheses) — stack
- EASY — [Maximum Difference Between Increasing Elements](https://leetcode.com/problems/maximum-difference-between-increasing-elements)
- EASY — [Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array) — two pointers
- MEDIUM — [Decode String](https://leetcode.com/problems/decode-string) — stack
- EASY — [Merge Sorted Array](https://leetcode.com/problems/merge-sorted-array) — two pointers
- EASY — [Move Zeroes](https://leetcode.com/problems/move-zeroes) — two pointers
- MEDIUM — [Insert Delete GetRandom O(1)](https://leetcode.com/problems/insert-delete-getrandom-o1) — hashmap + array design
- MEDIUM — [Group Anagrams](https://leetcode.com/problems/group-anagrams) — hashing
- MEDIUM — [Reverse Words in a String](https://leetcode.com/problems/reverse-words-in-a-string) — two pointers
- MEDIUM — [Longest Consecutive Sequence](https://leetcode.com/problems/longest-consecutive-sequence) — hashing
- EASY — [Valid Palindrome](https://leetcode.com/problems/valid-palindrome) — two pointers
- MEDIUM — [Merge Intervals](https://leetcode.com/problems/merge-intervals) — sort + merge (full pattern in file 11)
- HARD — [Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum) — sliding window + monotonic deque
- MEDIUM — [Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k) — prefix sum + hashmap
- HARD — [Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring) — sliding window
- MEDIUM — [Expressive Words](https://leetcode.com/problems/expressive-words) — two pointers
- HARD — [Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram) — stack, monotonic
- EASY — [Remove All Adjacent Duplicates In String](https://leetcode.com/problems/remove-all-adjacent-duplicates-in-string) — stack
- MEDIUM — [Permutation in String](https://leetcode.com/problems/permutation-in-string) — fixed sliding window
- MEDIUM — [Sort Colors](https://leetcode.com/problems/sort-colors) — two pointers
- EASY — [Valid Anagram](https://leetcode.com/problems/valid-anagram) — hashing
- EASY — [Can Place Flowers](https://leetcode.com/problems/can-place-flowers) — array scan
- MEDIUM — [Container With Most Water](https://leetcode.com/problems/container-with-most-water) — two pointers

---

## Related Modules

- [DSA 11 — Intervals (sort+merge, adjacent pattern)](dsa_11_intervals.md)
- [DSA 10 — Cisco Problem Bank (practice for this module)](dsa_10_cisco_problem_bank.md)
