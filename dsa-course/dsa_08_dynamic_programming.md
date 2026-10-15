# Dynamic Programming

**Courses:** **DSA** · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · [C++](../cpp-course/cpp_00_README_start_here.md) · [Python](../python-course/python_00_README_start_here.md)
**DSA modules:** [Index](dsa_00_README_start_here.md) · [C++ Setup](dsa_01_cpp_setup_and_best_practices.md) · [Arrays/Hashing/2Ptr/Stack](dsa_02_arrays_hashing_two_pointers_stack.md) · [Binary Search/Linked List](dsa_03_binary_search_and_linked_list.md) · [Recursion/Sorting](dsa_04_recursion_and_sorting.md) · [Trees/Tries/Union-Find](dsa_05_trees_tries_union_find.md) · [Heap/Backtracking](dsa_06_heap_priority_queue_backtracking.md) · [Graphs](dsa_07_graphs.md) · **DP** · [Bit Manipulation](dsa_09_bit_manipulation.md) · [Cisco Problem Bank](dsa_10_cisco_problem_bank.md) · [Intervals](dsa_11_intervals.md)

---

DP is recursion (see file 4) + memoization + recognizing overlapping subproblems and optimal substructure. Every pattern below follows the same three-question process:
1. What does `dp[i]` (or `dp[i][j]`) represent, in one sentence?
2. What's the recurrence relating `dp[i]` to smaller subproblems?
3. What's the base case?

Say these three things out loud before writing code — it's the single highest-leverage habit for DP under interview pressure, and it reads as structured thinking even when the code itself takes a few tries.

## 1-D DP

**Trigger:** "number of ways to..." / "minimum cost to reach step n" / decisions that only depend on a linear position.

Classic example — Climbing Stairs (1 or 2 steps at a time):
```cpp
// dp[i] = number of ways to reach step i
std::vector<int> dp(n + 1);
dp[0] = 1; dp[1] = 1;
for (int i = 2; i <= n; i++) dp[i] = dp[i-1] + dp[i-2];
```
```python
dp = [0] * (n + 1)
dp[0] = dp[1] = 1
for i in range(2, n + 1):
    dp[i] = dp[i-1] + dp[i-2]
```
- This is literally Fibonacci from file 4, renamed — recognizing that connection out loud is a good signal.
- **Space optimization:** if `dp[i]` only depends on the last 1-2 entries, drop the array and keep two rolling variables — O(1) space instead of O(n). Mention this even if you don't rewrite the code; it shows you see the full picture.

## 2-D DP

**Trigger:** two changing parameters — grid paths, two strings being compared, item + capacity (knapsack family).

Classic example — Unique Paths (grid, only move right/down):
```cpp
std::vector<std::vector<int>> dp(rows, std::vector<int>(cols, 1));
for (int r = 1; r < rows; r++)
    for (int c = 1; c < cols; c++)
        dp[r][c] = dp[r-1][c] + dp[r][c-1];
```
- `dp[r][c]` = "ways to reach cell (r,c)". Base case: first row and first column are all 1 (only one way to get there — straight line).
- **Space optimization:** if `dp[r][c]` only depends on the row above, you can collapse to a single 1-D array reused across rows — a common interviewer follow-up ("can you do this in O(cols) space?").

## 0/1 Knapsack

**Trigger:** items with weight+value, a capacity limit, **each item usable at most once**. "Can you make sum X using a subset of these numbers" is knapsack in disguise.

```cpp
// dp[i][w] = max value using first i items with capacity w
std::vector<std::vector<int>> dp(n + 1, std::vector<int>(capacity + 1, 0));
for (int i = 1; i <= n; i++) {
    for (int w = 0; w <= capacity; w++) {
        dp[i][w] = dp[i-1][w];                                   // don't take item i
        if (weights[i-1] <= w)
            dp[i][w] = std::max(dp[i][w], dp[i-1][w - weights[i-1]] + values[i-1]);  // take item i
    }
}
```
- **1-D space optimization (important, frequently asked as a follow-up):** iterate `w` from `capacity` down to `weights[i-1]` using a single 1-D array — iterating downward is what prevents an item from being counted twice within the same pass (which would turn it into unbounded knapsack by accident).
```cpp
std::vector<int> dp(capacity + 1, 0);
for (int i = 0; i < n; i++)
    for (int w = capacity; w >= weights[i]; w--)
        dp[w] = std::max(dp[w], dp[w - weights[i]] + values[i]);
```

## Unbounded Knapsack

**Trigger:** same as above but **items can be reused any number of times** — "coin change" (min coins to make amount, or number of ways to make amount) is the canonical version.

```cpp
// dp[w] = min coins to make amount w
std::vector<int> dp(amount + 1, INT_MAX);
dp[0] = 0;
for (int w = 1; w <= amount; w++)
    for (int coin : coins)
        if (coin <= w && dp[w - coin] != INT_MAX)
            dp[w] = std::min(dp[w], dp[w - coin] + 1);
```
- **The entire difference from 0/1 knapsack is the loop direction:** iterate `w` **upward** (or put the item loop outside and capacity loop increasing) so each item can be reused within the same pass. This one-line difference (`w-- ` vs `w++`) is the exact thing interviewers probe to check you actually understand the two patterns rather than having memorized one template.

## Longest Common Subsequence (LCS)

**Trigger:** comparing two strings/sequences, "longest common," "edit distance"-family problems.

```cpp
// dp[i][j] = LCS length of s1[0..i) and s2[0..j)
std::vector<std::vector<int>> dp(m + 1, std::vector<int>(n + 1, 0));
for (int i = 1; i <= m; i++) {
    for (int j = 1; j <= n; j++) {
        if (s1[i-1] == s2[j-1]) dp[i][j] = dp[i-1][j-1] + 1;
        else dp[i][j] = std::max(dp[i-1][j], dp[i][j-1]);
    }
}
// answer: dp[m][n]
```
- Base case: `dp[0][*] = dp[*][0] = 0` (empty string has LCS 0 with anything) — the extra row/column of zeros is why the loops start at 1 and index into the strings with `i-1`/`j-1`.
- This exact template (with small tweaks to the recurrence) also solves edit distance, longest common substring, and shortest common supersequence — worth knowing it's a template, not a one-off.

## Palindromes

**Trigger:** "longest palindromic substring/subsequence," "is this a valid palindrome partition."

```cpp
// dp[i][j] = true if s[i..j] (inclusive) is a palindrome
std::vector<std::vector<bool>> dp(n, std::vector<bool>(n, false));
for (int i = 0; i < n; i++) dp[i][i] = true;                 // single char
for (int len = 2; len <= n; len++) {
    for (int i = 0; i + len - 1 < n; i++) {
        int j = i + len - 1;
        if (s[i] == s[j]) {
            dp[i][j] = (len == 2) || dp[i+1][j-1];
        }
    }
}
```
- Fill order matters: shorter substrings must be computed before longer ones, since `dp[i][j]` depends on `dp[i+1][j-1]` — this is why the outer loop is over `len`, not `i`.
- **Longest palindromic subsequence** (not substring — characters don't need to be contiguous) is actually LCS of the string with its own reverse — another example of a "new-looking" problem reducing to a template you already have.

**Practice signal for this file:** if a naive recursive solution (file 4's `fib`) would recompute the same subproblem repeatedly, DP applies. Ask "what changes between subproblems?" — one changing value → 1-D; two → 2-D; "at most once per item" → 0/1 knapsack shape; "reuse allowed" → unbounded; "comparing two sequences" → LCS shape; "substring symmetric around itself" → palindrome shape.

---

## Cisco Practice Problems

Full methodology in `dsa_10_cisco_problem_bank.md`.

- MEDIUM — [House Robber](https://leetcode.com/problems/house-robber) — 1-D DP
- MEDIUM — [Longest Palindromic Substring](https://leetcode.com/problems/longest-palindromic-substring) — palindrome DP
- MEDIUM — [Edit Distance](https://leetcode.com/problems/edit-distance) — LCS-family
- EASY — [Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock) — 1-D DP
- MEDIUM — [Jump Game](https://leetcode.com/problems/jump-game) — 1-D DP / greedy
- MEDIUM — [Predict the Winner](https://leetcode.com/problems/predict-the-winner) — 2-D DP, game theory
- MEDIUM — [Palindromic Substrings](https://leetcode.com/problems/palindromic-substrings) — palindrome DP
- MEDIUM — [Longest Common Subsequence](https://leetcode.com/problems/longest-common-subsequence) — LCS
- HARD — [Burst Balloons](https://leetcode.com/problems/burst-balloons) — interval DP, stretch goal
- MEDIUM — [Maximum Length of Pair Chain](https://leetcode.com/problems/maximum-length-of-pair-chain) — LIS-family
- HARD — [Count of Integers](https://leetcode.com/problems/count-of-integers) — digit DP, niche, skip unless time allows
