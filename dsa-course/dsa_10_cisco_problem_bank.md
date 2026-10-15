# Cisco LeetCode Problem Bank

**Courses:** **DSA** · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · [C++](../cpp-course/cpp_00_README_start_here.md) · [Python](../python-course/python_00_README_start_here.md)
**DSA modules:** [Index](dsa_00_README_start_here.md) · [C++ Setup](dsa_01_cpp_setup_and_best_practices.md) · [Arrays/Hashing/2Ptr/Stack](dsa_02_arrays_hashing_two_pointers_stack.md) · [Binary Search/Linked List](dsa_03_binary_search_and_linked_list.md) · [Recursion/Sorting](dsa_04_recursion_and_sorting.md) · [Trees/Tries/Union-Find](dsa_05_trees_tries_union_find.md) · [Heap/Backtracking](dsa_06_heap_priority_queue_backtracking.md) · [Graphs](dsa_07_graphs.md) · [DP](dsa_08_dynamic_programming.md) · [Bit Manipulation](dsa_09_bit_manipulation.md) · **Cisco Problem Bank** · [Intervals](dsa_11_intervals.md)

---

Two independent company-tagged sources, cross-checked against each other:

- **Source A** (liquidslr's repo) — Last 3 Months, Last 6 Months, More Than 6 Months Ago, All-Time.
- **Source B** (second repo) — Last 30 Days, Last 3 Months, Last 6 Months, More Than 6 Months Ago, All-Time.

~90 unique problems combined. Two new problems came in from Source B that weren't in Source A: **Container With Most Water** (Module 02) and **Two Sum IV - Input is a BST** (Module 05) — both added to their module tables below.

## How priority is set

1. **Recency beats raw frequency**, and **agreement across both sources beats a single source.** A problem tagged recent by only one repo is a good signal; the same problem showing up recent in *both* independently-scraped repos is about as strong as this kind of data gets.
2. **All-time frequency**, for everything not in the recent window — each source normalizes frequency within its own file, so those numbers aren't comparable across sources or across time windows; they're only used to rank within a given module's list here.

## 🔥 Do these first — the recency signal, strongest to weakest

| Priority | Difficulty | Problem | Module | Why it's ranked here |
|---|---|---|---|---|
| 🔥🔥🔥 | HARD | [Shortest Path in a Grid with Obstacles Elimination](https://leetcode.com/problems/shortest-path-in-a-grid-with-obstacles-elimination) | 07 — Graphs (BFS) | The single strongest signal in either dataset — Source B shows it asked in the **last 30 days**, and it's in *both* sources' 3-month and 6-month windows. If you do one problem before Sept 1, do this one. |
| 🔥🔥 | MEDIUM | [Contiguous Array](https://leetcode.com/problems/contiguous-array) | 02 — Prefix Sum + Hashing | In *both* sources' 3-month and 6-month windows. |
| 🔥🔥 | MEDIUM | [3Sum](https://leetcode.com/problems/3sum) | 02 — Two Pointers | In *both* sources' 6-month windows. |
| 🔥 | EASY | [Two Sum](https://leetcode.com/problems/two-sum) | 02 — Hashing | In Source A's 6-month window; present but not flagged recent in Source B. Still worth leading with — it's the most common warm-up question industry-wide. |

If you only do four problems before Sept 1, do these, in this order.

---

## Full bank, by module

Each module's list is also appended directly to that module's file so it's next to the pattern when you study it. This file is the consolidated view. Sorted by all-time frequency within each module; 🔥 = recent (see above).

### Module 02 — Arrays, Hashing, Two Pointers, Stack

| Pri | Diff | Problem | Pattern note |
|---|---|---|---|
| 🔥 | EASY | [Two Sum](https://leetcode.com/problems/two-sum) | hashing |
| 🔥 | MEDIUM | [3Sum](https://leetcode.com/problems/3sum) | two pointers on sorted array |
| 🔥 | MEDIUM | [Contiguous Array](https://leetcode.com/problems/contiguous-array) | prefix sum + hashmap — same combo as Subarray Sum Equals K below |
| 69.0 | MEDIUM | [Maximum Subarray](https://leetcode.com/problems/maximum-subarray) | Kadane's, straight off the template in file 02 |
| 69.0 | MEDIUM | [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters) | sliding window, variable size |
| 61.6 | EASY | [Valid Parentheses](https://leetcode.com/problems/valid-parentheses) | stack |
| 61.6 | EASY | [Maximum Difference Between Increasing Elements](https://leetcode.com/problems/maximum-difference-between-increasing-elements) | single pass, track running min |
| 50.0 | EASY | [Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array) | two pointers |
| 50.0 | MEDIUM | [Decode String](https://leetcode.com/problems/decode-string) | stack (also solvable recursively — see file 04) |
| 44.0 | EASY | [Merge Sorted Array](https://leetcode.com/problems/merge-sorted-array) | two pointers, fill from the back |
| 44.0 | EASY | [Move Zeroes](https://leetcode.com/problems/move-zeroes) | two pointers |
| 44.0 | MEDIUM | [Insert Delete GetRandom O(1)](https://leetcode.com/problems/insert-delete-getrandom-o1) | hashmap + array, design |
| 44.0 | MEDIUM | [Group Anagrams](https://leetcode.com/problems/group-anagrams) | hashing on sorted-string key |
| 44.0 | MEDIUM | [Reverse Words in a String](https://leetcode.com/problems/reverse-words-in-a-string) | two pointers |
| 44.0 | MEDIUM | [Longest Consecutive Sequence](https://leetcode.com/problems/longest-consecutive-sequence) | hashing (Union-Find is an alternate approach — file 05) |
| 44.0 | EASY | [Valid Palindrome](https://leetcode.com/problems/valid-palindrome) | two pointers |
| 44.0 | MEDIUM | [Merge Intervals](https://leetcode.com/problems/merge-intervals) | sort + linear merge — primary home is now file 11 |
| 44.0 | HARD | [Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum) | sliding window + monotonic deque — extension beyond the base template |
| 35.7 | MEDIUM | [Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k) | prefix sum + hashmap, exactly the combo called out in file 02 |
| 35.7 | HARD | [Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring) | sliding window, variable size — the hardest version of the pattern |
| 35.7 | MEDIUM | [Expressive Words](https://leetcode.com/problems/expressive-words) | two pointers |
| 35.7 | HARD | [Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram) | stack + monotonic stack — extension |
| 35.7 | EASY | [Remove All Adjacent Duplicates In String](https://leetcode.com/problems/remove-all-adjacent-duplicates-in-string) | stack |
| 35.7 | MEDIUM | [Permutation in String](https://leetcode.com/problems/permutation-in-string) | sliding window, fixed size |
| 35.7 | MEDIUM | [Sort Colors](https://leetcode.com/problems/sort-colors) | two pointers (Dutch national flag) |
| 35.7 | EASY | [Valid Anagram](https://leetcode.com/problems/valid-anagram) | hashing |
| 35.7 | EASY | [Can Place Flowers](https://leetcode.com/problems/can-place-flowers) | single pass array scan |
| — | MEDIUM | [Container With Most Water](https://leetcode.com/problems/container-with-most-water) | two pointers — new from Source B, classic of the pattern |

### Module 03 — Binary Search, Linked List

| Pri | Diff | Problem | Pattern note |
|---|---|---|---|
| 71.0 | MEDIUM | [Implement Router](https://leetcode.com/problems/implement-router) | binary search + design |
| 66.8 | MEDIUM | [LRU Cache](https://leetcode.com/problems/lru-cache) | doubly linked list + hashmap — this is the canonical example already named in file 03 |
| 54.6 | EASY | [Linked List Cycle](https://leetcode.com/problems/linked-list-cycle) | fast/slow pointers, textbook |
| 54.6 | EASY | [Happy Number](https://leetcode.com/problems/happy-number) | fast/slow, cycle-detection outside a literal list |
| 50.0 | MEDIUM | [Add Two Numbers](https://leetcode.com/problems/add-two-numbers) | linked list traversal (also recursion — file 04) |
| 44.0 | EASY | [Reverse Linked List](https://leetcode.com/problems/reverse-linked-list) | linked list |
| 44.0 | HARD | [Reverse Nodes in k-Group](https://leetcode.com/problems/reverse-nodes-in-k-group) | linked list |
| 35.7 | MEDIUM | [Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array) | modified binary search |
| 35.7 | MEDIUM | [Search in Rotated Sorted Array II](https://leetcode.com/problems/search-in-rotated-sorted-array-ii) | modified binary search, with duplicates |
| 35.7 | MEDIUM | [Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number) | binary search or fast/slow (also bit manip — file 09) |
| 35.7 | EASY | [Convert Binary Number in a Linked List to Integer](https://leetcode.com/problems/convert-binary-number-in-a-linked-list-to-integer) | linked list traversal |
| 35.7 | MEDIUM | [Single Element in a Sorted Array](https://leetcode.com/problems/single-element-in-a-sorted-array) | binary search (XOR trick also works — file 09) |
| 35.7 | MEDIUM | [Time Based Key-Value Store](https://leetcode.com/problems/time-based-key-value-store) | binary search + design |

### Module 04 — Recursion & Sorting

| Pri | Diff | Problem | Pattern note |
|---|---|---|---|
| 35.7 | MEDIUM | [Pow(x, n)](https://leetcode.com/problems/powx-n) | recursion — fast exponentiation, halving each call |
| 35.7 | HARD | [Integer to English Words](https://leetcode.com/problems/integer-to-english-words) | recursion + heavy string simulation — low pattern value, mostly bookkeeping |

### Module 05 — Trees, Tries, Union-Find, Segment Tree

| Pri | Diff | Problem | Pattern note |
|---|---|---|---|
| 99.0 | HARD | [Collect Coins in a Tree](https://leetcode.com/problems/collect-coins-in-a-tree) | tree + topological sort, genuinely hard — stretch goal, not a must |
| 84.5 | HARD | [Word Search II](https://leetcode.com/problems/word-search-ii) | Trie + backtracking (grid) — also file 06 |
| 71.0 | MEDIUM | [Power Grid Maintenance](https://leetcode.com/problems/power-grid-maintenance) | Union-Find + graph — advanced |
| 58.4 | MEDIUM | [Minimum Time to Collect All Apples in a Tree](https://leetcode.com/problems/minimum-time-to-collect-all-apples-in-a-tree) | tree DFS |
| 44.0 | MEDIUM | [Lowest Common Ancestor of a Binary Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree) | DFS — classic LCA pattern, worth memorizing as its own micro-template |
| 35.7 | MEDIUM | [Kth Smallest Element in a BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst) | BST in-order traversal — exact application of "in-order = sorted" from file 05 |
| 35.7 | HARD | [Design Search Autocomplete System](https://leetcode.com/problems/design-search-autocomplete-system) | Trie + heap, design — advanced |
| — | EASY | [Two Sum IV - Input is a BST](https://leetcode.com/problems/two-sum-iv-input-is-a-bst) | new from Source B — the Two Sum pattern (hashset or two pointers) run over a BST's in-order sequence instead of an array; also cross-lists to Module 02 |

### Module 06 — Heap / Priority Queue, Backtracking

| Pri | Diff | Problem | Pattern note |
|---|---|---|---|
| 84.5 | HARD | [Word Search II](https://leetcode.com/problems/word-search-ii) | backtracking + Trie — also file 05 |
| 44.0 | MEDIUM | [Word Search](https://leetcode.com/problems/word-search) | backtracking — exact "tree maze" grid template from file 06 |
| 44.0 | MEDIUM | [Letter Combinations of a Phone Number](https://leetcode.com/problems/letter-combinations-of-a-phone-number) | backtracking |
| 44.0 | MEDIUM | [Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii) | heap of end times — primary home is now file 11 |
| 35.7 | MEDIUM | [Permutations](https://leetcode.com/problems/permutations) | backtracking — exact template match |
| 35.7 | HARD | [Find Servers That Handled Most Number of Requests](https://leetcode.com/problems/find-servers-that-handled-most-number-of-requests) | heap + simulation — advanced |
| 35.7 | HARD | [Design Search Autocomplete System](https://leetcode.com/problems/design-search-autocomplete-system) | heap — also file 05 |

### Module 07 — Graphs

| Pri | Diff | Problem | Pattern note |
|---|---|---|---|
| 🔥🔥🔥 | HARD | [Shortest Path in a Grid with Obstacles Elimination](https://leetcode.com/problems/shortest-path-in-a-grid-with-obstacles-elimination) | BFS on a grid, with a state twist — asked in the last 30 days per Source B |
| 71.0 | MEDIUM | [Power Grid Maintenance](https://leetcode.com/problems/power-grid-maintenance) | graph + Union-Find — also file 05 |
| 50.0 | MEDIUM | [Number of Islands](https://leetcode.com/problems/number-of-islands) | matrix DFS/BFS — exact template match, and given your networking background this one should feel very natural |
| 35.7 | MEDIUM | [Course Schedule](https://leetcode.com/problems/course-schedule) | topological sort — exact match (Kahn's algorithm from file 07) |
| 35.7 | HARD | [Word Ladder](https://leetcode.com/problems/word-ladder) | BFS over an implicit graph |
| 35.7 | MEDIUM | [Snakes and Ladders](https://leetcode.com/problems/snakes-and-ladders) | BFS — creative application, board as graph |
| 35.7 | MEDIUM | [Cheapest Flights Within K Stops](https://leetcode.com/problems/cheapest-flights-within-k-stops) | Dijkstra-style/Bellman-Ford-ish with a stop limit — advanced |

### Module 08 — Dynamic Programming

| Pri | Diff | Problem | Pattern note |
|---|---|---|---|
| 100.0 | MEDIUM | [House Robber](https://leetcode.com/problems/house-robber) | 1-D DP — the canonical example, exact match to file 08 |
| 99.5 | MEDIUM | [Longest Palindromic Substring](https://leetcode.com/problems/longest-palindromic-substring) | palindrome DP |
| 61.6 | MEDIUM | [Edit Distance](https://leetcode.com/problems/edit-distance) | LCS-family template, named directly in file 08 |
| 44.0 | EASY | [Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock) | 1-D DP (also solvable greedily in one pass) |
| 44.0 | MEDIUM | [Jump Game](https://leetcode.com/problems/jump-game) | 1-D DP / greedy |
| 44.0 | MEDIUM | [Predict the Winner](https://leetcode.com/problems/predict-the-winner) | 2-D DP, game theory — advanced |
| 35.7 | MEDIUM | [Palindromic Substrings](https://leetcode.com/problems/palindromic-substrings) | palindrome DP — exact match |
| 35.7 | MEDIUM | [Longest Common Subsequence](https://leetcode.com/problems/longest-common-subsequence) | LCS — exact template match |
| 35.7 | HARD | [Burst Balloons](https://leetcode.com/problems/burst-balloons) | interval DP — hard, stretch goal |
| 35.7 | MEDIUM | [Maximum Length of Pair Chain](https://leetcode.com/problems/maximum-length-of-pair-chain) | LIS-family DP — advanced |
| 35.7 | HARD | [Count of Integers](https://leetcode.com/problems/count-of-integers) | digit DP — not in your course, genuinely niche, skip unless you have time to spare |

### Module 09 — Bit Manipulation

| Pri | Diff | Problem | Pattern note |
|---|---|---|---|
| 35.7 | EASY | [Single Number](https://leetcode.com/problems/single-number) | XOR — exact match to file 09 |
| 35.7 | MEDIUM | [Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number) | bit-manip angle (also binary search/fast-slow — file 03) |
| 35.7 | MEDIUM | [Single Element in a Sorted Array](https://leetcode.com/problems/single-element-in-a-sorted-array) | XOR trick (also binary search — file 03) |

### Module 11 — Intervals

| Pri | Diff | Problem | Pattern note |
|---|---|---|---|
| 44.0 | MEDIUM | [Merge Intervals](https://leetcode.com/problems/merge-intervals) | sort + linear merge, the core pattern |
| 44.0 | MEDIUM | [Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii) | min-heap of end times |

---

## Not covered by your current course — flagged, not silently dropped

A handful of tagged problems don't map cleanly to any of your nine modules. Rather than force-fit them, here's what they are and what I'd do with each:

- **SQL/database:** [Find Third Transaction](https://leetcode.com/problems/find-third-transaction) (71.0), [Rising Temperature](https://leetcode.com/problems/rising-temperature) (35.7) — these are LeetCode database questions, not DSA. Almost certainly irrelevant to a DSA/C++ onsite round; skip.
- **Math & Geometry:** [Max Points on a Line](https://leetcode.com/problems/max-points-on-a-line) (54.6), [Find the Largest Area of Square Inside Two Rectangles](https://leetcode.com/problems/find-the-largest-area-of-square-inside-two-rectangles) (64.4) — this is the category you and I already agreed to skip. Consistent with that call.
- **Trivial / low prep value:** [Fizz Buzz](https://leetcode.com/problems/fizz-buzz) (72.9), [Number of Valid Words in a Sentence](https://leetcode.com/problems/number-of-valid-words-in-a-sentence) (66.8), [Check if Binary String Has at Most One Segment of Ones](https://leetcode.com/problems/check-if-binary-string-has-at-most-one-segment-of-ones) (64.4), [Palindrome Number](https://leetcode.com/problems/palindrome-number) (35.7) — no real pattern to internalize, just careful implementation. Skim once, don't drill.
- **Niche math/counting** (low value given your timeline): [Sum of k-Mirror Numbers](https://leetcode.com/problems/sum-of-k-mirror-numbers) (66.8), [Minimum Cost to Make Array Equal](https://leetcode.com/problems/minimum-cost-to-make-array-equal) (66.8), [Number of Equal Count Substrings](https://leetcode.com/problems/number-of-equal-count-substrings) (64.4), [Minimum Sensors to Cover Grid](https://leetcode.com/problems/minimum-sensors-to-cover-grid) (64.4).
- **Matrix simulation** (no dedicated pattern file — these are about careful indexing, not a named algorithm): [Rotate Image](https://leetcode.com/problems/rotate-image) (83.5), [Spiral Matrix](https://leetcode.com/problems/spiral-matrix) (80.1), [Lucky Numbers in a Matrix](https://leetcode.com/problems/lucky-numbers-in-a-matrix) (83.5), [Diagonal Traverse](https://leetcode.com/problems/diagonal-traverse) (61.6). Worth one pass each given the frequency, but there's no transferable "pattern" to teach the way there is for the rest of the course.

**Resolved:** Merge Intervals and Meeting Rooms II both showed up at 44.0 — enough of a signal that file 11 (Intervals) now covers this pattern directly.

**New from Source B, no gaps to flag:** both additions (Container With Most Water, Two Sum IV - Input is a BST) map cleanly onto patterns already in the course — nothing new to add here.
