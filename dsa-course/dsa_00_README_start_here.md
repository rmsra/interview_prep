# Cisco Onsite DSA Course — Start Here

**Courses:** **DSA** · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · [C++](../cpp-course/cpp_00_README_start_here.md) · [Python](../python-course/python_00_README_start_here.md)
**DSA modules:** **Index** · [C++ Setup](dsa_01_cpp_setup_and_best_practices.md) · [Arrays/Hashing/2Ptr/Stack](dsa_02_arrays_hashing_two_pointers_stack.md) · [Binary Search/Linked List](dsa_03_binary_search_and_linked_list.md) · [Recursion/Sorting](dsa_04_recursion_and_sorting.md) · [Trees/Tries/Union-Find](dsa_05_trees_tries_union_find.md) · [Heap/Backtracking](dsa_06_heap_priority_queue_backtracking.md) · [Graphs](dsa_07_graphs.md) · [DP](dsa_08_dynamic_programming.md) · [Bit Manipulation](dsa_09_bit_manipulation.md) · [Cisco Problem Bank](dsa_10_cisco_problem_bank.md) · [Intervals](dsa_11_intervals.md)

---

Built around: your topic lists (Basic + Advanced), the roadmap image you attached, and Tanique's note that the onsite goes **DSA round → C++ round**.

Two sibling courses exist for the **C++ round** itself and for Python (a stated minimum qualification): [C++ language course](../cpp-course/cpp_00_README_start_here.md) and [Python 3 course](../python-course/python_00_README_start_here.md). This DSA course's file 01 is a fast STL cheat sheet — those two go much deeper into language semantics.

## How this maps to your roadmap image

The image is the NeetCode roadmap — it's drawn top-to-bottom in roughly the order most interviewers weight topics. I used that shape to prioritize your two lists instead of treating all ~50 topics as equally urgent. Tiers below = study order, not difficulty.

| Tier | Topics | Priority |
|---|---|---|
| 1 | Arrays & Hashing, Two Pointers, Stack | **Must-know** — shows up in almost every round |
| 2 | Binary Search, Sliding Window, Linked List | **Must-know** |
| 3 | Trees (incl. BST, DFS/BFS) | **Must-know** |
| 4 | Tries, Backtracking | **Should-know** |
| 5 | Heap/Priority Queue, Graphs | **Should-know** |
| 6 | 1-D DP, Union-Find, Segment Tree, Iterative DFS, Intervals | **Should-know** |
| 7 | 2-D DP, Bit Manipulation, weighted graphs (Dijkstra/Prim/Kruskal/Topo Sort) | **Nice-to-have** — good if time allows |
| — | Greedy, Math & Geometry | **Skip** — see note below |

**Note:** Intervals, Greedy, and Math & Geometry appear in your roadmap image but weren't in either of your two topic lists. Intervals got added back in (file 11) once the Cisco tag data showed Merge Intervals and Meeting Rooms II recurring. Greedy and Math & Geometry are still out — say the word if you want those too.

Given your background (7+ yrs C/C++, real-time systems, packet processing), the fundamentals — pointers, memory, recursion basics, arrays — are not where your risk is. Your risk is **pattern recall under time pressure** (this is LeetCode-style pattern matching, not systems design) and **C++ STL fluency for interview-speed coding** (day job is likely closer to raw arrays/custom allocators than `std::` containers). The course leans into both.

## Files in this course

1. [`dsa_01_cpp_setup_and_best_practices.md`](dsa_01_cpp_setup_and_best_practices.md) — **read this first.** C++ standard/version guidance, STL cheat sheet, interview-speed C++ idioms, common pitfalls.
2. [`dsa_02_arrays_hashing_two_pointers_stack.md`](dsa_02_arrays_hashing_two_pointers_stack.md)
3. [`dsa_03_binary_search_and_linked_list.md`](dsa_03_binary_search_and_linked_list.md) — sliding window is covered in file 2, grouped with two pointers
4. [`dsa_04_recursion_and_sorting.md`](dsa_04_recursion_and_sorting.md)
5. [`dsa_05_trees_tries_union_find.md`](dsa_05_trees_tries_union_find.md)
6. [`dsa_06_heap_priority_queue_backtracking.md`](dsa_06_heap_priority_queue_backtracking.md)
7. [`dsa_07_graphs.md`](dsa_07_graphs.md)
8. [`dsa_08_dynamic_programming.md`](dsa_08_dynamic_programming.md)
9. [`dsa_09_bit_manipulation.md`](dsa_09_bit_manipulation.md)
10. [`dsa_10_cisco_problem_bank.md`](dsa_10_cisco_problem_bank.md) — Cisco's actual LeetCode-tagged problems, prioritized by recency + frequency, mapped to the files above. Each module file (02–09, 11) also has the relevant slice appended at the bottom, so the practice problems sit next to the pattern they test.
11. [`dsa_11_intervals.md`](dsa_11_intervals.md) — added after the Cisco tag data showed this pattern recurring.

## Format of every topic entry

Each topic follows the same shape so you can skim fast:
- **What it is / when to reach for it** — the trigger phrase in a problem statement
- **Complexity** — time/space
- **C++** — idiomatic snippet, standard/library notes
- **Python** — idiomatic snippet, built-in notes
- **Gotchas** — the mistake that actually loses interview points
- **Practice signal** — how to recognize this pattern in a fresh problem

## Suggested order given your Sept 1 timeline

Do Tiers 1–3 solid (that's ~70% of what actually gets asked), then Tier 4–5, then whatever's left of Tier 6–7. Don't start at file 1 and read linearly to the end — triage by tier first.

## Before anything else: the recency-confirmed problems

Two independent tagged-problem sources now cross-check each other (see [`dsa_10_cisco_problem_bank.md`](dsa_10_cisco_problem_bank.md)). In order of how strong the signal is:

1. **[Shortest Path in a Grid with Obstacles Elimination](https://leetcode.com/problems/shortest-path-in-a-grid-with-obstacles-elimination)** — asked in the **last 30 days** per one source, and in both sources' 3- and 6-month windows. The single strongest signal in either dataset.
2. **[Contiguous Array](https://leetcode.com/problems/contiguous-array)** — both sources' 3- and 6-month windows.
3. **[3Sum](https://leetcode.com/problems/3sum)** — both sources' 6-month windows.
4. **[Two Sum](https://leetcode.com/problems/two-sum)** — one source's 6-month window.

Do these regardless of how far you get through the rest, in this order.
