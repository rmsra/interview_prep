# C++ Setup & Best Practices for the Interview

**Courses:** **DSA** · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · [C++](../cpp-course/cpp_00_README_start_here.md) · [Python](../python-course/python_00_README_start_here.md)
**DSA modules:** [Index](dsa_00_README_start_here.md) · **C++ Setup** · [Arrays/Hashing/2Ptr/Stack](dsa_02_arrays_hashing_two_pointers_stack.md) · [Binary Search/Linked List](dsa_03_binary_search_and_linked_list.md) · [Recursion/Sorting](dsa_04_recursion_and_sorting.md) · [Trees/Tries/Union-Find](dsa_05_trees_tries_union_find.md) · [Heap/Backtracking](dsa_06_heap_priority_queue_backtracking.md) · [Graphs](dsa_07_graphs.md) · [DP](dsa_08_dynamic_programming.md) · [Bit Manipulation](dsa_09_bit_manipulation.md) · [Cisco Problem Bank](dsa_10_cisco_problem_bank.md) · [Intervals](dsa_11_intervals.md)

---

## Which standard to use

- Default to **C++17** unless told otherwise — it's the most common baseline for onsite coding platforms (HackerRank, CoderPad, CodeSignal) as of 2026, and every idiom below works in it.
- **C++11** is the floor — structured bindings (`auto [a,b] = ...`), `if`-with-initializer, and `std::optional`/`std::string_view` are C++17+, so don't rely on them if you're not sure which standard the platform compiles with.
- **C++20** features (ranges, `<=>`, concepts) are unnecessary risk in an interview — they're less universally supported on coding platforms and buy you nothing a senior interviewer will credit. Skip them even if you know them.
- **Ask Tanique which standard/compiler they use** (already in your draft reply) — this removes the guesswork entirely.

## Compile flags to mentally assume

Most platforms compile roughly equivalent to:
```
g++ -std=c++17 -O2 -Wall solution.cpp
```
Assume warnings are NOT treated as errors, but write as if they were — uninitialized variables and signed/unsigned comparisons are the two most common silent bugs under interview pressure.

## STL cheat sheet — mapped to what you'll actually reach for

| Need | Container/Tool | Notes |
|---|---|---|
| Dynamic array | `std::vector<T>` | `.push_back()`, `.size()` is `size_t` (unsigned — watch loop comparisons with `int`) |
| Stack | `std::stack<T>` or just `std::vector<T>` | vector is often faster to write live (`.back()`, `.pop_back()`) |
| Queue | `std::queue<T>` | for BFS |
| Deque (both ends) | `std::deque<T>` | for sliding window max, 0-1 BFS |
| Hash set | `std::unordered_set<T>` | O(1) avg lookup; no ordering |
| Hash map | `std::unordered_map<K,V>` | same; `.count(k)` or `.find(k)` to check existence — never `[]` to check, it inserts a default value as a side effect |
| Ordered set/map | `std::set<T>` / `std::map<K,V>` | red-black tree, O(log n), sorted iteration |
| Priority queue (max-heap default) | `std::priority_queue<T>` | for min-heap: `priority_queue<T, vector<T>, greater<T>>` |
| Linked list | `std::list<T>` (doubly) or raw `struct Node*` | **write raw `struct Node*` in interviews** — that's what's being tested, not `std::list` |
| Fixed-size array | `std::array<T,N>` or C-style `T[N]` | fine either way, C-style is faster to type |
| Sort | `std::sort(v.begin(), v.end())` | O(n log n), introsort; pass a comparator lambda for custom order |
| Binary search on sorted range | `std::lower_bound` / `std::upper_bound` | returns iterator; subtract `.begin()` for index |

## Idioms that read as "senior" vs "junior" live

- Pass containers **by `const&`** unless you're mutating: `void f(const vector<int>& v)`.
- Range-based for when you don't need the index: `for (auto& x : v)`. Use `auto&` not `auto` to avoid copies on non-trivial types.
- Reserve when you know the size: `v.reserve(n)` before a loop of `push_back` — small thing, but signals you think about allocation, which will land well given your background.
- Prefer `emplace_back(args...)` over `push_back(T(args...))` for structs/pairs — constructs in place.
- `int` is fine for indices in interview code; don't over-engineer with `size_t` everywhere — it creates the classic `size_t - 1` underflow bug when a container might be empty. Cast deliberately when you subtract from `.size()`.

## Gotchas that cost points

- **`unordered_map::operator[]` on a `const` reference doesn't compile** and using it on a non-const map for a *lookup* silently inserts a zero/empty value — always `.find()` or `.count()` to check existence.
- **Integer overflow**: default `int` is 32-bit; sums of counts or products in DP problems overflow fast — use `long long` the moment you're accumulating.
- **Off-by-one on `.size()`**: it's `size_t` (unsigned) — `v.size() - 1` when `v` is empty wraps to a huge number and segfaults/loops forever on `v[i]` access. Guard empty containers first.
- **Iterator invalidation**: erasing from a `vector` while iterating with a range-for is UB — use the iterator returned by `.erase()` or iterate by index.
- **Passing large structs/strings by value** into recursive calls (common in backtracking) — pass by reference and undo the mutation instead of copying, or your solution will be correct but slow.

## Python note (for parity — this course is dual-language)

You said you want both — for a C++-onsite this is mainly so you can prototype logic fast in Python, then port. Python's advantages you'll lean on: `collections.deque`, `collections.defaultdict`, `heapq`, and slicing. The rest of this course gives both, but if the assessment is explicitly C++, treat the Python as your scratch-pad language, not your submission language.

---

## Related Modules

- [Networking 05 — Sockets Programming (C++ in a networked context)](../networking-course/networking_05_sockets_programming.md)
- [OS 02 — Synchronization (std::atomic, std::mutex)](../os-course/os_02_synchronization_and_concurrency.md)
