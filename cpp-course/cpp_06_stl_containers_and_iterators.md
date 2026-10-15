# STL Containers & Iterators — Beyond the Cheat Sheet

**Courses:** [DSA](../dsa-course/dsa_00_README_start_here.md) · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · **C++** · [Python](../python-course/python_00_README_start_here.md)  
**C++ modules:** [Index](cpp_00_README_start_here.md) · [Language Evolution 11-17](cpp_01_language_evolution_11_to_17.md) · [OOP/Class Design](cpp_02_oop_and_class_design.md) · [RAII/Smart Pointers](cpp_03_raii_and_smart_pointers.md) · [Move Semantics](cpp_04_move_semantics_and_value_categories.md) · [Templates](cpp_05_templates_and_generic_programming.md) · **STL Containers/Iterators** · [Concurrency in C++](cpp_07_concurrency_in_cpp.md) · [Exceptions](cpp_08_exceptions_and_error_handling.md) · [Interview Trivia](cpp_09_interview_common_questions.md)

---

`dsa-course/dsa_01_cpp_setup_and_best_practices.md` covers *which* container to reach for. This file covers what interviewers actually probe once they know you know the names: **complexity guarantees precisely**, and **iterator invalidation rules**, which is where "I use `vector`" and "I know when a `vector` iterator becomes a landmine" diverge.

## Complexity Guarantees — precise, not "it's fast"

| Container | Access | Insert/erase (typical position) | Notes |
|---|---|---|---|
| `vector` | O(1) random access | O(1) amortized at end; O(n) elsewhere (shifts elements) | contiguous memory — cache-friendly, the default reach |
| `deque` | O(1) random access | O(1) at both ends; O(n) in the middle | not contiguous (chunked) — slightly worse cache behavior than `vector` for pure iteration |
| `list` (doubly-linked) | O(n) — no random access | O(1) anywhere, **given an iterator to the position** | the O(1) erase/insert only holds if you already have the iterator; *finding* the position is still O(n) |
| `set`/`map` (red-black tree) | O(log n) find | O(log n) insert/erase | ordered iteration; **iterators remain valid** across insert/erase of *other* elements (a real, useful guarantee) |
| `unordered_set`/`unordered_map` (hash table) | O(1) average, O(n) worst case | O(1) average, O(n) worst case | worst case from hash collisions/adversarial input — rare in practice, real in theory, worth naming if asked why `map` might be preferred despite being asymptotically slower |

**A genuinely good interview answer:** "`unordered_map` is O(1) average, but the constant factor and worst-case behavior matter — if I need a *guaranteed* bound (e.g., in a latency-sensitive path) or ordered iteration, I'd reach for `map` despite the O(log n), because O(1)-average with an occasional O(n) rehash spike can be worse for tail latency than a consistent O(log n)." This is exactly the kind of "average vs. worst-case, and which matters for this workload" reasoning that reads as senior.

## Iterator Invalidation — the rules that actually cause bugs

This is the single highest-yield topic in this file — invalidated-iterator bugs are real, common, and exactly the kind of thing a "write correct code live" interviewer watches for.

| Operation | Effect on iterators/references/pointers |
|---|---|
| `vector::push_back` (no reallocation needed) | end() iterator invalidated; all others remain valid |
| `vector::push_back` (triggers reallocation) | **everything invalidated** — all iterators, pointers, and references, since the entire buffer moved |
| `vector::erase(it)` | `it` and everything **after** it invalidated; everything before remains valid |
| `deque::push_front`/`push_back` | all iterators invalidated; references/pointers to existing elements remain valid (a real, specific asymmetry worth knowing) |
| `list::erase(it)` | only `it` itself invalidated — every other iterator, including adjacent ones, remains valid |
| `map`/`set::erase(it)` | only `it` itself invalidated — same guarantee as `list`, for the same underlying reason (node-based structure, erasing one node doesn't move any others) |
| `unordered_map::insert` (triggers rehash) | all iterators invalidated; **references/pointers to existing elements remain valid** even across a rehash (a specific, useful, standard-guaranteed exception) |

**The classic live-coding bug this produces:**
```cpp
for (auto it = v.begin(); it != v.end(); ) {
    if (should_remove(*it)) {
        it = v.erase(it);   // erase returns the NEXT valid iterator — must capture it
    } else {
        ++it;                // only advance if you didn't erase
    }
}
```
Writing `v.erase(it); ++it;` (incrementing an iterator that `erase` already invalidated) is undefined behavior — a very common mistake under interview pressure. `erase` returning the next valid iterator specifically exists to make this loop correct; know to use it.

## `std::vector` Growth Strategy

- Growth is **geometric** (typically doubling, though the standard doesn't mandate an exact factor) rather than growing by a fixed increment — this is what makes `push_back` **O(1) amortized**: the expensive O(n) reallocation happens increasingly rarely relative to the number of cheap O(1) pushes between reallocations.
- **`reserve(n)` vs. `resize(n)` — a real, common confusion:** `reserve` allocates capacity for `n` elements without constructing any of them (`size()` unchanged); `resize` actually changes `size()`, default-constructing (or value-initializing) new elements if growing, or destroying elements if shrinking. Call `reserve` when you know an upcoming loop will `push_back` roughly `n` times and want to avoid repeated reallocation; call `resize` when you need the container to actually *contain* `n` elements right now.

## Iterator Categories — worth knowing the hierarchy exists

From least to most capable: **Input** (single-pass, read-only, e.g. reading from a stream) → **Forward** (multi-pass, read, e.g. `forward_list`) → **Bidirectional** (also `--`, e.g. `list`, `map`, `set`) → **Random Access** (also `+n`, `[]`, e.g. `vector`, `deque`). Algorithms are written against the *minimum* category they need — `std::sort` requires random access (needs `[]`), which is precisely why `std::sort` doesn't compile on `std::list` (you'd use `list::sort()`, the container's own member function, instead). Knowing *why* `std::sort(list.begin(), list.end())` fails to compile — not just that it does — is a good, precise thing to have ready.

## `std::sort` Complexity & Stability

- `std::sort`: O(n log n) average and worst case (typically introsort — quicksort with a heapsort fallback to guarantee worst-case bound, the same idea flagged in the DSA course's sorting file) — but **not stable** (equal elements may be reordered relative to each other).
- `std::stable_sort`: O(n log n) average, but **O(n log² n) worst case** if extra memory isn't available (typically implemented as merge sort) — and, as the name says, preserves relative order of equal elements.
- **When stability actually matters, concretely:** sorting a list of objects by one field while wanting ties to remain in their original (e.g., previously-sorted-by-a-different-field) order — a genuinely common real requirement, and knowing to reach for `stable_sort` specifically for it (rather than not realizing plain `sort` doesn't guarantee this) is a good, precise signal.

## Practice signal

"What happens to my iterators if I erase from a vector while iterating" is close to a guaranteed question in some form — know the `erase`-returns-next-iterator pattern cold, be ready to write it correctly live. "Why would you use `map` over `unordered_map`" → ordered iteration, and/or worst-case guarantee over average-case — both are legitimate, precise answers.

---

## Related Modules

- [DSA 01 — C++ Setup (which container, fast)](../dsa-course/dsa_01_cpp_setup_and_best_practices.md)
- [DSA 04 — Sorting (stability, std::sort internals)](../dsa-course/dsa_04_recursion_and_sorting.md)
