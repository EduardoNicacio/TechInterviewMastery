# Go – Interview Questions & Answers

**Question**: How does Go's package export system work, and what determines whether an identifier is exported?

**Answer**: Go exports identifiers based on capitalization: names starting with an uppercase letter are exported (public), while lowercase names are package-private. This is the only visibility mechanism — there is no `public` or `private` keyword. A package must be imported via its module path, and only exported names are accessible outside it.

---

**Question**: What are zero values in Go, and which types have them?

**Answer**: Every type in Go has a zero value assigned automatically when declared without an explicit initializer: `0` for numeric types, `false` for booleans, `""` for strings, and `nil` for pointers, slices, maps, channels, functions, and interfaces. This eliminates the concept of uninitialized variables seen in other languages.

---

**Question**: How does the short variable declaration operator (`:=`) work, and what are its restrictions?

**Answer**: `:=` declares and initializes a variable simultaneously, inferring the type from the right-hand expression. It can only be used inside functions and requires at least one new variable on the left side. Unlike `var`, it cannot be used at the package level.

---

**Question**: When should you use `var` vs `:=` vs `const` in Go?

**Answer**: Use `const` for compile-time immutable values. Use `var` for zero-valued declarations (`var count int`), package-level variables, or when type clarity matters. Use `:=` inside functions for concise declaration and initialization where type inference is sufficient.

---

**Question**: How does the `iota` identifier work in Go's constant declarations?

**Answer**: `iota` is a predeclared integer counter that increments automatically within a `const` block, resetting to 0 for each new block. It's commonly used to create enumerated constants, often with the blank identifier to skip values or bitwise shifts for flags.

```go
const (
    Read  = 1 << iota // 1
    Write             // 2
    Execute           // 4
)
```

---

**Question**: How do structs and struct tags work in Go?

**Answer**: A struct is a composite type grouping fields of possibly different types. Struct tags are string literals attached to field declarations that provide metadata consumed by reflection-based packages like `encoding/json`. Tags do not affect runtime behavior directly — only packages that parse them via `reflect` behave differently.

```go
type User struct {
    Name  string `json:"name,omitempty"`
    Email string `json:"email"`
}
```

---

**Question**: How does type embedding work in Go, and how is it different from inheritance?

**Answer**: Type embedding promotes fields and methods of the embedded type to the outer struct, enabling composition over inheritance. Unlike class inheritance, there is no dynamic dispatch, no virtual methods, and the embedded type retains its own identity. Method calls on promoted methods resolve at compile time, and the outer type can shadow promoted methods but cannot override them polymorphically.

```go
type Reader struct{}
func (r Reader) Read() {}

type Logger struct{}
func (l Logger) Log() {}

type File struct {
    Reader
    Logger
}
```

---

**Question**: What is the zero value of a slice, map, or channel, and can you operate on a nil slice or map?

**Answer**: The zero value for slices, maps, and channels is `nil`. A nil slice has length 0 and capacity 0 and can be appended to (append allocates a new underlying array). A nil map cannot have values written to it — writing causes a panic. A nil channel blocks forever on both send and receive and is useful in `select` statements to disable cases.

---

**Question**: What are goroutines, and how does Go manage their stack growth?

**Answer**: Goroutines are lightweight concurrent execution contexts multiplexed onto OS threads by the Go runtime. Each goroutine starts with a small stack (~2 KB) that grows and shrinks dynamically as needed via stack copying, allowing tens of thousands of goroutines to run efficiently. This contrasts with OS threads that have fixed, larger stacks.

---

**Question**: What is the difference between unbuffered and buffered channels, and what are directional channels?

**Answer**: Unbuffered channels synchronize both sender and receiver — a send blocks until a receive occurs. Buffered channels have capacity and only block when full. Directional channels restrict channel operations: `chan<- T` allows only sends, `<-chan T` allows only receives, providing compile-time type safety.

```go
ch := make(chan int)       // unbuffered
buf := make(chan int, 10)  // buffered
var sendOnly chan<- int = ch
var recvOnly <-chan int = ch
```

---

**Question**: How does the `select` statement work, and what is the use of its `default` case?

**Answer**: `select` blocks until one of its channel operations can proceed, choosing uniformly at random among ready cases. A `default` case makes the select non-blocking — if no channel is ready, the default executes immediately. This enables multiplexing across multiple channels and implementing timeouts or non-blocking communication.

```go
select {
case msg := <-ch1:
    fmt.Println(msg)
case <-time.After(1 * time.Second):
    fmt.Println("timeout")
default:
    fmt.Println("no channels ready")
}
```

---

**Question**: What is `sync.WaitGroup` and how do you use it to wait for goroutines?

**Answer**: `WaitGroup` coordinates concurrent goroutines by blocking until a counter reaches zero. Call `Add(n)` before starting goroutines, `Done()` (typically via `defer`) in each goroutine when it finishes, and `Wait()` to block until the counter reaches zero. It has no internal timeout or error propagation — those require `errgroup` or custom coordination.

```go
var wg sync.WaitGroup
wg.Add(10)
for i := 0; i < 10; i++ {
    go func(id int) {
        defer wg.Done()
        // do work
    }(i)
}
wg.Wait()
```

---

**Question**: When should you use `sync.Mutex` vs `sync.RWMutex`?

**Answer**: Use `sync.Mutex` for exclusive locks when writes are frequent or reads are not significantly more common than writes. Use `sync.RWMutex` when reads vastly outnumber writes, as multiple readers can acquire the read lock simultaneously (`RLock`/`RUnlock`) without blocking each other. `RWMutex` carries a small overhead over `Mutex`.

---

**Question**: What are `sync.Once`, `sync.Pool`, `sync.Map`, and `sync/atomic` used for?

**Answer**: `sync.Once` ensures a function executes exactly once across goroutines — ideal for lazy singletons. `sync.Pool` caches allocated objects to reduce GC pressure. `sync.Map` is optimized for append-heavy concurrent maps where keys are written once and read often. `sync/atomic` provides lock-free operations (Add, CompareAndSwap, Load, Store) for high-performance counters and flags. For most cases, prefer a regular map guarded by `sync.Mutex` over `sync.Map`.

---

**Question**: Describe the worker pool pattern in Go.

**Answer**: A worker pool spawns a fixed number of goroutines that consume jobs from a shared buffered channel and send results to another channel. This controls concurrency, prevents unbounded goroutine creation, and enables graceful shutdown by closing the jobs channel and using a `WaitGroup` to wait for all workers to finish.

```go
func worker(id int, jobs <-chan Job, results chan<- Result, wg *sync.WaitGroup) {
    defer wg.Done()
    for job := range jobs {
        results <- process(job)
    }
}
```

---

**Question**: How does `errgroup` handle error propagation across goroutines?

**Answer**: `errgroup.Group` from `golang.org/x/sync/errgroup` extends `WaitGroup` with error propagation. It launches goroutines via `Go` and returns the first non-nil error from `Wait`. Canceling the group's derived context immediately stops other goroutines when one fails, enabling coordinated error handling without manual channel management.

```go
g, ctx := errgroup.WithContext(ctx)
for _, task := range tasks {
    task := task
    g.Go(func() error {
        return task.run(ctx)
    })
}
if err := g.Wait(); err != nil {
    // handle first error
}
```

---

**Question**: How does the `context` package support deadlines, cancellation, and value propagation?

**Answer**: `context.Context` carries deadlines, cancellation signals, and request-scoped values across API boundaries. Derive cancellable contexts with `context.WithCancel`, add deadlines with `context.WithTimeout` or `context.WithDeadline`, and propagate values with `context.WithValue`. Select on `ctx.Done()` to react to cancellation in long-running operations.

```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

select {
case result := <-doWork(ctx):
    fmt.Println(result)
case <-ctx.Done():
    fmt.Println("canceled:", ctx.Err())
}
```

---

**Question**: How does Go's race detector work, and how do you enable it?

**Answer**: The race detector instruments memory accesses during compilation to detect unsynchronized concurrent reads and writes. Enable it with the `-race` flag during `go build`, `go test`, or `go run`. It imposes a significant runtime overhead but catches data races reliably. It reports the exact goroutines and stack traces involved when a race is detected.

---

**Question**: How does Go's interface satisfaction work, and what is "duck typing"?

**Answer**: Interfaces in Go are satisfied implicitly — a type implements an interface simply by having all required methods. There is no `implements` keyword. This is structural typing, often called "duck typing," where a type's suitability is determined by its method set, not an explicit declaration.

```go
type Stringer interface {
    String() string
}

type User struct{ Name string }
func (u User) String() string { return u.Name } // User satisfies Stringer
```

---

**Question**: What is the empty interface (`interface{}` or `any`), and when should you use it?

**Answer**: The empty interface has no methods, so every type in Go satisfies it. Use it sparingly for functions that must accept values of arbitrary type, like `fmt.Println` or JSON deserialization. Prefer generics (Go 1.18+) over `interface{}` for type-safe, flexible APIs, as the empty interface loses all compile-time type information.

---

**Question**: How do type assertions work in Go, and what is the comma-ok pattern?

**Answer**: A type assertion extracts the concrete value from an interface type. The comma-ok pattern returns a boolean indicating success instead of panicking on failure. Always use the two-value form when the assertion may fail.

```go
var val any = "hello"
s, ok := val.(string)
if ok {
    fmt.Println(s)
}
```

---

**Question**: How does a type switch differ from a regular switch in Go?

**Answer**: A type switch explicitly switches on the dynamic type of an interface value, using the syntax `v.(type)` as the switch expression. Each case specifies a type, and the variable is automatically cast to that type within the case block. It's the idiomatic way to handle multiple possible concrete types behind an interface.

```go
func describe(v any) {
    switch t := v.(type) {
    case string:
        fmt.Println("string:", t)
    case int:
        fmt.Println("int:", t)
    default:
        fmt.Printf("unknown: %T\n", t)
    }
}
```

---

**Question**: How do generics work in Go 1.18+, including type parameters, constraints, `comparable`, and the `~` tilde constraint?

**Answer**: Generics introduce type parameters in square brackets, constrained by interfaces. The `comparable` constraint allows `==` and `!=` comparisons. The `~` tilde prefix in constraints permits any type whose underlying type matches, enabling constraints like `~int` that accept `type MyInt int` as well as plain `int`.

```go
func Min[T cmp.Ordered](a, b T) T {
    if a < b { return a }
    return b
}

type Age uint
func Process[T ~uint](val T) {} // accepts Age and uint
```

---

**Question**: What is the `error` interface in Go, and how do you create sentinel errors?

**Answer**: The `error` interface has a single method `Error() string`. Sentinel errors are predefined package-level errors created with `errors.New` or `fmt.Errorf`, used for comparing error identity via `==`. They signal specific, expected failure conditions that callers check against.

```go
var ErrNotFound = errors.New("item not found")

func Get(key string) (Value, error) {
    if !exists(key) {
        return Value{}, ErrNotFound
    }
    // ...
}
```

---

**Question**: How do you create custom error types in Go, and why would you?

**Answer**: Custom error types implement the `error` interface by defining an `Error() string` method. They carry additional context — status codes, retry metadata, or underlying errors — enabling richer error handling via type assertions or `errors.As` for structured inspection.

```go
type HTTPError struct {
    Code    int
    Message string
}

func (e *HTTPError) Error() string {
    return fmt.Sprintf("HTTP %d: %s", e.Code, e.Message)
}
```

---

**Question**: How does error wrapping work with `%w`, `errors.Is`, and `errors.As`?

**Answer**: Use `fmt.Errorf("...: %w", err)` to wrap an error, creating an error chain. `errors.Is` unwraps the chain to find a matching sentinel. `errors.As` finds the first error in the chain that matches a target type and assigns it. Wrapping preserves the original error while adding context.

```go
if err := doSomething(); err != nil {
    return fmt.Errorf("doSomething failed: %w", err)
}

// caller
if errors.Is(err, ErrNotFound) { ... }
var httpErr *HTTPError
if errors.As(err, &httpErr) { ... }
```

---

**Question**: When is `panic` and `recover` appropriate in Go?

**Answer**: `panic` should only be used for truly exceptional, unrecoverable conditions — programmer bugs, impossible state invariants, or initialization failures. `recover` captures a panic from within a deferred function to prevent a crash, typically at top-level request handlers or goroutine boundaries. Never use panic for normal error handling.

```go
defer func() {
    if r := recover(); r != nil {
        log.Printf("recovered from panic: %v", r)
    }
}()
```

---

**Question**: How do you write table-driven tests in Go using the `testing` package?

**Answer**: Define a slice of test case structs with input and expected output, then iterate over them with `t.Run` for subtests. Each subtest gets a name from the case, enabling selective execution via `go test -run`. Table-driven tests reduce duplication and make adding new cases trivial.

```go
func TestAdd(t *testing.T) {
    tests := []struct {
        name string
        a, b int
        want int
    }{
        {"positives", 2, 3, 5},
        {"negatives", -1, -1, -2},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            if got := Add(tt.a, tt.b); got != tt.want {
                t.Errorf("Add() = %d, want %d", got, tt.want)
            }
        })
    }
}
```

---

**Question**: How do benchmarks work in Go, and what is fuzzing?

**Answer**: Benchmarks are functions with the signature `func BenchmarkX(b *testing.B)` that measure performance by running the code `b.N` times. Fuzzing (Go 1.18+) uses functions `func FuzzX(f *testing.F)` where the framework generates random inputs to find edge cases and panics. Both are integrated into the `testing` package and run via `go test -bench=.` or `go test -fuzz=.`.

```go
func BenchmarkSum(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Sum(1, 2)
    }
}

func FuzzParse(f *testing.F) {
    f.Add("valid input")
    f.Fuzz(func(t *testing.T, input string) {
        Parse(input)
    })
}
```

---

**Question**: What does `go vet` check, and how do you enable the race detector?

**Answer**: `go vet` statically analyzes code for suspicious constructs — unreachable code, incorrect `Printf` format strings, unused function results, and unsafe atomic operations. Run it as part of CI to catch common bugs without writing additional tests.

---

**Question**: How do you use `pprof` and the execution trace tool for profiling Go programs?

**Answer**: `go tool pprof` analyzes CPU, heap, goroutine, and mutex profiles. Use `net/http/pprof` (import `_ "net/http/pprof"`) for HTTP-based profiling of running servers, or `runtime/pprof` for programmatic profiling in CLI tools. `go tool trace` visualizes goroutine scheduling, GC pauses, and syscall activity over time, captured by calling `trace.Start`/`trace.Stop`. Enable the HTTP profiler by importing `_ "net/http/pprof"`. Use pprof for allocation hotspots and trace for concurrency bottlenecks.

---

**Question**: How do you build HTTP handlers and middleware chains in Go?

**Answer**: Handlers implement `http.Handler` (`ServeHTTP(ResponseWriter, *Request)`). `http.ServeMux` routes requests by pattern. Middleware wraps a handler by accepting and returning an `http.Handler`, enabling cross-cutting concerns like logging, auth, and request ID injection through function composition.

```go
func loggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        log.Printf("%s %s", r.Method, r.URL.Path)
        next.ServeHTTP(w, r)
    })
}

mux := http.NewServeMux()
mux.HandleFunc("/api", apiHandler)
http.ListenAndServe(":8080", loggingMiddleware(mux))
```

---

**Question**: What are the `io.Reader` and `io.Writer` interfaces, and why are they fundamental?

**Answer**: `io.Reader` defines `Read(p []byte) (n int, err error)` and `io.Writer` defines `Write(p []byte) (n int, err error)`. These two interfaces underpin Go's entire I/O model — files, network connections, buffers, compressors, and encoders all satisfy them. Functions written against these interfaces work with any data source or sink interchangeably.

---

**Question**: How does `encoding/json` handle marshaling, unmarshaling, and struct tags?

**Answer**: `json.Marshal` serializes Go values to JSON; `json.Unmarshal` deserializes JSON into Go values. Struct tags like `json:"fieldname,omitempty"` control the output field name and omit zero values. Use custom marshaling via `json.Marshaler` for non-standard encodings. Unexported fields are never marshaled. Custom marshaling is possible via `json.Marshaler` and `json.Unmarshaler` interfaces.

```go
type Config struct {
    Host string `json:"host"`
    Port int    `json:"port,omitempty"`
}

data, _ := json.Marshal(Config{Host: "localhost"})
// {"host":"localhost"}
```

---

**Question**: How does the `time` package handle durations, tickers, and timeouts?

**Answer**: `time.Duration` is a named type wrapping `int64` nanoseconds, with convenient constants (`time.Second`). `time.Ticker` fires at regular intervals through a channel. `time.After` returns a channel that fires once after a duration, commonly used with `select` for timeouts. Always drain tickers to prevent goroutine leaks.

```go
ticker := time.NewTicker(1 * time.Second)
defer ticker.Stop()

select {
case <-ticker.C:
    // periodic work
case <-time.After(5 * time.Second):
    // timeout
}
```
