# Data Structures & Algorithms – Data Structures

**Question**: How does a dynamic array achieve amortized O(1) insertion?

**Answer**: A dynamic array (ArrayList, Python list, vector) allocates a fixed-capacity backing array. When full, it allocates a new array of double the capacity and copies elements over — an O(n) operation. Because this doubling occurs only after n/2 inserts, the cost is amortized over all insertions, yielding O(1) amortized per insert. Geometric growth ensures the resizing cost is spread evenly.

```java
int size = 0;
int[] arr = new int[16];

void add(int value) {
    if (size == arr.length) {
        arr = Arrays.copyOf(arr, arr.length * 2);
    }
    arr[size++] = value;
}
```

---

**Question**: Explain the two-pointer technique and when to apply it.

**Answer**: Two pointers traverse a sequence from different positions — both from the start, one fast and one slow, or one from each end (opposite ends). It is used for sorted array problems (pair sum, deduplication), in-place reversal, and cycle detection. The key insight is that two pointers can reduce a nested loop (O(n²)) to a single pass (O(n)).

```python
# Pair sum in sorted array — O(n)
def pair_sum(arr, target):
    left, right = 0, len(arr) - 1
    while left < right:
        s = arr[left] + arr[right]
        if s == target:
            return [left, right]
        elif s < target:
            left += 1
        else:
            right -= 1
    return []
```

---

**Question**: Describe the sliding window pattern and its complexity.

**Answer**: A sliding window maintains a contiguous subarray satisfying a condition, expanding the right boundary and contracting the left. It converts O(n²) brute-force substring problems into O(n) single-pass solutions. Use it for problems like longest substring without repeating characters, maximum sum subarray of size k, or minimum window containing all targets.

```python
# Longest substring without repeating characters — O(n)
def length_of_longest_substring(s: str) -> int:
    char_set = set()
    left = max_len = 0
    for right in range(len(s)):
        while s[right] in char_set:
            char_set.remove(s[left])
            left += 1
        char_set.add(s[right])
        max_len = max(max_len, right - left + 1)
    return max_len
```

---

**Question**: What are prefix sums and how do you use them for range queries?

**Answer**: A prefix sum array stores cumulative sums from index 0 to i, allowing range sum queries in O(1) after O(n) preprocessing: sum(l..r) = prefix[r+1] - prefix[l] (using 0-indexed prefix array with prefix[0] = 0). This is the foundation for difference arrays, 2D prefix sums (image processing), and running-sum problems. It trades O(n) precomputation for O(1) queries.

```python
def prefix_sum(arr):
    prefix = [0] * (len(arr) + 1)
    for i, v in enumerate(arr):
        prefix[i + 1] = prefix[i] + v
    return prefix

# Range sum [l, r] inclusive: prefix[r+1] - prefix[l]
```

---

**Question**: Compare KMP, Rabin-Karp, and trie-based string matching.

**Answer**: KMP uses a prefix function (LPS array) to skip re-examining matched characters, achieving O(n + m) for pattern matching. Rabin-Karp uses rolling hash for O(n + m) average case (but O(nm) worst-case with hash collisions). A trie (prefix tree) stores all strings in a shared prefix tree, enabling O(k) lookup per string (k = key length) and efficient autocomplete. Use KMP for single-pattern matching, Rabin-Karp for multiple patterns, and tries for dictionary-based problems.

```python
# KMP: build LPS (longest prefix suffix) array
def compute_lps(pattern):
    lps = [0] * len(pattern)
    j = 0
    for i in range(1, len(pattern)):
        while j > 0 and pattern[i] != pattern[j]:
            j = lps[j - 1]
        if pattern[i] == pattern[j]:
            j += 1
            lps[i] = j
    return lps
```

---

**Question**: Implement linked list reversal iteratively and recursively.

**Answer**: Iterative reversal uses three pointers (prev, curr, next) to reverse links in a single pass — O(n) time, O(1) space. Recursive reversal reverses the sublist after the current node, then points the next node back to current — O(n) time, O(n) space (call stack). The iterative version is preferred in interviews for its constant space.

```python
# Iterative
def reverse(head):
    prev, curr = None, head
    while curr:
        next_node = curr.next
        curr.next = prev
        prev = curr
        curr = next_node
    return prev

# Recursive
def reverse_recursive(head):
    if not head or not head.next:
        return head
    new_head = reverse_recursive(head.next)
    head.next.next = head
    head.next = None
    return new_head
```

---

**Question**: Explain Floyd's cycle detection (tortoise and hare) algorithm.

**Answer**: Use a slow pointer (moves 1 step) and a fast pointer (moves 2 steps). If they meet, a cycle exists. To find the cycle start, reset one pointer to head and advance both at 1 step until they meet again — that node is the cycle entry. Time O(n), space O(1). The key mathematical property is that the distance from head to cycle start equals the distance from meeting point to cycle start.

```python
def has_cycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            return True
    return False

def detect_cycle_start(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            slow = head
            while slow != fast:
                slow = slow.next
                fast = fast.next
            return slow
    return None
```

---

**Question**: How do you merge two sorted linked lists?

**Answer**: Use a dummy head node and a tail pointer. Compare the heads of both lists, attach the smaller node to tail, and advance that list. When one list is exhausted, attach the remainder of the other. Time O(n + m), space O(1). This is the foundation for merge sort on linked lists and the merge step of merge sort.

```python
def merge_two_lists(l1, l2):
    dummy = tail = ListNode()
    while l1 and l2:
        if l1.val < l2.val:
            tail.next = l1
            l1 = l1.next
        else:
            tail.next = l2
            l2 = l2.next
        tail = tail.next
    tail.next = l1 or l2
    return dummy.next
```

---

**Question**: Describe the monotonic stack pattern with next greater element.

**Answer**: A monotonic stack maintains elements in increasing or decreasing order by popping smaller/larger elements as new elements arrive. For Next Greater Element, iterate right-to-left, keeping a stack of candidates. Each element pops smaller values from the stack, then the top is the next greater. Time O(n) — each element pushed and popped at most once. Use for stock span, largest rectangle in histogram, and trapping rain water.

```python
def next_greater_element(nums):
    stack, result = [], [-1] * len(nums)
    for i in range(len(nums) - 1, -1, -1):
        while stack and stack[-1] <= nums[i]:
            stack.pop()
        result[i] = stack[-1] if stack else -1
        stack.append(nums[i])
    return result
```

---

**Question**: Implement a queue using two stacks.

**Answer**: Use an input stack for enqueue and an output stack for dequeue. On dequeue, if the output stack is empty, pop all elements from the input stack and push onto the output stack (reversing order). Each element is moved at most twice, giving amortized O(1) per operation. This is the classic two-stack queue and is sometimes asked as a warm-up.

```python
class QueueUsingStacks:
    def __init__(self):
        self.in_stack = []
        self.out_stack = []

    def enqueue(self, x):
        self.in_stack.append(x)

    def dequeue(self):
        if not self.out_stack:
            while self.in_stack:
                self.out_stack.append(self.in_stack.pop())
        return self.out_stack.pop()
```

---

**Question**: Compare min-heap vs max-heap and explain heapify.

**Answer**: A min-heap has the smallest element at the root; a max-heap has the largest at the root. Both are complete binary trees stored in arrays where parent ≤ children (min) or parent ≥ children (max). Heapify (build heap) runs in O(n) by sifting down from the last non-leaf node to the root. In Python, `heapq` is a min-heap; for a max-heap, negate values. Use heaps for priority queues, kth largest/smallest, and median tracking.

```python
import heapq

# Min-heap (default)
heap = [3, 1, 4, 1, 5]
heapq.heapify(heap)          # O(n)
heapq.heappush(heap, 2)      # O(log n)
smallest = heapq.heappop(heap)  # O(log n)

# Max-heap via negation
max_heap = [-x for x in [3, 1, 4, 1, 5]]
heapq.heapify(max_heap)
largest = -heapq.heappop(max_heap)
```

---

**Question**: How do you solve sliding window maximum using a deque?

**Answer**: Maintain a deque storing indices with values in decreasing order. For each new element, remove smaller elements from the back (they can never be the maximum for future windows), then add the new index. Remove the front index if it falls outside the window. The front is always the maximum for the current window. Time O(n) — each index added and removed once.

```python
from collections import deque

def max_sliding_window(nums, k):
    dq = deque()
    result = []
    for i, v in enumerate(nums):
        while dq and nums[dq[-1]] < v:
            dq.pop()
        dq.append(i)
        if dq[0] <= i - k:
            dq.popleft()
        if i >= k - 1:
            result.append(nums[dq[0]])
    return result
```

---

**Question**: Describe BST properties, insertion, deletion, and traversal orders.

**Answer**: A BST has the property that for any node, all keys in its left subtree are smaller and all keys in its right subtree are larger. Insertion traverses left or right until finding a null child — O(h). Deletion has three cases: leaf (remove directly), one child (replace with child), two children (replace with inorder successor/predecessor). Traversals: inorder (sorted order), preorder (root first, for serialization), postorder (children first, for deletion).

```python
class TreeNode:
    def __init__(self, val):
        self.val = val
        self.left = self.right = None

def inorder(root):
    return inorder(root.left) + [root.val] + inorder(root.right) if root else []

def insert(root, val):
    if not root: return TreeNode(val)
    if val < root.val: root.left = insert(root.left, val)
    else: root.right = insert(root.right, val)
    return root

def delete(root, val):
    if not root: return None
    if val < root.val: root.left = delete(root.left, val)
    elif val > root.val: root.right = delete(root.right, val)
    else:
        if not root.left: return root.right
        if not root.right: return root.left
        successor = root.right
        while successor.left: successor = successor.left
        root.val = successor.val
        root.right = delete(root.right, successor.val)
    return root
```

---

**Question**: Explain AVL tree rotations and balancing.

**Answer**: An AVL tree maintains the invariant that the height difference between left and right subtrees (balance factor) is -1, 0, or 1. When an insertion or deletion violates this, rotations restore balance: left rotation, right rotation, left-right rotation, and right-left rotation. Each rotation is O(1), and rebalancing propagates up the path — overall O(log n) for insert/delete. AVLs are more strictly balanced than red-black trees, making lookups faster but insertions/deletions slightly slower.

```python
# Right rotation (left-heavy case)
def rotate_right(y):
    x = y.left
    T2 = x.right
    x.right = y
    y.left = T2
    return x
```

---

**Question**: State the properties of a Red-Black tree.

**Answer**: (1) Every node is red or black. (2) The root is black. (3) Every leaf (null) is black. (4) Red nodes cannot have red children (no two consecutive reds). (5) Every path from a node to its descendant leaves has the same number of black nodes (black-height). These properties guarantee O(log n) for search, insert, and delete. Red-black trees are used in TreeMap, TreeSet, and std::map — they are approximately balanced with fewer rotations than AVL trees.

---

**Question**: Describe the segment tree data structure for range queries and updates.

**Answer**: A segment tree is a binary tree storing aggregate information (sum, min, max) over contiguous array segments. Each leaf represents a single element; each internal node merges its children. Build in O(n), range query in O(log n), point update in O(log n). Lazy propagation extends it to range updates in O(log n). Use for problems requiring dynamic range queries with updates (e.g., range sum with modifications).

```python
class SegmentTree:
    def __init__(self, data):
        n = len(data)
        self.tree = [0] * (2 * n)
        # build leaves
        for i in range(n):
            self.tree[n + i] = data[i]
        # build internal nodes
        for i in range(n - 1, 0, -1):
            self.tree[i] = self.tree[i * 2] + self.tree[i * 2 + 1]
        self.n = n

    def update(self, idx, value):
        i = idx + self.n
        self.tree[i] = value
        while i > 1:
            i //= 2
            self.tree[i] = self.tree[i * 2] + self.tree[i * 2 + 1]
```

---

**Question**: Explain the Fenwick tree (Binary Indexed Tree) and its use cases.

**Answer**: A Fenwick tree supports point updates and prefix sum queries in O(log n) using O(n) space. It uses the least significant set bit (LSB) to navigate between nodes — index i stores the sum of a range ending at i. It requires half the code of a segment tree but supports fewer operations (no arbitrary range queries without prefix difference). Use for frequency counts, inversion count, and running prefix sums.

```python
class Fenwick:
    def __init__(self, n):
        self.n = n
        self.bit = [0] * (n + 1)

    def add(self, idx, delta):
        i = idx + 1
        while i <= self.n:
            self.bit[i] += delta
            i += i & -i

    def sum(self, idx):
        i, s = idx + 1, 0
        while i > 0:
            s += self.bit[i]
            i -= i & -i
        return s

    def range_sum(self, l, r):
        return self.sum(r) - self.sum(l - 1)
```

---

**Question**: Implement a trie (prefix tree) with insert, search, and autocomplete.

**Answer**: A trie is an N-ary tree where each node contains an array of children (size of alphabet) and a boolean flag marking the end of a word. Insert iterates over characters, creating missing nodes — O(k) where k is key length. Search follows the same path — O(k). Autocomplete performs a DFS from the prefix node collecting all words. Space is O(total characters) with hash map children (or O(total characters × alphabet size) with fixed-size arrays).

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word):
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.is_end = True

    def search(self, word):
        node = self.root
        for ch in word:
            if ch not in node.children:
                return False
            node = node.children[ch]
        return node.is_end
```

---

**Question**: Compare adjacency list vs adjacency matrix for graph representation.

**Answer**: An adjacency list stores each vertex's neighbors in a list, using O(V + E) space — efficient for sparse graphs. An adjacency matrix uses O(V²) space but supports O(1) edge lookup. Use adjacency lists for most problems (they are the default in interviews). Use an adjacency matrix when the graph is dense or when you need constant-time edge existence checks (e.g., Floyd-Warshall).

```python
# Adjacency list: {0: [1, 2], 1: [2], 2: [0]}
graph = {i: [] for i in range(V)}
graph[u].append(v)

# Adjacency matrix: V x V boolean/int array
matrix = [[0] * V for _ in range(V)]
matrix[u][v] = 1
```

---

**Question**: Compare BFS and DFS graph traversals with complexity analysis.

**Answer**: BFS uses a queue, visits nodes level by level, and finds the shortest path in unweighted graphs — O(V + E). DFS uses a stack (or recursion), visits deep first, and is useful for topological sort, cycle detection, and connected components — O(V + E). BFS space is O(width) — worst-case O(V) for a wide graph. DFS space is O(depth) — worst-case O(V) for a skewed graph. Choose BFS for shortest path, DFS for path existence and backtracking.

```python
from collections import deque

def bfs(graph, start):
    visited, queue = set(), deque([start])
    while queue:
        node = queue.popleft()
        if node not in visited:
            visited.add(node)
            queue.extend(n for n in graph[node] if n not in visited)
    return visited

def dfs(graph, start, visited=None):
    if visited is None:
        visited = set()
    visited.add(start)
    for neighbor in graph[start]:
        if neighbor not in visited:
            dfs(graph, neighbor, visited)
    return visited
```

---

**Question**: Explain topological sort using Kahn's algorithm and DFS-based approaches.

**Answer**: Kahn's algorithm uses in-degree counts: repeatedly remove vertices with in-degree 0, adding them to the result, and decrement in-degrees of their neighbors. DFS-based topological sort performs a post-order traversal — add a node to the result after visiting all its descendants, then reverse. Kahn's algorithm also detects cycles (if not all vertices are processed). Both run in O(V + E). Use for dependency resolution, build systems, and course scheduling.

```python
from collections import deque

def kahn_topo(graph, n):
    in_degree = [0] * n
    for u in graph:
        for v in graph[u]:
            in_degree[v] += 1
    q = deque([i for i in range(n) if in_degree[i] == 0])
    result = []
    while q:
        u = q.popleft()
        result.append(u)
        for v in graph[u]:
            in_degree[v] -= 1
            if in_degree[v] == 0:
                q.append(v)
    return result if len(result) == n else []  # empty if cycle
```

---

**Question**: Describe Dijkstra's shortest path algorithm.

**Answer**: Dijkstra's algorithm finds the shortest path from a source to all nodes in a weighted graph with non-negative edges. It uses a min-heap (priority queue) to repeatedly extract the closest unvisited node, then relaxes its neighbors. Time O((V + E) log V) with a binary heap. It fails with negative weights (use Bellman-Ford). It is essentially BFS with a priority queue replacing the regular queue.

```python
import heapq

def dijkstra(graph, start):
    dist = {node: float('inf') for node in graph}
    dist[start] = 0
    pq = [(0, start)]
    while pq:
        d, u = heapq.heappop(pq)
        if d > dist[u]:
            continue
        for v, w in graph[u]:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                heapq.heappush(pq, (dist[v], v))
    return dist
```

---

**Question**: Explain Union-Find (Disjoint Set Union) with path compression and union by rank.

**Answer**: DSU maintains disjoint sets with two operations: find (returns the representative) and union (merges two sets). Path compression flattens the tree during find, making subsequent finds nearly O(1). Union by rank attaches the smaller tree under the larger tree, keeping depth ≤ log n. Together they achieve near-constant amortized O(α(n)) per operation (inverse Ackermann). Use for connected components in graphs, Kruskal's MST, and dynamic connectivity.

```python
class DSU:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n

    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])  # path compression
        return self.parent[x]

    def union(self, x, y):
        px, py = self.find(x), self.find(y)
        if px == py:
            return False
        if self.rank[px] < self.rank[py]:
            self.parent[px] = py
        elif self.rank[px] > self.rank[py]:
            self.parent[py] = px
        else:
            self.parent[py] = px
            self.rank[px] += 1
        return True
```

---

**Question**: Describe Bloom filters — their use cases and limitations.

**Answer**: A Bloom filter is a probabilistic data structure using k hash functions and a bit array. Insert sets bits at k positions; query checks if all k bits are set. False positives are possible (item reported present when absent), but false negatives are impossible. The false positive rate depends on array size, number of hash functions, and number of inserted items. Use for cache filtering, spell checkers, and avoiding expensive lookups in databases. Bloom filters cannot delete items (unless using counting variants) and cannot store associated data.

```python
import hashlib
import math

class BloomFilter:
    def __init__(self, n, p):
        self.size = int(-n * math.log(p) / (math.log(2) ** 2))
        self.hash_count = int(self.size / n * math.log(2))
        self.bit_array = [False] * self.size

    def add(self, item):
        for i in range(self.hash_count):
            digest = int(hashlib.sha256(f"{item}{i}".encode()).hexdigest(), 16)
            self.bit_array[digest % self.size] = True

    def might_contain(self, item):
        for i in range(self.hash_count):
            digest = int(hashlib.sha256(f"{item}{i}".encode()).hexdigest(), 16)
            if not self.bit_array[digest % self.size]:
                return False
        return True
```

---

**Question**: Explain rolling hash and its application in Rabin-Karp.

**Answer**: A rolling hash computes hash values of substrings in O(1) using a sliding window: hash(s[i+1..j+1]) = (hash(s[i..j]) - s[i] * base^(len-1)) * base + s[j+1], modulo a large prime. Rabin-Karp uses this to find pattern occurrences by comparing hash values instead of characters directly. Average-case O(n + m) for pattern matching, worst-case O(nm) with hash collisions. Use rolling hash for plagiarism detection, longest common substring, and near-duplicate detection.

```python
def rabin_karp(text, pattern):
    base, mod = 256, 10**9 + 7
    n, m = len(text), len(pattern)
    if m > n: return []
    p_hash = sum(ord(pattern[i]) * pow(base, m - 1 - i, mod) for i in range(m)) % mod
    t_hash = sum(ord(text[i]) * pow(base, m - 1 - i, mod) for i in range(m)) % mod
    result = []
    for i in range(n - m + 1):
        if t_hash == p_hash and text[i:i+m] == pattern:
            result.append(i)
        if i < n - m:
            t_hash = ((t_hash - ord(text[i]) * pow(base, m - 1, mod)) * base + ord(text[i + m])) % mod
    return result
```

---

