# 60 Questions and Answers - .Net/C# Developer/Engineer

**Question**: What is the purpose of the partial keyword in C#?

**Answer**: It allows splitting a single type definition across multiple files, enabling code organization without inheritance or assembly boundaries (e.g., one file for logic, another for serialization). This is useful for large classes that need to be maintained by different teams.

***

**Question**: When should you use private protected instead of protected?

**Answer**: Use it when you want to expose implementation details only to derived classes within your own assembly but not to external assemblies. It combines the visibility of internal and protected, allowing library authors to control access tightly while enabling inheritance where needed.

***

**Question**: How does memory layout differ between const fields and readonly fields in .NET?

**Answer**: const values are stored in a read-only data segment (RODATA) at compile time, while readonly fields are initialized at runtime and can be on the stack or heap depending on type. This makes const slightly more memory-efficient for primitive types but less flexible than readonly.

***

**Question**: What is the impact of marking a method as sealed on JIT compilation?

**Answer**: It prevents virtual dispatch overhead because the compiler knows no derived class will override it, allowing for more aggressive inlining optimizations by the JIT. This improves performance compared to non-sealed methods which require indirection through the vtable.

***

**Question**: How do access modifiers affect overriding behavior (override) in C#?

**Answer**: The override method must have equal or lower visibility than the base method (e.g., a private method cannot be overridden). If you want to override, ensure the base is virtual, abstract, or sealed.

***

**Question**: What are the implications of using static readonly for thread safety?

**Answer**: It ensures initialization happens once during static constructor execution, making it inherently thread-safe without requiring locks. Unlike instance fields which require synchronization if accessed concurrently, static fields have a single shared state managed by the CLR.

***

**Question**: Why is the virtual keyword necessary when implementing polymorphism in C#?

**Answer**: It marks a method as capable of being overridden by derived classes. Without it, the compiler treats the call as non-virtual (static dispatch), preventing runtime polymorphism and binding to the base class implementation regardless of the object type.

***

**Question**: What is the difference between const and readonly regarding type constraints?

**Answer**: const must be a compile-time constant expression with no types other than primitive or string/enum, while readonly can hold any reference type value assigned in a constructor. This makes readonly more versatile for complex objects.

***

**Question**: How does the internal modifier interact with assembly boundaries?

**Answer**: It restricts access to members within the same assembly unless combined with protected. This is useful for exposing APIs only within your project but not publicly available, helping manage dependency scope.

***

**Question**: When would you use a readonly struct in C#?

**Answer**: Use it when you need value semantics (like primitives) without heap allocation overhead. Unlike classes which are reference types and require copying to modify, structs are passed by value, making them ideal for immutable data structures or high-performance scenarios.

***

**Question**: What is the difference between sealed and abstract classes in C#?

**Answer**: An abstract class cannot be instantiated directly but can have concrete members, while a sealed class prevents inheritance entirely. Use abstract when you need to define common logic that must be implemented by subclasses; use sealed for performance optimization or preventing unintended subclassing.

***

**Question**: Explain the concept of covariance and contravariance in C# generic types.

**Answer**: Covariance allows a derived type to be used where a base type is expected (e.g., List<Derived>). Contravariance applies to method parameters, allowing a more specific type to replace a general one (e.g., Func<Derived, void> vs Func<Base, void>), ensuring type safety in generic delegates.

***

**Question**: Why is composition preferred over inheritance in modern C# development?

**Answer**: Composition promotes loose coupling and higher flexibility. It allows classes to maintain their own identity while using other objects' capabilities (e.g., IRepository interface). Inheritance can lead to tight coupling and "God Classes," making testing harder.

***

**Question**: How does the .NET Garbage Collector work, specifically regarding Generational GC?

**Answer**: The GC uses a generational model where short-lived objects are collected frequently in Gen 0, while long-lived objects move to Gen 1 or Gen 2 and are collected less often. This reduces overhead by assuming most objects die quickly.

***

**Question**: What is the difference between Span<T> and Memory<T>?

**Answer**: Both represent a contiguous sequence of bytes without copying data. Span<T> is stack-allocated and doesn't require bounds checking if passed correctly, while Memory<T> includes length information and can be used with unsafe code or to capture spans from arrays that might grow/shrink.

***

**Question**: When should you use Task vs ValueTask in C#?

**Answer**: Use Task for standard asynchronous operations where allocations are acceptable. Use ValueTask when the operation completes synchronously often (e.g., local database calls) to avoid unnecessary stack allocation overhead, but ensure it doesn't introduce complexity with callers expecting a Task.

***

**Question**: What is the impact of ConfigureAwait(false) in async methods?

**Answer**: It suspends the continuation on the current synchronization context (like UI thread). This prevents deadlocks in multi-threaded environments by allowing the task to run on any available thread, but it may increase latency if the caller expects synchronous execution.

***

**Question**: How do you handle memory leaks in .NET applications?

**Answer**: Common causes include static references or unclosed streams. Mitigate by using using statements for disposable resources, avoiding static caches of large objects, and monitoring heap size via tools like dotnet-dump or Visual Studio Profiler.

***

**Question**: What is the purpose of Dependency Injection (DI) in Clean Architecture?

**Answer**: DI decouples application logic from external frameworks (e.g., Database, HTTP). It enforces dependency direction where inner layers depend on outer layers only through interfaces, making testing and swapping implementations easier.

***

**Question**: Explain the difference between IEnumerable and IQueryable.

**Answer**: IEnumerable executes queries in memory after retrieval, while IQueryable pushes SQL logic to the database provider (e.g., LINQ to Entities). Use IQueryable for large datasets to avoid transferring all rows to the application server.

***

**Question**: What is the "Vertical Slice" architecture pattern?

**Answer**: It organizes code by feature rather than layer (e.g., UserController, UserService, UserRepository). This improves testability and reduces cross-layer dependencies, making it easier to add new features without touching existing logic.

***

**Question**: How does the .NET JIT compiler optimize code?

**Answer**: It translates IL into native machine code at runtime using techniques like inlining, loop unrolling, and constant folding. It also uses the Just-In-Time (JIT) to analyze hot paths for performance tuning based on execution frequency.

***

**Question**: What is the difference between async void and async Task?

**Answer**: Use async Task for methods that can be awaited or thrown exceptions. Avoid async void except for event handlers, as it cannot be awaited and exceptions are swallowed by the runtime unless handled in a specific way.

***

**Question**: How do you implement CQRS (Command Query Responsibility Segregation)?

**Answer**: Separate read operations into a query model optimized for retrieval and write operations into a command model optimized for processing. This allows different data models, caching strategies, and indexing for reads vs writes.

***

**Question**: What is the role of using statements in C#?

**Answer**: They ensure that objects implementing IDisposable are disposed immediately after use via the finalizer pattern (try-finally), preventing resource leaks like file handles or database connections from being held open.

***

**Question**: Explain the concept of "Modular Monolith".

**Answer**: A monolithic application divided into loosely coupled modules, each with its own domain logic and testability boundaries. It offers better maintainability than a single giant app but avoids the complexity of microservices until necessary scale is reached.

***

**Question**: How do you handle exceptions in an async pipeline?

**Answer**: Use try-catch blocks around the async method or let the exception propagate up to a global handler (e.g., ASP.NET Core's middleware). Avoid swallowing errors, as they indicate bugs that need investigation.

***

**Question**: What is the difference between List<T> and Array in terms of performance?

**Answer**: Arrays have better cache locality but fixed size; Lists grow dynamically with overhead. Use arrays for known-size collections to improve performance, or Lists when flexibility is needed.

***

**Question**: How does Task.WhenAll differ from Task.Run?

**Answer**: Task.WhenAll aggregates multiple tasks and waits for all to complete before returning a result. Task.Run schedules a single task on the thread pool. Use WhenAll for parallel operations needing combined results, Run for offloading CPU-bound work.

***

**Question**: What is the purpose of SuppressFinalize in C#?

**Answer**: It tells the GC to skip finalizing an object if it's no longer needed, reducing memory pressure on objects that don't need cleanup logic (e.g., simple data structures).

***

**Question**: How do you manage configuration in .NET applications?

**Answer**: Use appsettings.json or environment variables via IConfiguration. Avoid hardcoding values; use dependency injection to inject configuration into services for flexibility and testing.

***

**Question**: What is the difference between IEnumerable<T> and ICollection<T>?

**Answer**: ICollection<T> adds methods like Add, Remove, and Count that allow modification of the collection, while IEnumerable<T> only supports iteration. Use ICollection when you need to modify the list frequently.

***

**Question**: How do you implement a Repository Pattern in .NET?

**Answer**: Create an interface defining data access methods (e.g., GetById, Save). Implement it using Entity Framework or Dapper, injecting it into services to decouple business logic from database specifics.

***

**Question**: What is the "Unit of Work" pattern and why is it used?

**Answer**: It manages a set of changes made by multiple entities within an application context. It ensures all changes are committed together (ACID) or rolled back if one fails, commonly implemented via Entity Framework's DbContext.

***

**Question**: How do you handle concurrent access to shared resources in C#?

**Answer**: Use synchronization primitives like lock, Mutex, or SemaphoreSlim. For thread-safe collections, use ConcurrentDictionary which handles locking internally without explicit locks.

***

**Question**: What is the impact of string concatenation with + operator on performance?

**Answer**: It creates multiple temporary string objects in memory before assigning to the final result. Use StringBuilder or string interpolation ($"") for better performance, especially in loops.

***

**Question**: How do you implement a Singleton pattern safely in .NET?

**Answer**: Use lazy initialization with Lazy<T> or static field locking (e.g., static readonly). Avoid thread-unsafe singletons by ensuring the instance creation is atomic and protected against race conditions.

***

**Question**: What is the purpose of ConfigureAwait in async methods?

**Answer**: It determines whether to suspend the continuation on the current synchronization context. Use it when you want to avoid deadlocks (e.g., in UI apps) or when the task will be executed on a different thread.

***

**Question**: How do you handle large datasets efficiently with LINQ?

**Answer**: Avoid loading all data into memory by using AsEnumerable() only if necessary, otherwise rely on IQueryable to push queries to the database. Use Take, Skip, and ToListAsync for pagination.

***

**Question**: What is the difference between async void and Task in terms of error handling?

**Answer**: Exceptions in async Task propagate up and can be caught by callers or global handlers. In async void, exceptions are swallowed unless handled via a specific mechanism, making debugging harder.

***

**Question**: How do you implement the Strategy Pattern in C#?

**Answer**: Define an interface for algorithms (e.g., PaymentStrategy). Create concrete implementations (e.g., CreditCardPayment) and inject them into a service that selects the appropriate strategy based on input conditions.

***

**Question**: What is the purpose of IDisposable pattern in .NET?

**Answer**: It ensures resources like file handles or network sockets are released promptly. Implement it using the using statement to guarantee cleanup even if an exception occurs during execution.

***

**Question**: How do you optimize LINQ queries for performance?

**Answer**: Avoid materializing results with .ToList() unless necessary. Use projection (Select) instead of selecting all fields, and leverage database-specific optimizations (e.g., Include for eager loading).

***

**Question**: What is the "Dependency Inversion Principle" in SOLID?

**Answer**: High-level modules should not depend on low-level details; both should depend on abstractions. This allows swapping implementations without changing core logic, improving testability and flexibility.

***

**Question**: How do you handle distributed transactions in microservices?

**Answer**: Use the Saga Pattern or Outbox Pattern to manage transactional consistency across services. Avoid two-phase commit (2PC) as it introduces latency and complexity; use eventual consistency instead.

***

**Question**: What is the difference between Task and ValueTask regarding stack allocation?

**Answer**: Task allocates on the heap, while ValueTask can allocate on the stack if completed synchronously. This reduces memory pressure in high-frequency scenarios but adds complexity to callers expecting a Task.

***

**Question**: How do you implement the Observer Pattern in C#?

**Answer**: Use event-driven programming with Action<T> delegates or IEventEmitter. A publisher emits events, and subscribers subscribe to receive updates without knowing each other's existence.

***

**Question**: What is the purpose of volatile keyword in .NET?

**Answer**: It ensures visibility across threads by preventing compiler optimizations that might reorder memory access. However, it does not provide atomicity; use Interlocked or locks for true synchronization.

***

**Question**: How do you implement a Factory Pattern in C#?

**Answer**: Create an interface (e.g., IFactory) with methods to create objects of different types based on input parameters. This decouples the creation logic from the object usage, allowing easy swapping of implementations.

***

**Question**: What is the impact of Task vs ValueTask on memory allocation?

**Answer**: Task always allocates a heap object for the task state. ValueTask avoids this if completed synchronously, but requires careful handling to avoid confusion in callers expecting Task.

***

**Question**: How do you implement the Repository Pattern with Entity Framework Core?

**Answer**: Create an interface (e.g., IUserRepository) and a concrete class implementing it using EF Core's DbContext. Inject this into services to abstract database logic from business rules.

***

**Question**: What is the purpose of using directive in C#?

**Answer**: It imports namespaces for classes, interfaces, or types used in your codebase. This reduces verbosity and improves readability by avoiding fully qualified names (e.g., System.Console.WriteLine).

***

**Question**: How do you handle configuration changes at runtime in .NET Core?

**Answer**: Use IConfiguration to read settings from environment variables or appsettings.json. Implement a service that reloads configuration when needed, though this is often unnecessary for most apps.

***

**Question**: What is the difference between IEnumerable and ICollection regarding modification?

**Answer**: IEnumerable supports only iteration; ICollection allows adding/removing elements. Use ICollection if you need to modify the collection frequently during runtime.

***

**Question**: How do you implement a Middleware pipeline in ASP.NET Core?

**Answer**: Create a class implementing IApplicationBuilder. Register middleware using .Use() methods, which execute in order (e.g., Logging -> Authentication -> Routing). Use this for cross-cutting concerns like error handling or logging.

***

**Question**: What is the purpose of TaskCompletionSource?

**Answer**: It allows you to create a Task manually and control its completion state (SetResult, SetException). Useful when you need to wait for external events before completing an async operation.

***

**Question**: How do you implement the Observer Pattern with C# Events?

**Answer**: Use Action<T> delegates or IEventEmitter interfaces. A publisher emits events, and subscribers subscribe to receive updates without knowing each other's existence.

***

**Question**: What is the purpose of IDisposable pattern in .NET?

**Answer**: It ensures resources like file handles or network sockets are released promptly. Implement it using the using statement to guarantee cleanup even if an exception occurs during execution.

***

**Question**: How do you implement a Factory Pattern in C#?

**Answer**: Create an interface (e.g., IFactory) with methods to create objects of different types based on input parameters. This decouples the creation logic from the object usage, allowing easy swapping of implementations.

***

**Question**: What is the purpose of ConfigureAwait in async methods?

**Answer**: It determines whether to suspend the continuation on the current synchronization context. Use it when you want to avoid deadlocks (e.g., in UI apps) or when the task will be executed on a different thread.
