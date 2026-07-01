# Data Structures & Algorithms – Algorithms

**Question**: Explain quicksort — partitioning schemes, worst-case scenario, and optimizations.

**Answer**: Quicksort picks a pivot, partitions the array so elements < pivot come before and > pivot come after, then recursively sorts each partition. Lomuto partition (pivot = last element) is simpler but degrades to O(n²) on sorted arrays; Hoare partition (two pointers from ends) is more efficient in practice. Worst-case O(n²) occurs when the pivot is always the smallest or largest element (sorted or reverse-sorted). Optimizations: random pivot selection, median-of-three, switching to insertion sort for small subarrays (< 16 elements), and tail-call recursion optimization.

```python
import random

def quicksort(arr, lo, hi):
    while lo < hi:
        pivot = partition(arr, lo, hi)
        quicksort(arr, lo, pivot - 1)
        lo = pivot + 1  # tail-call optimization

def partition(arr, lo, hi):
    # random pivot to avoid worst-case
    r = random.randint(lo, hi)
    arr[r], arr[hi] = arr[hi], arr[r]
    pivot = arr[hi]
    i = lo - 1
    for j in range(lo, hi):
        if arr[j] <= pivot:
            i += 1
            arr[i], arr[j] = arr[j], arr[i]
    arr[i + 1], arr[hi] = arr[hi], arr[i + 1]
    return i + 1
```

---

**Question**: Explain merge sort — why it is stable and how divide and conquer applies.

**Answer**: Merge sort recursively divides the array into halves until single elements, then merges them back in sorted order. It is stable because when merging, if two elements are equal, the one from the left subarray is placed first (preserving original order). The divide and conquer strategy guarantees O(n log n) worst-case time, but requires O(n) auxiliary space. Merge sort is preferred for linked lists (O(1) extra space) and external sorting where sequential access dominates.

```python
def merge_sort(arr):
    if len(arr) <= 1:
        return arr
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    return merge(left, right)

def merge(left, right):
    result = []
    i = j = 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:  # <= ensures stability
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
    return result + left[i:] + right[j:]
```

---

**Question**: Describe heap sort and why it is an in-place sorting algorithm.

**Answer**: Heap sort first builds a max-heap (O(n)), then repeatedly extracts the maximum by swapping the root with the last element and sifting down. It is in-place because it sorts within the original array using O(1) extra space (excluding recursion/iteration overhead). It is not stable due to the swap operation. Heap sort is O(n log n) worst-case but slower in practice than quicksort due to poor cache locality.

```python
def heapify(arr, n, i):
    largest = i
    l, r = 2 * i + 1, 2 * i + 2
    if l < n and arr[l] > arr[largest]:
        largest = l
    if r < n and arr[r] > arr[largest]:
        largest = r
    if largest != i:
        arr[i], arr[largest] = arr[largest], arr[i]
        heapify(arr, n, largest)

def heap_sort(arr):
    n = len(arr)
    for i in range(n // 2 - 1, -1, -1):
        heapify(arr, n, i)
    for i in range(n - 1, 0, -1):
        arr[0], arr[i] = arr[i], arr[0]
        heapify(arr, i, 0)
```

---

**Question**: Describe binary search variants — rotated array, first/last occurrence.

**Answer**: Standard binary search finds an element in O(log n) on a sorted array. For a rotated sorted array, compare mid with left to determine which half is sorted. For first/last occurrence, once the target is found, continue searching left for first and right for last using the same pattern (don't return immediately). These variants change the termination condition and which half to explore next while maintaining O(log n).

```python
# Rotated array search
def search_rotated(nums, target):
    lo, hi = 0, len(nums) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if nums[mid] == target:
            return mid
        if nums[lo] <= nums[mid]:  # left sorted
            if nums[lo] <= target < nums[mid]:
                hi = mid - 1
            else:
                lo = mid + 1
        else:  # right sorted
            if nums[mid] < target <= nums[hi]:
                lo = mid + 1
            else:
                hi = mid - 1
    return -1
```

---

**Question**: How do counting sort and radix sort work, and when should you use them?

**Answer**: Counting sort counts frequencies of each key value within a known integer range, then computes prefix sums to place elements in sorted order — O(n + k) where k is the range. Radix sort sorts digit by digit using a stable counting sort per digit — O(d × (n + k)) where d is digit count. Both are non-comparison sorts that beat O(n log n) lower bound. Use counting sort when k = O(n) (e.g., sorting ages 0-150). Use radix sort for fixed-length integer or string keys.

```python
def counting_sort(arr, k):
    count = [0] * (k + 1)
    for v in arr:
        count[v] += 1
    # prefix sums → positions
    for i in range(1, k + 1):
        count[i] += count[i - 1]
    result = [0] * len(arr)
    for v in reversed(arr):
        count[v] -= 1
        result[count[v]] = v
    return result
```

---

**Question**: Compare memoization (top-down) vs tabulation (bottom-up) in dynamic programming.

**Answer**: Memoization caches recursive call results in a hash map or array, computing only needed subproblems but incurring recursion overhead and potential stack overflow. Tabulation fills a DP table iteratively from the base case up, avoiding recursion entirely. Both use O(DP state space) time and space. Tabulation is usually faster (no recursion/cache-lookup overhead) and preferred when the dependency order is straightforward; memoization is easier to derive from a recursive solution.

```python
# Fibonacci — memoization (top-down)
def fib_memo(n, memo=None):
    if memo is None:
        memo = {}
    if n in memo:
        return memo[n]
    if n <= 1:
        return n
    memo[n] = fib_memo(n - 1, memo) + fib_memo(n - 2, memo)
    return memo[n]

# Fibonacci — tabulation (bottom-up)
def fib_tab(n):
    if n <= 1:
        return n
    dp = [0] * (n + 1)
    dp[1] = 1
    for i in range(2, n + 1):
        dp[i] = dp[i - 1] + dp[i - 2]
    return dp[n]
```

---

**Question**: Solve the 0/1 knapsack problem using dynamic programming.

**Answer**: Define dp[i][w] = max value using first i items with capacity w. Recurrence: dp[i][w] = max(dp[i-1][w], values[i-1] + dp[i-1][w - weights[i-1]]) if weights[i-1] ≤ w. Base case: dp[0][*] = 0. Time O(n × capacity), space O(capacity) with 1D array (iterate capacity backwards to avoid reuse). The 0/1 constraint means each item is either taken in full or not at all.

```python
def knapsack_01(weights, values, capacity):
    n = len(weights)
    dp = [0] * (capacity + 1)
    for i in range(n):
        for w in range(capacity, weights[i] - 1, -1):
            dp[w] = max(dp[w], values[i] + dp[w - weights[i]])
    return dp[capacity]
```

---

**Question**: Explain Longest Common Subsequence (LCS) and Longest Increasing Subsequence (LIS).

**Answer**: LCS uses DP where dp[i][j] = length of LCS of text1[:i] and text2[:j]. If chars match, dp[i][j] = 1 + dp[i-1][j-1]; otherwise dp[i][j] = max(dp[i-1][j], dp[i][j-1]). O(m × n) time and space (optimizable to O(n)). LIS can be solved with O(n²) DP (dp[i] = LIS ending at i) or O(n log n) with patience sorting (binary search on the tails array). Both are classic DP patterns for sequence comparison.

```python
# LCS — O(m × n)
def longest_common_subsequence(text1, text2):
    m, n = len(text1), len(text2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if text1[i - 1] == text2[j - 1]:
                dp[i][j] = 1 + dp[i - 1][j - 1]
            else:
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
    return dp[m][n]

# LIS — O(n log n)
import bisect
def length_of_lis(nums):
    tails = []
    for v in nums:
        i = bisect.bisect_left(tails, v)
        if i == len(tails):
            tails.append(v)
        else:
            tails[i] = v
    return len(tails)
```

---

**Question**: Describe the edit distance (Levenshtein distance) algorithm.

**Answer**: Edit distance computes the minimum number of single-character operations (insert, delete, substitute) to transform one string into another. DP recurrence: if characters match, cost = dp[i-1][j-1]; otherwise cost = 1 + min(delete = dp[i-1][j], insert = dp[i][j-1], substitute = dp[i-1][j-1]). Time O(m × n), space optimizable to O(min(m, n)). Used in spell checkers, DNA sequence alignment, and diff tools.

```python
def edit_distance(s1, s2):
    m, n = len(s1), len(s2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    for i in range(m + 1):
        dp[i][0] = i
    for j in range(n + 1):
        dp[0][j] = j
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if s1[i - 1] == s2[j - 1]:
                dp[i][j] = dp[i - 1][j - 1]
            else:
                dp[i][j] = 1 + min(dp[i - 1][j], dp[i][j - 1], dp[i - 1][j - 1])
    return dp[m][n]
```

---

**Question**: Explain matrix chain multiplication using DP.

**Answer**: Given matrices A1 × A2 × ... × An with dimensions p0×p1, p1×p2, ..., find optimal parenthesization minimizing scalar multiplications. dp[i][j] = minimum cost to multiply matrices i through j. Recurrence: dp[i][j] = min_{k=i}^{j-1} (dp[i][k] + dp[k+1][j] + p[i] × p[k+1] × p[j+1]). Time O(n³), space O(n²). This is a classic DP on intervals pattern — solve smaller intervals first, then combine.

```python
def matrix_chain_order(p):
    n = len(p) - 1
    dp = [[0] * n for _ in range(n)]
    for length in range(2, n + 1):  # interval length
        for i in range(n - length + 1):
            j = i + length - 1
            dp[i][j] = float('inf')
            for k in range(i, j):
                cost = dp[i][k] + dp[k + 1][j] + p[i] * p[k + 1] * p[j + 1]
                dp[i][j] = min(dp[i][j], cost)
    return dp[0][n - 1]
```

---

**Question**: Solve the "buy and sell stock" problems using state machine DP.

**Answer**: Model each day as a state machine with states: hold (have stock), sold (just sold), and cooldown/rest (no stock). Transition: rest → hold (buy), hold → sold (sell), sold → rest (cooldown). For k transactions, use dp[k][i] with states. The pattern generalizes from one transaction (O(n)) to infinite with cooldown (O(n)) to at most k transactions (O(k × n)). State machine DP clarifies the decision flow for multi-phase problems.

```python
# Best time with cooldown — state machine
def max_profit_cooldown(prices):
    rest, hold, sold = 0, float('-inf'), float('-inf')
    for p in prices:
        prev_sold = sold
        rest, hold, sold = (
            max(rest, prev_sold),         # rest: skip or post-cooldown
            max(hold, rest - p),           # hold: keep or buy
            hold + p                       # sold: sell
        )
    return max(rest, sold)
```

---

**Question**: Describe the interval scheduling greedy algorithm.

**Answer**: Sort intervals by end time, then repeatedly select the earliest-finishing interval that does not overlap with the last selected one. This greedy choice (earliest finish time) guarantees the maximum number of non-overlapping intervals. Time O(n log n) due to sorting, space O(1). The same pattern applies to meeting rooms II — use a min-heap of end times to find minimum rooms required.

```python
def max_non_overlapping(intervals):
    intervals.sort(key=lambda x: x[1])
    count, end = 0, float('-inf')
    for s, e in intervals:
        if s >= end:
            count += 1
            end = e
    return count
```

---

**Question**: Explain the quick select algorithm for finding the kth smallest/largest element.

**Answer**: Quick select uses the same partition step as quicksort but recurses only into one side: after partitioning, if the pivot index is k, return it; if k < pivot index, recurse left; otherwise recurse right. Average O(n), worst-case O(n²) (mitigated by random pivot selection). It is the optimal algorithm for finding kth order statistics and is the basis for the C++ std::nth_element and Python's statistics.median_low.

```python
import random

def quick_select(arr, k):
    lo, hi = 0, len(arr) - 1
    while lo <= hi:
        pivot_idx = random.randint(lo, hi)
        arr[pivot_idx], arr[hi] = arr[hi], arr[pivot_idx]
        p = partition(arr, lo, hi)  # Lomuto partition
        if p == k:
            return arr[p]
        elif p < k:
            lo = p + 1
        else:
            hi = p - 1
    return -1

def partition(arr, lo, hi):
    pivot = arr[hi]
    i = lo
    for j in range(lo, hi):
        if arr[j] <= pivot:
            arr[i], arr[j] = arr[j], arr[i]
            i += 1
    arr[i], arr[hi] = arr[hi], arr[i]
    return i
```

---

**Question**: Describe the Bellman-Ford algorithm and how it handles negative weights.

**Answer**: Bellman-Ford finds shortest paths from a source to all vertices, supporting negative edge weights. It relaxes all edges V-1 times — after k iterations, it finds shortest paths using at most k edges. After V-1 iterations, a Vth pass detects negative cycles: if any edge can still be relaxed, a negative cycle exists. Time O(V × E). Use when Dijkstra is inapplicable (negative weights) or to detect negative cycles for currency arbitrage.

```python
def bellman_ford(edges, V, src):
    dist = [float('inf')] * V
    dist[src] = 0
    for _ in range(V - 1):
        for u, v, w in edges:
            if dist[u] != float('inf') and dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
    # Detect negative cycles
    for u, v, w in edges:
        if dist[u] != float('inf') and dist[u] + w < dist[v]:
            return None  # negative cycle exists
    return dist
```

---

**Question**: Explain the Floyd-Warshall algorithm for all-pairs shortest paths.

**Answer**: Floyd-Warshall uses DP with intermediate vertices: dp[i][j] = shortest path from i to j using vertices {0..k} as intermediates. Recurrence: dp[i][j] = min(dp[i][j], dp[i][k] + dp[k][j]). Time O(V³), space O(V²). It handles negative weights (but not negative cycles — detect by checking dp[i][i] < 0). Use for dense graphs, transitive closure, and problems needing shortest paths between all pairs (e.g., finding a central node).

```python
def floyd_warshall(graph):
    V = len(graph)
    dist = [row[:] for row in graph]  # copy adjacency matrix
    for k in range(V):
        for i in range(V):
            for j in range(V):
                if dist[i][k] + dist[k][j] < dist[i][j]:
                    dist[i][j] = dist[i][k] + dist[k][j]
    # Check negative cycles
    for i in range(V):
        if dist[i][i] < 0:
            return None
    return dist
```

---

