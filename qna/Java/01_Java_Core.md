# Java – Interview Questions & Answers

**Question**: Explain the JVM architecture and its main components.

**Answer**: The JVM consists of three main subsystems: the Class Loader (loading, linking, initialization), Runtime Data Areas (Method Area, Heap, Stack, PC Registers, Native Method Stack), and the Execution Engine (Interpreter, JIT Compiler, Garbage Collector). The Class Loader loads `.class` files into the Method Area, the Execution Engine executes bytecode, and the GC manages heap memory.

```java
// Class loading happens lazily when class is first referenced
public class JvmDemo {
    static { System.out.println("Class loaded!"); }
    public static void main(String[] args) {
        // Triggers class loading of JvmDemo
    }
}
```

---

**Question**: How does the Class Loader work in Java?

**Answer**: The Class Loader follows the delegation hierarchy: Bootstrap ClassLoader → Extension ClassLoader → Application/System ClassLoader. Before a class is loaded, the parent ClassLoader is asked first; if it cannot load it, the child attempts. This ensures core Java APIs are never replaced by custom classes.

```java
public class ClassLoaderDemo {
    public static void main(String[] args) {
        ClassLoader cl = ClassLoaderDemo.class.getClassLoader();
        System.out.println(cl); // jdk.internal.loader.ClassLoaders$AppClassLoader
        System.out.println(cl.getParent()); // jdk.internal.loader.ClassLoaders$PlatformClassLoader
        System.out.println(cl.getParent().getParent()); // null (Bootstrap)
    }
}
```

---

**Question**: What are the Runtime Data Areas in JVM?

**Answer**: The Runtime Data Areas include: Method Area (class metadata, static variables, constants), Heap (all object instances and arrays), Stack (frames with local variables, operand stack, frame data per thread), PC Registers (address of current instruction per thread), and Native Method Stack (native method calls). The Heap and Method Area are shared across threads; Stack, PC Registers, and Native Method Stack are thread-private.

---

**Question**: Explain Java Garbage Collection with the Generational approach.

**Answer**: The Heap is divided into Young Generation (Eden + Survivor S0/S1) and Old Generation. New objects start in Eden; after a Minor GC, surviving objects move to S0, then S1 on the next collection, and finally to Old Generation after a tenure threshold. A Major GC (Full GC) collects the Old Generation. This generational design exploits the weak generational hypothesis that most objects die young.

```java
// GC can be tuned via JVM flags:
// -Xms512m -Xmx2g -XX:NewRatio=2 -XX:SurvivorRatio=8
// -XX:+UseG1GC (default since Java 9)
public class GCDemo {
    public static void main(String[] args) {
        for (int i = 0; i < 100_000; i++) {
            new Object(); // short-lived objects collected in Eden
        }
    }
}
```

---

**Question**: How does the G1 Garbage Collector work?

**Answer**: G1 (Garbage-First) divides the heap into equal-sized regions (typically 2048, each 1–32 MB). It performs incremental, concurrent garbage collection with a predictable pause-time goal (`-XX:MaxGCPauseMillis=200`). G1 prioritizes regions with the most garbage (most live data reclaimed per pause) and uses remembered sets (RSets) to track cross-region references without scanning the full heap.

```java
// Enable G1 explicitly: java -XX:+UseG1GC -XX:MaxGCPauseMillis=100 MyApp
```

---

**Question**: What are ZGC and Shenandoah GC?

**Answer**: ZGC (Java 11+, production in Java 15) is a scalable, low-latency concurrent GC with sub-millisecond pause times, regardless of heap size. It uses colored pointers (load barriers) and region-based compaction. Shenandoah (backported to Java 8, production in Java 15) performs evacuation concurrently with the application threads, using Brooks pointers. Both aim to keep pause times under 10ms even for multi-terabyte heaps.

```java
// ZGC: -XX:+UseZGC -Xms16g -Xmx16g
// Shenandoah: -XX:+UseShenandoahGC
```

---

**Question**: Describe the Java Memory Model - Heap, Stack, and Metaspace.

**Answer**: The Heap stores all object instances and arrays; it is shared across threads and managed by the GC. The Stack stores local variables, partial results, and method call frames (one frame per method call); each thread has its own stack. Metaspace (introduced in Java 8, replacing PermGen) stores class metadata; it grows dynamically in native memory and is not subject to GC.

```java
public class MemoryDemo {
    int instanceVar; // Heap
    static int classVar; // Heap (part of java.lang.Class mirror object)
    public void method() {
        int localVar = 42; // Stack
    }
}
```

---

**Question**: What is the String Pool and why are Strings immutable?

**Answer**: The String Pool is a special area in the Heap (interned strings) where String literals are cached. When you create a String literal, the JVM checks the pool first and reuses the reference if found. Strings are immutable for several reasons: security (no modification of sensitive strings like passwords), thread safety (no synchronization needed), caching (hashCode can be cached), and performance (String Pool reuse).

```java
public class StringPoolDemo {
    public static void main(String[] args) {
        String s1 = "hello";
        String s2 = "hello";
        String s3 = new String("hello");
        System.out.println(s1 == s2);    // true (same pool reference)
        System.out.println(s1 == s3);    // false (heap vs pool)
        System.out.println(s1.equals(s3)); // true (same value)
        s3 = s3.intern();                // add to pool if not present
        System.out.println(s1 == s3);    // true
    }
}
```

---

**Question**: Explain the `equals()` and `hashCode()` contract.

**Answer**: The contract states: if two objects are equal according to `equals()`, they must have the same `hashCode()`. The reverse is not required (different objects can have the same hash - hash collision). Breaking this contract means hash-based collections (HashMap, HashSet) will not function correctly - objects that are equal may end up in different buckets, causing lookups to fail.

```java
public class User {
    private String email;
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof User)) return false;
        return Objects.equals(email, ((User) o).email);
    }
    @Override
    public int hashCode() {
        return Objects.hashCode(email); // consistent with equals, null-safe
    }
}
```

---

**Question**: What is the difference between `==` and `equals()` in Java?

**Answer**: `==` compares reference equality - it checks whether two references point to the same memory location. `equals()` compares logical/structural equality (object content) and can be overridden by user classes. For primitives, `==` compares values. For String comparison, always use `equals()` unless you explicitly want to check reference identity.

```java
String a = new String("hello");
String b = new String("hello");
System.out.println(a == b);      // false (different objects)
System.out.println(a.equals(b)); // true (same value)
```

---

**Question**: Explain `final`, `finally`, and `finalize()` in Java.

**Answer**: `final` is a keyword used for classes (cannot be subclassed), methods (cannot be overridden), and variables (cannot be reassigned). `finally` is a block used with try-catch that always executes (except if JVM exits abruptly), typically for cleanup. `finalize()` is a method called by GC before reclaiming an object's memory - it is deprecated since Java 9 and should never be relied upon for resource cleanup.

```java
public class FinalDemo {
    private final int id = 1; // cannot be reassigned
    public final void display() {} // cannot be overridden

    public void cleanUp() {
        try {
            // resource usage
        } finally {
            // always executes (cleanup)
        }
    }

    @Override @Deprecated(since = "9", forRemoval = true)
    protected void finalize() throws Throwable {
        // Avoid - unpredictable and unreliable
    }
}
```

---

**Question**: Differentiate between Checked and Unchecked Exceptions.

**Answer**: Checked exceptions (subclasses of `Exception` but not `RuntimeException`) must be declared in the method signature using `throws` or caught with try-catch; the compiler enforces this. Unchecked exceptions (`RuntimeException` and `Error`) do not require explicit handling. Checked exceptions represent recoverable conditions (e.g., `IOException`), while unchecked represent programming bugs (e.g., `NullPointerException`).

```java
// Checked - must handle or declare
public void readFile() throws IOException {
    throw new IOException("File not found");
}

// Unchecked - no declaration needed
public void divide() {
    throw new ArithmeticException("/ by zero");
}
```

---

---

**Question**: How do Generics work in Java and what is type erasure?

**Answer**: Generics enable type-safe collections by parameterizing types. Due to type erasure, generic type information is removed at runtime - the compiler replaces type parameters with their leftmost bound (or `Object` if unbounded). This means `List<String>` and `List<Integer>` are the same `List` at runtime. Type erasure ensures backward compatibility with pre-generics code but prevents runtime type queries like `new T()` or `instanceof T`.

```java
public class GenericDemo<T> {
    private T value;
    // After erasure: private Object value; (T is erased to Object)

    public void setValue(T value) { this.value = value; }
    // After erasure: public void setValue(Object value)

    // Cannot do: if (value instanceof T) {}
    // Cannot do: T copy = new T();
}
```

---

**Question**: Explain wildcards and bounds in Java Generics.

**Answer**: Wildcards (`?`) represent unknown types. Upper-bounded wildcards (`? extends T`) allow reading items of type T (producer - covariance). Lower-bounded wildcards (`? super T`) allow writing items of type T (consumer - contravariance). Unbounded wildcards (`?`) work with any type. The PECS rule (Producer Extends, Consumer Super) guides correct usage.

```java
public class WildcardDemo {
    // Producer - read items as Number
    public double sum(List<? extends Number> numbers) {
        // numbers.add(42); // Compile error - cannot add
        return numbers.stream().mapToDouble(Number::doubleValue).sum();
    }

    // Consumer - add Integer items
    public void addNumbers(List<? super Integer> list) {
        list.add(1);  // OK
        list.add(2);  // OK
        // Integer i = list.get(0); // Compile error - ? super Integer
        Object o = list.get(0); // OK
    }
}
```

---

**Question**: What is covariance and contravariance in Java generics?

**Answer**: Covariance (`? extends T`) preserves the subtyping relationship: `List<Integer>` is a subtype of `List<? extends Number>`. You can read from it safely but cannot write (except null). Contravariance (`? super T`) reverses the relationship: `List<Number>` is a subtype of `List<? super Integer>`. You can write to it but reading is unsafe (only `Object`). Java arrays are covariant (reified), generics are invariant (due to erasure).

```java
// Array covariance (runtime check)
Number[] nums = new Integer[10]; // compiles, but:
nums[0] = 3.14; // ArrayStoreException at runtime

// Generic invariance (compile-time safety)
// List<Number> list = new ArrayList<Integer>(); // Compile error
List<? extends Number> list = new ArrayList<Integer>(); // OK - covariance
// list.add(1); // Compile error - can't add
Number n = list.get(0); // OK - can read
```

---

**Question**: Explain the internals of HashMap, ConcurrentHashMap, and TreeMap.

**Answer**: `HashMap` uses an array of Node buckets; it stores keys in buckets based on `hashCode()` and resolves collisions with linked lists (or red-black trees when threshold > 8). `ConcurrentHashMap` uses fine-grained locking (or CAS + synchronized on specific bins since Java 8) for thread-safe concurrent access without locking the entire map. `TreeMap` is a Red-Black tree implementation that stores keys sorted by their natural order or a custom `Comparator` - all operations are O(log n).

```java
Map<String, Integer> hashMap = new HashMap<>();
hashMap.put("key", 1); // O(1) avg, O(log n) worst with tree

Map<String, Integer> concurrent = new ConcurrentHashMap<>();
concurrent.put("key", 1); // thread-safe, no full map lock

Map<String, Integer> tree = new TreeMap<>();
tree.put("b", 2); tree.put("a", 1); // sorted: {a=1, b=2}
```

---

**Question**: How does HashMap handle collisions?

**Answer**: When two keys produce the same bucket index (hash collision), HashMap stores them in a linked list at that bucket (separate chaining). Starting from Java 8, if the chain length exceeds TREEIFY_THRESHOLD (8) and the array length is at least 64, the list converts to a red-black tree to improve worst-case lookup from O(n) to O(log n). If the tree shrinks below UNTREEIFY_THRESHOLD (6), it reverts to a linked list.

```java
// Internal Node structure (simplified)
// static class Node<K,V> {
//     final int hash;
//     final K key;
//     V value;
//     Node<K,V> next; // linked list for collisions
// }
```

---

**Question**: Compare Comparable vs Comparator in Java.

**Answer**: `Comparable` defines a natural ordering within the class itself via `compareTo()` - it's used for String, Integer, etc. `Comparator` is a separate strategy object that defines custom ordering without modifying the class. Use `Comparable` when there's a single natural order; use `Comparator` for multiple or external sorting logic.

```java
public class Employee implements Comparable<Employee> {
    int id;
    String name;
    public int id() { return id; }
    public int compareTo(Employee o) { return Integer.compare(this.id, o.id); }
}

// Comparator for custom ordering
Comparator<Employee> byName = Comparator.comparing(e -> e.name);
Comparator<Employee> byIdDesc = Comparator.comparingInt(Employee::id).reversed();

List<Employee> list = new ArrayList<>();
list.sort(byName); // using Comparator
```

---

**Question**: Explain the Streams API - intermediate and terminal operations.

**Answer**: The Streams API (Java 8+) enables functional-style operations on collections. Intermediate operations (`filter`, `map`, `flatMap`, `sorted`, `distinct`, `limit`) return a new stream and are lazy - they don't execute until a terminal operation is invoked. Terminal operations (`collect`, `forEach`, `reduce`, `count`, `anyMatch`, `findFirst`) produce a result or side effect and close the stream. A stream cannot be reused after a terminal operation.

```java
List<String> names = List.of("Alice", "Bob", "Charlie", "David");

List<String> result = names.stream()
    .filter(name -> name.length() > 3)   // intermediate (lazy)
    .map(String::toUpperCase)            // intermediate (lazy)
    .sorted()                            // intermediate (lazy)
    .collect(Collectors.toList());       // terminal (eager)

// Parallel stream for multi-threading
long count = names.parallelStream()
    .filter(name -> name.startsWith("A"))
    .count();
```

---

**Question**: What are lambda expressions and functional interfaces?

**Answer**: A lambda expression is a concise anonymous function - `(parameters) -> expression` or `(parameters) -> { statements; }`. It can only be used with functional interfaces (interfaces with a single abstract method, annotated with `@FunctionalInterface`). Common built-in functional interfaces include `Function<T,R>`, `Predicate<T>`, `Consumer<T>`, `Supplier<T>`, `UnaryOperator<T>`, and `BinaryOperator<T>`.

```java
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}

// Lambda usage
Calculator add = (a, b) -> a + b;
Calculator multiply = (a, b) -> a * b;

// Built-in functional interfaces
Function<String, Integer> parser = Integer::parseInt;
Predicate<String> isEmpty = String::isEmpty;
Consumer<String> printer = System.out::println;
Supplier<Double> random = Math::random;
```

---

**Question**: Explain method references in Java.

**Answer**: Method references are shorthand for lambda expressions that call an existing method. They use the `::` operator and come in four kinds: static method reference (`Class::staticMethod`), instance method of a specific object (`instance::method`), instance method of an arbitrary object of a given type (`Class::instanceMethod`), and constructor reference (`Class::new`). They make code more concise and readable.

```java
List<String> names = List.of("alice", "bob", "charlie");

// Static method reference
Function<String, Integer> parser = Integer::parseInt;

// Instance method of a specific object
names.forEach(System.out::println);

// Instance method of an arbitrary object
names.stream().map(String::toUpperCase).collect(Collectors.toList());

// Constructor reference
Supplier<List<String>> listSupplier = ArrayList::new;

// Specific instance method reference
String prefix = "Hello: ";
names.stream().map(prefix::concat).forEach(System.out::println);
```

---

**Question**: What is the `Optional` class and when should it be used?

**Answer**: `Optional<T>` is a container object that may or may not contain a non-null value, introduced in Java 8 to reduce `NullPointerException`. It provides methods like `isPresent()`, `ifPresent()`, `orElse()`, `orElseGet()`, `orElseThrow()`, `map()`, and `flatMap()` for functional-style handling of absent values. Use it as a return type for methods that might not have a result - never use it for fields, constructor parameters, or collections.

```java
public Optional<User> findUser(String id) {
    User user = database.lookup(id);
    return Optional.ofNullable(user);
}

// Consumer usage
findUser("123").ifPresent(u -> System.out.println(u.getName()));

// Defensive usage with defaults
User user = findUser("456")
    .orElseThrow(() -> new UserNotFoundException("User not found"));

String name = findUser("789")
    .map(User::getName)
    .orElse("Guest");
```

---

**Question**: How does `CompletableFuture` enable asynchronous programming?

**Answer**: `CompletableFuture` (Java 8+) implements `Future` and `CompletionStage`, allowing chaining of asynchronous tasks without blocking. It supports methods like `supplyAsync()`, `thenApply()`, `thenCompose()`, `thenCombine()`, `allOf()`, and `anyOf()` for composing async pipelines, with fine-grained control over the executor. It also provides `complete()`, `completeExceptionally()`, and `join()` for completion handling.

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    return fetchFromRemoteService();
}).thenApply(result -> result.toUpperCase())
  .exceptionally(ex -> "Fallback value");

// Combine multiple futures
CompletableFuture<String> f1 = CompletableFuture.supplyAsync(() -> "Hello");
CompletableFuture<String> f2 = CompletableFuture.supplyAsync(() -> "World");
String combined = f1.thenCombine(f2, (a, b) -> a + " " + b).join();

// Wait for all
CompletableFuture.allOf(f1, f2).join();
```

---

**Question**: Describe the thread lifecycle and its states in Java.

**Answer**: A thread goes through the following states (defined in `Thread.State` enum): `NEW` (created but not started), `RUNNABLE` (executing or ready to execute), `BLOCKED` (waiting for a monitor lock), `WAITING` (waiting indefinitely for another thread to perform an action via `wait()`, `join()`, `park()`), `TIMED_WAITING` (waiting with a timeout), and `TERMINATED` (completed execution). Transitions occur via `start()`, `sleep()`, `wait()`, `notify()`, lock acquisition, and `join()`.

```java
final Object lock = new Object();
Thread thread = new Thread(() -> {
    try {
        Thread.sleep(1000); // TIMED_WAITING
        synchronized (lock) { // BLOCKED if lock not available
            lock.wait(); // WAITING
        }
    } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
});
System.out.println(thread.getState()); // NEW
thread.start(); // RUNNABLE
```

---

**Question**: How does the `synchronized` keyword work with intrinsic locks?

**Answer**: Every Java object has an intrinsic lock (monitor). The `synchronized` keyword acquires this lock before entering the block/method and releases it upon exit (even if exceptions occur). Synchronized instance methods lock on `this`; synchronized static methods lock on the `Class` object. `synchronized` provides mutual exclusion and visibility (happens-before guarantee) - all memory operations before the unlock are visible to subsequent lock acquisitions.

```java
public class Counter {
    private int count = 0;

    // Instance method lock - synchronized on 'this'
    public synchronized void increment() {
        count++;
    }

    // Block synchronization - finer granularity
    public void decrement() {
        synchronized (this) {
            count--;
        }
    }

    // Static method lock - synchronized on Counter.class
    public static synchronized void staticMethod() {}
}
```

---

**Question**: What does the `volatile` keyword guarantee?

**Answer**: `volatile` ensures visibility and ordering - writes to a `volatile` variable are immediately visible to all threads (no thread-local caching), and reads/writes to it establish happens-before relationships. Unlike `synchronized`, it does not provide mutual exclusion - compound operations (like `count++`) remain non-atomic. Use `volatile` for flags, status variables, or cases where only visibility is needed, not atomicity.

```java
public class VolatileDemo {
    private volatile boolean running = true;

    public void stop() { running = false; } // visible to other threads immediately

    public void run() {
        while (running) {
            // guaranteed to see updated value of running
        }
    }
}
```

---

**Question**: Explain `wait()`, `notify()`, and `notifyAll()`.

**Answer**: These are methods on `Object` used for inter-thread coordination within a synchronized block. `wait()` releases the monitor and puts the thread in WAITING state until another thread calls `notify()` or `notifyAll()` on the same object. `notify()` wakes one waiting thread (chosen arbitrarily); `notifyAll()` wakes all waiting threads. Always call these in a loop checking for the condition (spurious wakeup protection).

```java
public class WaitNotifyDemo {
    private final Object lock = new Object();
    private boolean ready = false;

    public void produce() throws InterruptedException {
        synchronized (lock) {
            // produce data
            ready = true;
            lock.notify(); // wake consumer
        }
    }

    public void consume() throws InterruptedException {
        synchronized (lock) {
            while (!ready) { // loop prevents spurious wakeup
                lock.wait();
            }
            // consume data
        }
    }
}
```

---

**Question**: How does `ExecutorService` and `ThreadPoolExecutor` work?

**Answer**: `ExecutorService` manages a pool of worker threads, decoupling task submission from execution. `ThreadPoolExecutor` is the configurable implementation with core pool size, maximum pool size, keep-alive time, work queue, and rejection policy. When a task is submitted, it's handled as: if fewer than core threads are running, a new thread is created; otherwise, the task is queued; if the queue is full, additional threads are created up to max; if max is reached, the rejection policy applies.

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    2,                    // corePoolSize
    4,                    // maximumPoolSize
    60, TimeUnit.SECONDS, // keep-alive
    new LinkedBlockingQueue<>(100), // work queue
    new ThreadPoolExecutor.CallerRunsPolicy() // rejection policy
);

ExecutorService fixedPool = Executors.newFixedThreadPool(4);
ExecutorService cachedPool = Executors.newCachedThreadPool();
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);

executor.submit(() -> System.out.println("Task executed"));
executor.shutdown();
```

---

**Question**: Explain the Fork/Join Framework in Java.

**Answer**: The Fork/Join Framework (Java 7+) is designed for divide-and-conquer parallelism. It splits a task into smaller subtasks (fork) recursively until the subtask is small enough to process directly, then combines results (join). It uses a work-stealing algorithm where idle threads steal work from busy threads' queues. `ForkJoinPool` is its executor, and `RecursiveTask<T>` (with result) or `RecursiveAction` (no result) are the base classes.

```java
public class SumTask extends RecursiveTask<Long> {
    private static final int THRESHOLD = 10_000;
    private final long[] array;
    private final int start, end;

    public SumTask(long[] array, int start, int end) {
        this.array = array; this.start = start; this.end = end;
    }

    @Override
    protected Long compute() {
        if (end - start <= THRESHOLD) {
            long sum = 0;
            for (int i = start; i < end; i++) sum += array[i];
            return sum;
        }
        int mid = (start + end) / 2;
        SumTask left = new SumTask(array, start, mid);
        SumTask right = new SumTask(array, mid, end);
        left.fork();
        return right.compute() + left.join();
    }
}

// Usage
long[] data = new long[1_000_000];
long sum = ForkJoinPool.commonPool().invoke(new SumTask(data, 0, data.length));
```

---

**Question**: Describe the Lock API - `ReentrantLock` and `ReadWriteLock`.

**Answer**: The `java.util.concurrent.locks` package provides more flexible locking than `synchronized`. `ReentrantLock` offers try-lock (`tryLock()`), timed lock, fairness policy, and `lockInterruptibly()`. `ReadWriteLock` maintains a pair of locks - multiple threads can read simultaneously (read lock) but write access is exclusive (write lock), improving concurrency for read-heavy workloads.

```java
public class LockDemo {
    private final ReentrantLock lock = new ReentrantLock(true); // fair
    private final ReadWriteLock rwLock = new ReentrantReadWriteLock();
    private int value;

    public void update(int v) {
        lock.lock();
        try {
            value = v;
        } finally {
            lock.unlock();
        }
    }

    public int read() {
        rwLock.readLock().lock();
        try {
            return value;
        } finally {
            rwLock.readLock().unlock();
        }
    }

    public boolean tryUpdate(int v, long timeout, TimeUnit unit) throws InterruptedException {
        if (lock.tryLock(timeout, unit)) {
            try {
                value = v;
                return true;
            } finally {
                lock.unlock();
            }
        }
        return false;
    }
}
```

---

**Question**: What are atomic classes and how does CAS work?

**Answer**: Atomic classes (`AtomicInteger`, `AtomicLong`, `AtomicReference`, `AtomicBoolean`, etc.) in `java.util.concurrent.atomic` provide lock-free, thread-safe operations on single variables. They use Compare-And-Swap (CAS), a hardware-level atomic instruction: read a value, compare it to an expected value, and if they match, swap in a new value - all in one atomic operation. CAS avoids the overhead of locking and is essential for lock-free data structures.

```java
public class AtomicDemo {
    private final AtomicInteger counter = new AtomicInteger(0);

    public int increment() {
        return counter.incrementAndGet(); // CAS internally
    }

    public int addAndGet(int delta) {
        return counter.addAndGet(delta);
    }

    // Custom CAS loop
    public int update() {
        int prev, next;
        do {
            prev = counter.get();
            next = prev + 1;
        } while (!counter.compareAndSet(prev, next));
        return next;
    }
}
```

---

**Question**: What concurrent collections are available in Java?

**Answer**: The `java.util.concurrent` package provides thread-safe collections: `ConcurrentHashMap` (highly concurrent, non-blocking reads), `CopyOnWriteArrayList` (thread-safe list where writes create a new copy - good for read-heavy scenarios), `ConcurrentLinkedQueue` (lock-free FIFO queue), `ConcurrentLinkedDeque`, `BlockingQueue` variants (`ArrayBlockingQueue`, `LinkedBlockingQueue`, `PriorityBlockingQueue`, `DelayQueue`, `SynchronousQueue`), and `ConcurrentSkipListMap`/`ConcurrentSkipListSet` (sorted concurrent maps/sets).

```java
// BlockingQueue for producer-consumer
BlockingQueue<String> queue = new LinkedBlockingQueue<>(100);

// Producer
new Thread(() -> {
    try { queue.put("item"); } catch (InterruptedException e) {}
}).start();

// Consumer
new Thread(() -> {
    try { String item = queue.take(); } catch (InterruptedException e) {}
}).start();

// Copy-on-write for read-heavy lists
List<String> list = new CopyOnWriteArrayList<>();
list.add("safe"); // creates new copy
```

---

**Question**: Explain deadlock, livelock, and starvation.

**Answer**: Deadlock occurs when two or more threads hold locks that the others need, causing all to wait indefinitely - detected via thread dumps (`jstack`). Livelock is similar but threads are actively running (not blocked) yet making no progress - e.g., two threads repeatedly releasing and re-acquiring locks in response to each other. Starvation happens when a thread is perpetually denied access to a resource because other threads monopolize it - can be mitigated with fair locks.

```java
// Deadlock example
public class DeadlockDemo {
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();

    public void method1() {
        synchronized (lock1) {
            synchronized (lock2) { /* ... */ }
        }
    }

    public void method2() {
        synchronized (lock2) { // acquire in reverse order = deadlock
            synchronized (lock1) { /* ... */ }
        }
    }
}
```

---

**Question**: What is the Java Memory Model and the happens-before relationship?

**Answer**: The Java Memory Model (JMM) defines how threads interact through memory and what constitutes a legal execution. The happens-before guarantee ensures that a write to a variable is visible to a subsequent read of that variable by another thread. Happens-before occurs with: `synchronized` (unlock before lock), `volatile` (write before read), thread `start()` (before the thread's actions), `join()` (thread's actions before the join returns), and `transitive` (if A happens-before B and B happens-before C, then A happens-before C).

---

**Question**: What is the Reflection API and what are its pros and cons?

**Answer**: Reflection allows inspecting and invoking classes, methods, fields, and constructors at runtime, bypassing compile-time checks. Pros: enables frameworks (Spring, Hibernate), serialization, dependency injection, and dynamic proxies. Cons: performance overhead (no JIT optimization), breaks encapsulation (access private members), security implications, and no compile-time type safety. Use with caution and consider `MethodHandle` or `java.lang.invoke` as lighter alternatives.

```java
public class ReflectionDemo {
    public static void main(String[] args) throws Exception {
        Class<?> clazz = Class.forName("java.util.ArrayList");
        Method addMethod = clazz.getMethod("add", Object.class);

        List<String> list = (List<String>) clazz.getDeclaredConstructor().newInstance();
        addMethod.invoke(list, "reflection");

        Field field = String.class.getDeclaredField("value");
        field.setAccessible(true); // breaks encapsulation
        byte[] value = (byte[]) field.get("hello"); // byte[] since Java 9 (Compact Strings)

        // Java 9+ use of VarHandle for safer access
    }
}
```

---

**Question**: Explain annotations - retention policies and creating custom annotations.

**Answer**: Annotations provide metadata for code. Retention policies: `SOURCE` (discarded by compiler - e.g., `@Override`), `CLASS` (retained in bytecode but not at runtime), `RUNTIME` (available at runtime via reflection). Custom annotations are defined with `@interface`, and can include targets (`@Target`), retention (`@Retention`), and optionally documented/inherited.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD, ElementType.TYPE})
public @interface Loggable {
    String level() default "INFO";
    boolean enabled() default true;
}

// Usage
@Loggable(level = "DEBUG")
public class MyService {
    @Loggable
    public void process() { }
}

// Processing at runtime
for (Method m : MyService.class.getMethods()) {
    if (m.isAnnotationPresent(Loggable.class)) {
        Loggable log = m.getAnnotation(Loggable.class);
        System.out.println("Log level: " + log.level());
    }
}
```

---

**Question**: How does the Proxy pattern work with Dynamic Proxy in Java?

**Answer**: `java.lang.reflect.Proxy` creates a dynamic proxy class that implements one or more interfaces at runtime, delegating all method calls to an `InvocationHandler`. This enables AOP-style cross-cutting concerns (logging, security, transactions) without modifying the target class. The proxy exists only at runtime and requires the target to implement interfaces. For class-based proxies, use CGLIB/ByteBuddy (used by Spring).

```java
public interface Service {
    void execute();
}

public class RealService implements Service {
    public void execute() { System.out.println("Business logic"); }
}

public class LoggingHandler implements InvocationHandler {
    private final Object target;

    public LoggingHandler(Object target) { this.target = target; }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Before: " + method.getName());
        Object result = method.invoke(target, args);
        System.out.println("After: " + method.getName());
        return result;
    }
}

// Usage
Service proxy = (Service) Proxy.newProxyInstance(
    Service.class.getClassLoader(),
    new Class[]{Service.class},
    new LoggingHandler(new RealService())
);
proxy.execute();
```

---

**Question**: What are exception handling best practices in Java?

**Answer**: Catch specific exceptions, not generic `Exception` or `Throwable`. Never swallow exceptions in empty catch blocks. Use try-with-resources for auto-closeable resources. Throw early, catch late - propagate exceptions to the appropriate handling layer. Use custom exceptions for domain-specific errors. Log exceptions with context, not just the stack trace. Prefer unchecked exceptions for programming errors.

```java
// Good
try {
    Files.readString(Path.of("file.txt"));
} catch (FileNotFoundException e) {
    logger.error("File not found: {}", path, e);
    throw new MyAppException("Configuration file missing", e);
}

// Bad
try {
    // risky code
} catch (Exception e) { /* swallowed */ }
```

---

**Question**: Differentiate between NIO and IO in Java.

**Answer**: Java IO (java.io) is stream-oriented and blocking - reads are sequential, one byte/character at a time with no buffering control. Java NIO (java.nio, Java 1.4+) is buffer-oriented and non-blocking - data is read into a `Buffer` for processing, and channels support both read and write. NIO also supports selectors for multiplexed I/O (one thread managing multiple channels). NIO is more efficient for high-throughput, scalable server applications.

```java
// IO - blocking stream
try (InputStream in = new FileInputStream("file.txt")) {
    int data = in.read(); // blocks until available
}

// NIO - non-blocking with Buffer
try (FileChannel channel = FileChannel.open(Path.of("file.txt"))) {
    ByteBuffer buffer = ByteBuffer.allocate(1024);
    channel.read(buffer); // returns immediately, may read partial data
    buffer.flip();
    while (buffer.hasRemaining()) {
        System.out.print((char) buffer.get());
    }
}
```

---

**Question**: Explain try-with-resources and the `AutoCloseable` interface.

**Answer**: Try-with-resources (Java 7+) declares resources in the try header that are automatically closed when the block exits, in reverse order of declaration. Any class implementing `AutoCloseable` (or its subinterface `Closeable` which throws `IOException`) can be used. Resources are closed even if an exception occurs, and suppressed exceptions are appended via `Throwable.addSuppressed()`.

```java
public class CustomResource implements AutoCloseable {
    public CustomResource() { System.out.println("Open"); }
    public void use() { System.out.println("Use"); }
    @Override
    public void close() { System.out.println("Close"); }
}

// Auto-close even if exception occurs
try (CustomResource res = new CustomResource()) {
    res.use();
    throw new RuntimeException("Error");
} // Close is called before catch

// Multiple resources
try (ResourceA a = new ResourceA();
     ResourceB b = new ResourceB()) {
    // a.close() called, then b.close() in reverse order
}
```

---

**Question**: What are Records in Java (Java 16+)?

**Answer**: Records are transparent carriers for immutable data. A record declares a final class with automatically generated canonical constructor, `equals()`, `hashCode()`, `toString()`, and accessor methods (named like the field, not `getX()`). All fields are private and final. Records cannot extend other classes but can implement interfaces. They're ideal for DTOs, value objects, and API responses.

```java
public record Point(int x, int y) {
    // Compact constructor - validation
    public Point {
        if (x < 0 || y < 0) {
            throw new IllegalArgumentException("Coordinates must be non-negative");
        }
    }

    // Additional methods allowed
    public double distanceFromOrigin() {
        return Math.sqrt(x * x + y * y);
    }
}

// Usage
Point p = new Point(3, 4);
System.out.println(p.x()); // accessor, not getX()
System.out.println(p); // Point[x=3, y=4]
```

---

**Question**: What are Sealed Classes in Java (Java 17+)?

**Answer**: Sealed classes restrict which other classes or interfaces may extend or implement them. Declared with `sealed` keyword and a `permits` clause listing the allowed subtypes. Subtypes must be `final`, `sealed`, or `non-sealed`. This enables exhaustive pattern matching and a more controlled inheritance hierarchy, improving API design and security.

```java
public sealed class Shape permits Circle, Rectangle, Triangle { }

public final class Circle extends Shape { double radius; }
public final class Rectangle extends Shape { double w, h; }
public non-sealed class Triangle extends Shape { double base, height; } // open to further extension

// Exhaustive pattern matching (Java 17+)
double area(Shape s) {
    return switch (s) {
        case Circle c -> Math.PI * c.radius * c.radius;
        case Rectangle r -> r.w * r.h;
        case Triangle t -> 0.5 * t.base * t.height;
        // no default needed - exhaustive
    };
}
```

---

**Question**: Explain Pattern Matching for `instanceof` (Java 16+).

**Answer**: Pattern matching for `instanceof` eliminates the boilerplate of casting after a type check. It combines the type check and variable declaration into a single expression, and the variable is in scope only when the check succeeds (flow-sensitive scoping). Starting Java 17, it also works with `switch` expressions (preview in 17, final in 21).

```java
// Before Java 16
if (obj instanceof String) {
    String s = (String) obj;
    System.out.println(s.length());
}

// Java 16+
if (obj instanceof String s) {
    System.out.println(s.length()); // no cast needed
    // s is in scope only here
}

// Pattern matching with switch (Java 21+)
String formatted = switch (obj) {
    case Integer i -> "Integer: " + i;
    case String s -> "String: " + s;
    case null -> "null";
    default -> "Unknown";
};
```

---

**Question**: What are Virtual Threads (Project Loom, Java 21+)?

**Answer**: Virtual threads are lightweight, JVM-managed threads that enable high-throughput concurrent applications without the complexity of reactive programming. Unlike platform threads (OS threads), millions of virtual threads can exist - they are mounted on carrier platform threads when running and unmounted when blocking (e.g., I/O). Virtual threads use `Thread.ofVirtual()` or `Executors.newVirtualThreadPerTaskExecutor()`. They're ideal for I/O-bound workloads with many concurrent tasks.

```java
// Creating virtual threads
Thread vThread = Thread.ofVirtual().start(() -> {
    System.out.println("Running on virtual thread");
});

try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> System.out.println("Virtual task"));
}

// Scale - millions of virtual threads
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    IntStream.range(0, 1_000_000).forEach(i ->
        executor.submit(() -> {
            // I/O operation - thread unmounts automatically
        })
    );
}
```

---

**Question**: Explain the Maven/Gradle build lifecycle.

**Answer**: Maven's lifecycle consists of phases in order: `validate`, `compile`, `test`, `package`, `verify`, `install`, `deploy`. Plugins bind to phases to execute goals. Gradle uses a task-based DAG (directed acyclic graph) with named tasks like `compileJava`, `test`, `jar`, `build`. Gradle builds are incremental (only changed files are recompiled) and support parallel execution. Both support dependency management, but Gradle is faster and more flexible.

```xml
<!-- Maven phases -->
<!-- mvn clean compile test package install -->
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration><source>21</source><target>21</target></configuration>
        </plugin>
    </plugins>
</build>
```

```groovy
// Gradle - task-based
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.2.0'
}

java { sourceCompatibility = '21' }

tasks.named('test') {
    useJUnitPlatform()
}
```

---

**Question**: How do you handle multi-threading with `CompletableFuture`?

**Answer**: `CompletableFuture` supports multi-threading via async methods that accept an `Executor`. Supply `supplyAsync(Supplier, Executor)` to run on a custom thread pool. Chain tasks with `thenApplyAsync()`, `thenCompose()`, and combine multiple futures with `allOf()` (wait for all) or `anyOf()` (wait for first). Use `exceptionally()` for error recovery and `handle()` for processing both results and errors.

```java
ExecutorService executor = Executors.newFixedThreadPool(10);
private static final Logger log = LoggerFactory.getLogger(MyClass.class);

CompletableFuture<Double> userFuture = CompletableFuture.supplyAsync(() ->
    fetchUser(), executor);

CompletableFuture<Double> orderFuture = CompletableFuture.supplyAsync(() ->
    fetchOrders(), executor);

// Combine results from parallel operations
CompletableFuture<Double> combined = userFuture
    .thenCombine(orderFuture, (user, orders) -> process(user, orders))
    .exceptionally(ex -> {
        log.error("Processing failed", ex);
        return 0.0;
    });

// Wait for multiple independent tasks
List<Task> tasks = List.of(task1, task2, task3);
List<CompletableFuture<String>> futures = tasks.stream()
    .map(t -> CompletableFuture.supplyAsync(t::process, executor))
    .toList();

CompletableFuture<Void> all = CompletableFuture.allOf(
    futures.toArray(new CompletableFuture[0]));
List<String> results = all.thenApply(v ->
    futures.stream().map(CompletableFuture::join).collect(Collectors.toList())
).join();
```

---

**Question**: What is the Java Platform Module System (JPMS, Java 9+)?

**Answer**: JPMS (Project Jigsaw) introduces modularization with `module-info.java`. Modules encapsulate packages and explicitly declare dependencies (`requires`) and exported packages (`exports`). It provides strong encapsulation (internal packages are hidden), reliable configuration (no classpath ambiguity), and improved security/performance. Key directives: `exports`, `requires`, `opens` (for reflection), `provides ... with` (service loading), `uses`.

```java
// module-info.java
module com.example.myapp {
    requires java.sql;
    requires spring.core;
    requires transitive java.logging;

    exports com.example.myapp.api;
    exports com.example.myapp.dto;

    opens com.example.myapp.internal to spring.core;
    opens com.example.myapp.entities to org.hibernate.orm.core;

    provides com.example.myapp.spi.Plugin
        with com.example.myapp.internal.PluginImpl;
    uses com.example.myapp.spi.Plugin;
}
```

---

**Question**: How do you approach performance tuning and profiling in Java?

**Answer**: Use profilers (Async Profiler, JFR, VisualVM, YourKit) to identify CPU hotspots, memory leaks, and GC issues. Key diagnostics: `jstack` (thread dumps), `jmap` (heap dumps), `jstat` (GC stats), `jcmd` (comprehensive JVM diagnostics). Start with high-level application profiling, then drill into specific areas. Common optimizations: reduce object allocation (object pooling, primitives), tune GC (`-XX:+UseG1GC`, `-XX:MaxGCPauseMillis`), optimize data structures, use connection pooling, and apply caching.

```bash
# JVM flags for diagnostics
# -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xlog:gc*
# -Xms4g -Xmx4g -XX:MetaspaceSize=256m
# -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/path

# Profiling with async-profiler
# profiler.sh -d 30 -e cpu -f profile.jfr <pid>
# profiler.sh -d 30 -e alloc -f allocation.jfr <pid>
```

---

**Question**: Explain the Java Module System directives - `exports`, `requires`, `opens`.

**Answer**: `requires` declares that a module depends on another module at compile and runtime. `exports package` makes the package accessible to all modules (or a specific module with `exports ... to ...`). `opens package` enables runtime reflection (`setAccessible(true)`) on the package's types - needed by frameworks like Spring, Hibernate, and Jackson. Without `opens`, reflection on package-private/internal types fails at runtime.

```java
// module-info.java examples

// Standard exports
exports com.example.app.dto;

// Qualified export - only visible to specific module
exports com.example.app.internal to com.example.admin;

// Open entire module for reflection (all packages)
open module com.example.framework {
    requires spring.context;
}

// Selective opens
module com.example.entities {
    exports com.example.entities;
    opens com.example.entities to org.hibernate.orm.core;
}
```

---

**Question**: What is the difference between `abstract class` and `interface` in Java?

**Answer**: Abstract classes can have constructors, instance fields, and both abstract and concrete methods; a class can extend only one abstract class. Interfaces (pre-Java 8) could only have abstract methods; since Java 8, they can have `default` and `static` methods; since Java 9, private methods. A class can implement multiple interfaces. Use abstract classes for a base with shared state; use interfaces for behavior contracts and multiple inheritance of type.

```java
public abstract class Vehicle {
    protected String brand;
    public abstract void move();
    public void displayBrand() { System.out.println(brand); }
}

public interface Flyable {
    void fly();
    default void glide() { System.out.println("Gliding"); }
}

public class Airplane extends Vehicle implements Flyable {
    public Airplane() { this.brand = "Boeing"; }
    @Override public void move() { System.out.println("Moving"); }
    @Override public void fly() { System.out.println("Flying"); }
}
```

---

**Question**: How does `ConcurrentHashMap` achieve thread safety without locking the entire map?

**Answer**: In Java 8+, `ConcurrentHashMap` uses CAS (Compare-And-Swap) for table initialization and certain updates, and `synchronized` on individual bins (linked list/tree nodes) during concurrent writes. Read operations are generally lock-free (volatile reads). The `size()` method uses a counter maintained with `LongAdder`-like stripes to avoid contention. This design allows concurrent reads and concurrent writes to different bins simultaneously.

```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

// Thread-safe compound operations
map.computeIfAbsent("key", k -> expensiveComputation(k));

// Atomic replace
map.replace("key", 1, 2); // only if current value is 1

// merge - atomic upsert
map.merge("key", 1, Integer::sum);
```

---

**Question**: What are the different types of references in Java - Strong, Soft, Weak, Phantom?

**Answer**: Strong references (normal references) prevent GC of the referent. `SoftReference` is cleared by GC only when memory is low - useful for caches. `WeakReference` is cleared at the next GC cycle - used by `WeakHashMap`. `PhantomReference` cannot be accessed after GC (no `get()` method); it's used for post-mortem cleanup (e.g., direct buffer deallocation). Reference queues (`ReferenceQueue`) notify when references are enqueued.

```java
// Soft reference - cache
SoftReference<Cache> cache = new SoftReference<>(new Cache());
Cache c = cache.get(); // null if GC cleared it

// Weak reference - WeakHashMap uses WeakReference for keys
WeakReference<Object> weak = new WeakReference<>(new Object());
System.out.println(weak.get()); // non-null
System.gc();
System.out.println(weak.get()); // likely null

// Phantom reference - cleanup
ReferenceQueue<Object> queue = new ReferenceQueue<>();
PhantomReference<Object> phantom = new PhantomReference<>(new Object(), queue);
// phantom.get() always returns null
```
