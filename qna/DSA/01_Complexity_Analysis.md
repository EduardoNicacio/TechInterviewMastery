# Data Structures & Algorithms – Complexity Analysis

**Question**: Explain Big O, Big Theta, and Big Omega notations.

**Answer**: Big O (O) describes the upper bound — worst-case growth rate. Big Omega (Ω) describes the lower bound on the growth rate of a function. Big Theta (Θ) describes the tight bound when the upper and lower bounds match, giving an exact asymptotic characterization. In practice, Big O is most commonly used in interviews to communicate worst-case complexity.

---

**Question**: How do you analyze time vs space complexity tradeoffs?

**Answer**: Time-space tradeoffs arise when you can reduce execution time by using extra memory (e.g., caching, hash maps) or reduce memory usage at the cost of recomputation. A classic example is memoization in DP: storing subproblem results (space) avoids redundant recursive calls (time). Always clarify with the interviewer which resource is more constrained before optimizing.

```java
// Time O(n), Space O(n) — using a HashMap to cache results
Map<Integer, Integer> cache = new HashMap<>();
int fib(int n) {
    if (n <= 1) return n;
    if (cache.containsKey(n)) return cache.get(n);
    int result = fib(n - 1) + fib(n - 2);
    cache.put(n, result);
    return result;
}
```

---

**Question**: What is amortized analysis and how does it apply to dynamic array resizing?

**Answer**: Amortized analysis averages the cost of expensive operations (like resizing) over a sequence of cheaper operations. For a dynamic array (e.g., ArrayList, Python list), doubling the capacity when full costs O(n) copying, but this happens only after n/2 inserts, giving an amortized O(1) per insertion. The key is geometric growth (doubling) rather than arithmetic growth.

```java
// Amortized O(1) insert — resizing doubles capacity
void add(int value) {
    if (size == arr.length) {
        int[] newArr = new int[arr.length * 2];
        System.arraycopy(arr, 0, newArr, 0, arr.length);
        arr = newArr;
    }
    arr[size++] = value;
}
```

---

**Question**: State the Master Theorem and when it applies to recurrence relations.

**Answer**: For recurrences of the form T(n) = aT(n/b) + f(n) where a ≥ 1, b > 1, compare f(n) with n^(log_b a). If f(n) = O(n^(log_b a - ε)), then T(n) = Θ(n^(log_b a)). If f(n) = Θ(n^(log_b a)), then T(n) = Θ(n^(log_b a) log n). If f(n) = Ω(n^(log_b a + ε)) and af(n/b) ≤ cf(n) for some c < 1, then T(n) = Θ(f(n)). It does not apply when the recurrence is not of this form (e.g., divide step varies, or f(n) is not polynomial).

```python
# T(n) = 2T(n/2) + O(n)  →  case 2: Θ(n log n)  (merge sort)
# T(n) = T(n/2) + O(1)    →  case 2: Θ(log n)   (binary search)
```

---

**Question**: Differentiate best-case, average-case, and worst-case analysis.

**Answer**: Best-case is the minimum time for the most favorable input (e.g., sorted array for bubble sort). Worst-case is the maximum time for the least favorable input (e.g., reverse-sorted array). Average-case is the expected time over all possible inputs of a given size, requiring knowledge of the input distribution. Worst-case is most commonly used for guarantees; average-case is used for randomized algorithms like quicksort.

---

**Question**: List the common complexity classes from fastest to slowest.

**Answer**: O(1) — constant, O(log n) — logarithmic (binary search), O(n) — linear (array traversal), O(n log n) — linearithmic (merge sort), O(n²) — quadratic (nested loops), O(2^n) — exponential (subset generation), O(n!) — factorial (permutations). The transition from polynomial to exponential is the most critical threshold in algorithm design.

```python
# O(1)   — arr[0]
# O(log n) — binary search
# O(n)   — linear scan
# O(n log n) — divide and conquer sorts
# O(n^2) — bubble sort
# O(2^n) — all subsets of n items
# O(n!) — all permutations of n items
```

---

**Question**: How do you analyze the complexity of a recursive function?

**Answer**: Write a recurrence relation T(n) describing the recursive calls, then solve it using the Master Theorem, substitution method, or recursion-tree method. For example, a recursive function that makes 2 calls of size n-1 gives T(n) = 2T(n-1) + O(1) → O(2^n). The number of recursive calls and reduction in problem size per level are the key factors.

```python
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)
# T(n) = T(n-1) + T(n-2) + O(1) → O(2^n)
```

---

**Question**: Compare space complexity of recursive vs iterative solutions.

**Answer**: Recursive solutions use O(depth) stack space for call frames, which can be O(n) for linear recursion or O(log n) for balanced divide-and-conquer. Iterative solutions typically use O(1) or O(n) explicit heap space but avoid stack overflow risks. The tradeoff is readability vs memory safety — rewrite recursion to iteration when depth could exceed the stack limit (typically ~1000 frames).

```python
# Recursive DFS — O(h) stack space (h = tree height)
def dfs(node):
    if not node: return
    dfs(node.left)
    dfs(node.right)

# Iterative DFS — O(h) heap space (explicit stack)
def dfs(root):
    if root is None: return
    stack = [root]
    while stack:
        node = stack.pop()
        if node.right: stack.append(node.right)
        if node.left: stack.append(node.left)
```

---

**Question**: How do you identify and optimize performance bottlenecks in an algorithm?

**Answer**: Profile to find the hottest code paths — the operations with the highest asymptotic cost or most frequent execution. Common bottlenecks include nested loops (reduce to single loop), repeated linear searches (replace with hash map), and excessive allocations (reuse buffers). Apply the 80/20 rule: optimize the 20% of code responsible for 80% of runtime, and always measure before and after.

```python
# Bottleneck: O(n²) linear search in a loop
for item in large_list:
    if item in another_list:  # O(n) per check → O(n²)
        ...

# Optimized: O(n) with a hash set
lookup = set(another_list)
for item in large_list:
    if item in lookup:  # O(1) per check → O(n)
        ...
```

---

**Question**: Explain NP-complete, NP-hard, and NP — when should you use approximation?

**Answer**: NP contains problems verifiable in polynomial time. NP-complete problems (e.g., SAT, 3-SAT, the decision version of TSP) are the hardest in NP — if any one is solvable in P, then P = NP. NP-hard problems are at least as hard as NP-complete but need not be in NP (e.g., the optimization version of TSP). Use approximation algorithms, heuristics (genetic, simulated annealing), or pseudo-polynomial DP (e.g., knapsack) when exact solutions are exponential. For interviews, recognize NP-complete problems and pivot to greedy or approximate approaches.

```python
# TSP optimization: NP-hard — use nearest-neighbor heuristic O(n²)
def tsp_nearest_neighbor(graph, start):
    path = [start]
    unvisited = set(graph.keys()) - {start}
    current = start
    while unvisited:
        next_city = min(unvisited, key=lambda c: graph[current][c])
        path.append(next_city)
        unvisited.remove(next_city)
        current = next_city
    return path
```

---

