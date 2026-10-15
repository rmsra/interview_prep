# Tier 3 — Trees, Tries, Union-Find, Segment Tree

**Courses:** **DSA** · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · [C++](../cpp-course/cpp_00_README_start_here.md) · [Python](../python-course/python_00_README_start_here.md)
**DSA modules:** [Index](dsa_00_README_start_here.md) · [C++ Setup](dsa_01_cpp_setup_and_best_practices.md) · [Arrays/Hashing/2Ptr/Stack](dsa_02_arrays_hashing_two_pointers_stack.md) · [Binary Search/Linked List](dsa_03_binary_search_and_linked_list.md) · [Recursion/Sorting](dsa_04_recursion_and_sorting.md) · **Trees/Tries/Union-Find** · [Heap/Backtracking](dsa_06_heap_priority_queue_backtracking.md) · [Graphs](dsa_07_graphs.md) · [DP](dsa_08_dynamic_programming.md) · [Bit Manipulation](dsa_09_bit_manipulation.md) · [Cisco Problem Bank](dsa_10_cisco_problem_bank.md) · [Intervals](dsa_11_intervals.md)

---

Trees are the single highest-frequency topic across onsite loops after arrays/hashing. Know DFS/BFS traversal cold — everything else in this file is a variation on one of those two.

## Binary Tree

```cpp
struct TreeNode {
    int val;
    TreeNode *left, *right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
```
```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
```

## Depth-First Search (recursive — default choice)

```cpp
void dfs(TreeNode* node) {
    if (!node) return;
    // pre-order: process node here
    dfs(node->left);
    // in-order: process node here
    dfs(node->right);
    // post-order: process node here
}
```
```python
def dfs(node):
    if not node:
        return
    dfs(node.left)
    dfs(node.right)
```
- **Pre-order** (node, left, right): copying/serializing a tree.
- **In-order** (left, node, right): gives sorted order for a BST — this fact is a common "aha" answer interviewers look for.
- **Post-order** (left, right, node): deleting a tree, computing subtree-dependent values (height, diameter).

## Breadth-First Search (level order)

```cpp
std::queue<TreeNode*> q;
if (root) q.push(root);
while (!q.empty()) {
    int levelSize = q.size();
    for (int i = 0; i < levelSize; i++) {
        TreeNode* node = q.front(); q.pop();
        // process node
        if (node->left) q.push(node->left);
        if (node->right) q.push(node->right);
    }
}
```
```python
from collections import deque
q = deque([root]) if root else deque()
while q:
    for _ in range(len(q)):
        node = q.popleft()
        if node.left: q.append(node.left)
        if node.right: q.append(node.right)
```
- The `levelSize` snapshot trick (capture `q.size()` before the inner loop) is how you separate levels — needed for "level order traversal as list of lists," "right side view," "max width," etc. This exact pattern is worth memorizing verbatim.

## Iterative DFS (when recursion depth is a real risk, or explicitly asked)

```cpp
std::stack<TreeNode*> st;
if (root) st.push(root);
while (!st.empty()) {
    TreeNode* node = st.top(); st.pop();
    // process node (this order is pre-order)
    if (node->right) st.push(node->right);  // push right first
    if (node->left) st.push(node->left);    // so left is popped first
}
```
- For in-order iteratively (used for BST k-th smallest, etc.): push all left children onto a stack until null, pop, process, move to right child, repeat. Worth practicing once by hand since it's less intuitive than pre-order.

## Binary Search Tree (BST)

- Invariant: `left subtree < node < right subtree`, recursively.
- **Insert:**
```cpp
TreeNode* insert(TreeNode* root, int val) {
    if (!root) return new TreeNode(val);
    if (val < root->val) root->left = insert(root->left, val);
    else root->right = insert(root->right, val);
    return root;
}
```
- **Remove** (the classic three-case interview question):
  1. Node is a leaf → just delete it.
  2. Node has one child → replace node with that child.
  3. Node has two children → find the in-order successor (smallest value in the right subtree = leftmost node of `root->right`), copy its value into the node, then recursively delete the successor from the right subtree.
```cpp
TreeNode* remove(TreeNode* root, int key) {
    if (!root) return nullptr;
    if (key < root->val) root->left = remove(root->left, key);
    else if (key > root->val) root->right = remove(root->right, key);
    else {
        if (!root->left) return root->right;
        if (!root->right) return root->left;
        TreeNode* succ = root->right;
        while (succ->left) succ = succ->left;
        root->val = succ->val;
        root->right = remove(root->right, succ->val);
    }
    return root;
}
```
- **Gotcha:** the two-children case is where people freeze live. Say the plan out loud ("find in-order successor, copy value, delete successor recursively") before coding it — it signals you know the structure even if the code takes a moment.

## BST as Sets/Maps

- `std::set<T>` / `std::map<K,V>` in C++ and `sortedcontainers.SortedList`/`SortedDict` in Python (not built into base Python — base `set`/`dict` are hash-based, *not* ordered by value) are typically red-black trees under the hood: O(log n) insert/erase/find, and support ordered operations `std::set::lower_bound`, `.begin()`/`.rbegin()` for min/max.
- Use over a hash set/map when you need **ordering** (range queries, "next greater element in the set," k-th smallest) — otherwise `unordered_map`/dict is faster and simpler.

---

## Trie (Prefix Tree)

**Trigger:** "prefix," "autocomplete," "word search with a dictionary," "longest common prefix."

```cpp
struct TrieNode {
    TrieNode* children[26] = {};
    bool isEnd = false;
};
class Trie {
    TrieNode* root = new TrieNode();
public:
    void insert(const std::string& word) {
        TrieNode* cur = root;
        for (char c : word) {
            int i = c - 'a';
            if (!cur->children[i]) cur->children[i] = new TrieNode();
            cur = cur->children[i];
        }
        cur->isEnd = true;
    }
    bool search(const std::string& word) {
        TrieNode* cur = root;
        for (char c : word) {
            int i = c - 'a';
            if (!cur->children[i]) return false;
            cur = cur->children[i];
        }
        return cur->isEnd;
    }
};
```
```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False

class Trie:
    def __init__(self):
        self.root = TrieNode()
    def insert(self, word):
        cur = self.root
        for ch in word:
            cur = cur.children.setdefault(ch, TrieNode())
        cur.is_end = True
    def search(self, word):
        cur = self.root
        for ch in word:
            if ch not in cur.children:
                return False
            cur = cur.children[ch]
        return cur.is_end
```
- Fixed `children[26]` array (C++) is faster than a hashmap of children when alphabet is known (lowercase a-z) — use the array form unless the alphabet is large/sparse, then switch to `unordered_map<char, TrieNode*>`.
- Complexity: insert/search is O(L) where L = word length, independent of how many words are stored — that's the whole point of the structure.

## Union-Find (Disjoint Set Union)

**Trigger:** "connected components," "cycle detection in an undirected graph," "accounts merge," Kruskal's MST.

```cpp
class UnionFind {
    std::vector<int> parent, rank_;
public:
    UnionFind(int n) : parent(n), rank_(n, 0) {
        for (int i = 0; i < n; i++) parent[i] = i;
    }
    int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]);  // path compression
        return parent[x];
    }
    bool unite(int a, int b) {
        int ra = find(a), rb = find(b);
        if (ra == rb) return false;                        // already connected — cycle if edge
        if (rank_[ra] < rank_[rb]) std::swap(ra, rb);
        parent[rb] = ra;
        if (rank_[ra] == rank_[rb]) rank_[ra]++;            // union by rank
        return true;
    }
};
```
```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
    def unite(self, a, b):
        ra, rb = self.find(a), self.find(b)
        if ra == rb:
            return False
        if self.rank[ra] < self.rank[rb]:
            ra, rb = rb, ra
        self.parent[rb] = ra
        if self.rank[ra] == self.rank[rb]:
            self.rank[ra] += 1
        return True
```
- **Path compression + union by rank together** give near-O(1) amortized (technically O(α(n)), inverse Ackermann — effectively constant). Mention both by name; interviewers listen for it.

## Segment Tree

**Trigger:** many range-query + range-update operations on an array (range sum/min/max query with point or range updates) — the situation where prefix sums fall short because the array is being *mutated*.

```cpp
class SegmentTree {
    std::vector<int> tree;
    int n;
public:
    SegmentTree(std::vector<int>& nums) {
        n = nums.size();
        tree.assign(2 * n, 0);
        for (int i = 0; i < n; i++) tree[n + i] = nums[i];
        for (int i = n - 1; i > 0; i--) tree[i] = tree[2*i] + tree[2*i+1];
    }
    void update(int i, int val) {
        i += n;
        tree[i] = val;
        for (i /= 2; i >= 1; i /= 2) tree[i] = tree[2*i] + tree[2*i+1];
    }
    int query(int l, int r) {   // sum over [l, r)
        int res = 0;
        for (l += n, r += n; l < r; l /= 2, r /= 2) {
            if (l & 1) res += tree[l++];
            if (r & 1) res += tree[--r];
        }
        return res;
    }
};
```
- This is the **iterative bottom-up array-based** segment tree — fastest to code correctly live vs. the recursive pointer-based version. Both query and update are O(log n).
- Given your topic list flags this as advanced, and it genuinely is: if it comes up, stating the *concept* correctly (binary tree over array ranges, each node = aggregate of its range, O(log n) query/update) is often enough even if you don't finish the implementation — say that plan out loud first.

**Practice signal for this file:** "tree," "binary search tree," "prefix," "connected components," "range query with updates" map directly to the five structures above, in that order of frequency.

---

## Cisco Practice Problems

Full methodology in `dsa_10_cisco_problem_bank.md`.

- HARD — [Collect Coins in a Tree](https://leetcode.com/problems/collect-coins-in-a-tree) — tree + topological sort, genuinely hard, stretch goal
- HARD — [Word Search II](https://leetcode.com/problems/word-search-ii) — Trie + backtracking
- MEDIUM — [Power Grid Maintenance](https://leetcode.com/problems/power-grid-maintenance) — Union-Find + graph
- MEDIUM — [Minimum Time to Collect All Apples in a Tree](https://leetcode.com/problems/minimum-time-to-collect-all-apples-in-a-tree) — tree DFS
- MEDIUM — [Lowest Common Ancestor of a Binary Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree) — DFS/LCA
- MEDIUM — [Kth Smallest Element in a BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst) — in-order traversal
- HARD — [Design Search Autocomplete System](https://leetcode.com/problems/design-search-autocomplete-system) — Trie + heap
- EASY — [Two Sum IV - Input is a BST](https://leetcode.com/problems/two-sum-iv-input-is-a-bst) — Two Sum pattern over a BST's in-order sequence

---

## Related Modules

- [DSA 07 — Graphs (Union-Find is shared with Kruskal's)](dsa_07_graphs.md)
- [OS 01 — Scheduling (CFS uses a red-black tree)](../os-course/os_01_processes_threads_and_scheduling.md)
