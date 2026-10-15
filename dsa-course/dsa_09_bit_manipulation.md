# Bit Manipulation

**Courses:** **DSA** · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · [C++](../cpp-course/cpp_00_README_start_here.md) · [Python](../python-course/python_00_README_start_here.md)
**DSA modules:** [Index](dsa_00_README_start_here.md) · [C++ Setup](dsa_01_cpp_setup_and_best_practices.md) · [Arrays/Hashing/2Ptr/Stack](dsa_02_arrays_hashing_two_pointers_stack.md) · [Binary Search/Linked List](dsa_03_binary_search_and_linked_list.md) · [Recursion/Sorting](dsa_04_recursion_and_sorting.md) · [Trees/Tries/Union-Find](dsa_05_trees_tries_union_find.md) · [Heap/Backtracking](dsa_06_heap_priority_queue_backtracking.md) · [Graphs](dsa_07_graphs.md) · [DP](dsa_08_dynamic_programming.md) · **Bit Manipulation** · [Cisco Problem Bank](dsa_10_cisco_problem_bank.md) · [Intervals](dsa_11_intervals.md)

---

Lowest priority tier for this role given your timeline, but it comes up as quick "sanity check" questions and the tricks are cheap to memorize.

## Bit Operations — reference table

| Operation | C++ | Python | Notes |
|---|---|---|---|
| Get bit `i` | `(x >> i) & 1` | `(x >> i) & 1` | |
| Set bit `i` | `x \| (1 << i)` | `x \| (1 << i)` | |
| Clear bit `i` | `x & ~(1 << i)` | `x & ~(1 << i)` | |
| Toggle bit `i` | `x ^ (1 << i)` | `x ^ (1 << i)` | |
| Check power of 2 | `x > 0 && (x & (x-1)) == 0` | `x > 0 and (x & (x-1)) == 0` | clears lowest set bit; power of 2 has exactly one set bit |
| Clear lowest set bit | `x & (x-1)` | `x & (x-1)` | used to count set bits in a loop |
| Isolate lowest set bit | `x & (-x)` | `x & (-x)` | relies on two's complement |
| Count set bits | `__builtin_popcount(x)` | `bin(x).count('1')` or `x.bit_count()` (3.10+) | don't hand-roll a loop if a builtin exists — using it is the senior move |

## Common patterns

**XOR to find the unique element** (every element appears twice except one):
```cpp
int result = 0;
for (int x : nums) result ^= x;
// pairs cancel out (x ^ x == 0); the leftover is the unique value
```
```python
from functools import reduce
result = reduce(lambda a, b: a ^ b, nums, 0)
```
- Relies on: `x ^ x = 0` and `x ^ 0 = x`, and XOR being commutative/associative — so pairing order doesn't matter.

**Swap without a temp variable** (rarely needed in practice, occasionally asked as a trivia check):
```cpp
a ^= b; b ^= a; a ^= b;
```
- Know it exists; don't actually use it in real code — `std::swap` is clearer and the XOR version breaks if `a` and `b` are the same memory location.

**Check if two numbers have opposite signs:**
```cpp
bool oppositeSigns = (a ^ b) < 0;
```
- Works because the sign bit (MSB) differs iff the XOR's MSB is 1, which makes the result negative in two's complement.

**Gotcha:** in C++, left-shifting into or past the sign bit of a signed `int` is undefined behavior (e.g., `1 << 31` for a 32-bit signed int). Use `unsigned` or `1u << 31` when you need the full bit range, or `long long` if you need more than 31 usable bits.

**Practice signal:** "single number," "count bits," "power of two," "missing number in a range" (also solvable via XOR against the full expected range) → bit manipulation.

---

## Cisco Practice Problems

Full methodology in `dsa_10_cisco_problem_bank.md`.

- EASY — [Single Number](https://leetcode.com/problems/single-number) — XOR
- MEDIUM — [Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number) — bit-manip angle
- MEDIUM — [Single Element in a Sorted Array](https://leetcode.com/problems/single-element-in-a-sorted-array) — XOR trick
