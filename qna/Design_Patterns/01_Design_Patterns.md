# Design Patterns – Interview Questions & Answers

**Question**: How do you implement a thread-safe Singleton with double-checked locking?

**Answer**: Use a `volatile` static field with a nested lock check — the outer check avoids lock contention once the instance exists, the inner check ensures only one thread creates it. Use `Lazy<T>` or an enum in Java for simpler, guaranteed-safe alternatives. When to use: when exactly one instance must coordinate global state (e.g., logging, caching). Pitfall: forgetting `volatile` can let a thread see a partially constructed object.

```csharp
public sealed class ThreadSafeSingleton
{
    private static volatile ThreadSafeSingleton? _instance;
    private static readonly object _lock = new();

    private ThreadSafeSingleton() { }

    public static ThreadSafeSingleton Instance
    {
        get
        {
            if (_instance is null)
            {
                lock (_lock)
                {
                    _instance ??= new ThreadSafeSingleton();
                }
            }
            return _instance;
        }
    }
}

// Simpler alternative using Lazy<T>
public sealed class LazySingleton
{
    private static readonly Lazy<LazySingleton> _instance =
        new(() => new LazySingleton());
    private LazySingleton() { }
    public static LazySingleton Instance => _instance.Value;
}
```

---

**Question**: How does the Factory Method pattern differ from Simple Factory?

**Answer**: Factory Method defines an interface for creating an object but lets subclasses decide which class to instantiate. Simple Factory moves creation into a single static method with conditional logic. When to use: when a class can't anticipate the type of objects it must create, or you want subclasses to specify the object type. Pitfall: overuse leads to many small factory subclasses — consider Abstract Factory or DI instead.

```csharp
public abstract class DocumentProcessor
{
    // Factory Method
    public abstract IDocument CreateDocument();

    public void Process()
    {
        var doc = CreateDocument();
        doc.Open();
        doc.Parse();
    }
}

public class PdfProcessor : DocumentProcessor
{
    public override IDocument CreateDocument() => new PdfDocument();
}

public class WordProcessor : DocumentProcessor
{
    public override IDocument CreateDocument() => new WordDocument();
}
```

---

**Question**: When should you use Abstract Factory over Factory Method?

**Answer**: Abstract Factory provides an interface for creating *families of related or dependent products* without specifying their concrete classes (e.g., UI widgets for Windows vs. macOS). Factory Method creates a single product; Abstract Factory composes multiple Factory Methods. When to use: when your system must be configured with one of multiple product families. Pitfall: adding a new product family requires changing the abstract interface and all implementations.

```csharp
public interface IUIFactory
{
    IButton CreateButton();
    ICheckbox CreateCheckbox();
}

public class WinFactory : IUIFactory
{
    public IButton CreateButton() => new WinButton();
    public ICheckbox CreateCheckbox() => new WinCheckbox();
}

public class MacFactory : IUIFactory
{
    public IButton CreateButton() => new MacButton();
    public ICheckbox CreateCheckbox() => new MacCheckbox();
}
```

---

**Question**: How do you build immutable objects with a fluent Builder pattern?

**Answer**: A Builder exposes chainable `With*` methods that return `this`, culminating in a `Build()` call that produces the immutable object. This eliminates telescoping constructors and improves readability. When to use: when an object has many optional parameters or requires a multi-step construction process. Pitfall: forgetting to call `Build()` leads to uninitialized objects — enforce validation inside `Build()`.

```csharp
public class FluentBuilder
{
    private string _name = string.Empty;
    private int _port;
    private bool _useTls;

    public FluentBuilder WithName(string name)   { _name = name; return this; }
    public FluentBuilder WithPort(int port)      { _port = port; return this; }
    public FluentBuilder WithTls(bool useTls)    { _useTls = useTls; return this; }

    public Configuration Build() => new(_name, _port, _useTls);
}

// Immutable result
public record Configuration(string Name, int Port, bool UseTls);

// Usage: new FluentBuilder().WithName("api").WithPort(443).WithTls(true).Build();
```

---

**Question**: What is the difference between shallow copy and deep copy in the Prototype pattern?

**Answer**: Shallow copy duplicates the object but shares references to mutable child objects; deep copy recursively clones all reachable objects, producing a fully independent graph. When to use: when object creation is expensive (e.g., loading config) and cloning is cheaper. Pitfall: shallow copies cause unintended shared-state bugs — always verify that `MemberwiseClone()` is safe for your use case.

```csharp
public class Report : ICloneable
{
    public string Title { get; set; } = "";
    public List<string> Sections { get; set; } = new();

    // Shallow copy — Sections list is shared
    public object Clone() => MemberwiseClone();

    // Deep copy
    public Report DeepClone() => new()
    {
        Title = Title,
        Sections = new List<string>(Sections)
    };
}
```

---

**Question**: Is Dependency Injection a design pattern or just a technique? How does it differ from Service Locator?

**Answer**: DI is a pattern where an object receives its dependencies from an external source (constructor, property, or method) rather than creating them internally. Service Locator *hides* dependencies by having objects pull from a global registry, making it harder to test and reason about. When to use DI: for most application services — it's the foundation of testable, decoupled code. Simple utilities or performance-critical inner loops may not warrant it. Pitfall: Service Locator is now widely considered an anti-pattern because it obscures dependencies and violates the Inversion of Control principle.

```csharp
// Constructor Injection (preferred)
public class OrderService(IPaymentGateway gateway, ILogger logger)
{
    // Dependencies are explicit — easy to mock
}

// Service Locator anti-pattern
public class OrderService
{
    private IPaymentGateway _gateway => Locator.Resolve<IPaymentGateway>();
    // Hidden dependency — makes testing harder
}
```

---

**Question**: How do you implement an Object Pool and when is it useful?

**Answer**: Maintain a pre-allocated set of reusable objects in a concurrent collection (e.g., `ConcurrentBag<T>`). Clients acquire, use, and return objects to the pool instead of creating new ones. When to use: when construction/destruction is costly (e.g., database connections, thread objects, large buffers). Pitfall: pooled objects must be reset to a clean state before reuse, and pool sizing requires careful tuning.

```csharp
public class ObjectPool<T> where T : new()
{
    private readonly ConcurrentBag<T> _items = new();
    private readonly int _maxSize;

    public ObjectPool(int maxSize) => _maxSize = maxSize;

    public T Get() => _items.TryTake(out var item) ? item : new T();

    public void Return(T item)
    {
        if (_items.Count < _maxSize)
            _items.Add(item);
    }
}
```

---

**Question**: What is the Multiton pattern and how is it different from Singleton?

**Answer**: Multiton (or Registry) manages a map of named instances, returning the same instance for the same key — essentially a Singleton-per-key. When to use: when you need controlled instances keyed by identifier (e.g., per-tenant database connections, per-currency formatters). Pitfall: it's essentially a global mutable registry, making testing harder; consider DI scoped containers instead.

```csharp
public class Multiton
{
    private static readonly ConcurrentDictionary<string, Multiton> _instances = new();

    private Multiton() { }

    public static Multiton GetInstance(string key) =>
        _instances.GetOrAdd(key, _ => new Multiton());
}
```

---

**Question**: Explain the difference between a class Adapter and an object Adapter.

**Answer**: Class Adapter uses inheritance (adapts by subclassing both target and adaptee) and is static at compile time; object Adapter uses composition (wraps an adaptee instance) and is more flexible. When to use: prefer object Adapter — it works with any adaptee subclass and doesn't require multiple inheritance. Pitfall: class Adapter locks you into a single adaptee, limiting reuse.

```csharp
// Object Adapter (composition — preferred)
public class LegacyLoggerAdapter : INewLogger
{
    private readonly LegacyLogger _legacy;

    public LegacyLoggerAdapter(LegacyLogger legacy) => _legacy = legacy;

    public void Log(string message) => _legacy.WriteToFile(message);
}

// Class Adapter (inheritance — requires access to adaptee's class)
public class ClassAdapter : LegacyLogger, INewLogger
{
    public void Log(string message) => WriteToFile(message);
}
```

---

**Question**: How does the Bridge pattern decouple abstraction from implementation?

**Answer**: Bridge splits a concept into two orthogonal hierarchies — an abstraction layer and its implementation — connected via composition, not inheritance. When to use: when you want to avoid a permanent binding between abstraction and implementation (e.g., different OS APIs for rendering, device drivers). Pitfall: over-engineering — don't use Bridge unless both hierarchies genuinely vary independently.

```csharp
// Implementation hierarchy
public interface IRenderer
{
    void RenderCircle(float x, float y, float radius);
}

public class VectorRenderer : IRenderer { /* ... */ }
public class RasterRenderer : IRenderer { /* ... */ }

// Abstraction hierarchy
public abstract class Shape(IRenderer renderer)
{
    public abstract void Draw();
}

public class Circle(IRenderer renderer, float x, float y, float r)
    : Shape(renderer)
{
    public override void Draw() => renderer.RenderCircle(x, y, r);
}
```

---

**Question**: How do you implement the Composite pattern for tree structures?

**Answer**: Define a component interface with minimal common operations (e.g., `Name`, `GetSize`). Leaf nodes implement these directly; Composite nodes store children and delegate by summing across them. When to use: when objects must be treated uniformly in a part-whole hierarchy (e.g., UI tree, file system). Pitfall: enforcing child operations on leaf nodes violates ISP — consider separate `ILeaf` and `IComposite` interfaces.

```csharp
public interface IFileSystemNode
{
    string Name { get; }
    long GetSize();
}

public class File : IFileSystemNode
{
    public string Name { get; set; } = "";
    public long Size { get; set; }
    public long GetSize() => Size;
}

public class Directory : IFileSystemNode
{
    private readonly List<IFileSystemNode> _children = new();
    public string Name { get; set; } = "";
    public void Add(IFileSystemNode node) => _children.Add(node);
    public long GetSize() => _children.Sum(c => c.GetSize());
}
```

---

**Question**: How does the Decorator pattern add behavior at runtime without subclassing?

**Answer**: A decorator implements the same interface as the component it wraps, forwarding calls and adding behavior before/after delegation. Multiple decorators can be stacked. When to use: when you need to add responsibilities to individual objects dynamically (e.g., .NET streams, ASP.NET Core middleware pipeline). Pitfall: decorator chains can become deep and hard to debug — keep decorated types focused.

```csharp
public interface IStream
{
    byte[] Read(int count);
}

public class FileStream : IStream
{
    public byte[] Read(int count) { /* read from disk */ }
}

public class CompressedStream(IStream inner) : IStream
{
    public byte[] Read(int count)
    {
        var data = inner.Read(count);
        return Decompress(data);
    }
}

public class EncryptedStream(IStream inner) : IStream
{
    public byte[] Read(int count)
    {
        var data = inner.Read(count);
        return Decrypt(data);
    }
}

// Usage: new EncryptedStream(new CompressedStream(new FileStream())).Read(1024);
```

---

**Question**: When should you use Facade over just calling subsystem classes directly?

**Answer**: Facade provides a unified, simplified interface to a complex subsystem, reducing coupling and learning curve for clients. When to use: when you need to shield clients from subsystem complexity (e.g., a `OrderFacade` that coordinates inventory, payment, and shipping). Pitfall: Facade can become a god object if it exposes too many subsystem methods — keep it focused on a single use case.

```csharp
public class OrderFacade
{
    private readonly InventorySystem _inventory = new();
    private readonly PaymentSystem _payment = new();
    private readonly ShippingSystem _shipping = new();

    public void PlaceOrder(Order order)
    {
        _inventory.Reserve(order.Items);
        _payment.Charge(order.Total);
        _shipping.Schedule(order);
    }
}
```

---

**Question**: How does the Flyweight pattern reduce memory usage for many small objects?

**Answer**: Flyweight separates intrinsic state (shared, context-independent) from extrinsic state (unique, passed at call time). Shared objects are stored in a factory and reused. When to use: when a large number of fine-grained objects would exhaust memory (e.g., text editors storing character glyphs). Pitfall: if most state is extrinsic, the performance overhead of passing it each time negates the memory savings.

```csharp
public class TreeType
{
    public string Name { get; }     // Intrinsic — shared
    public string Color { get; }

    public TreeType(string name, string color) => (Name, Color) = (name, color);

    public void Draw(int x, int y)  // Extrinsic — passed per call
    { /* render at (x, y) */ }
}

public class TreeFactory
{
    private static readonly ConcurrentDictionary<string, TreeType> _types = new();

    public static TreeType GetTreeType(string name, string color) =>
        _types.GetOrAdd($"{name}_{color}", _ => new TreeType(name, color));
}
```

---

**Question**: What are the three common types of Proxy and when do you use each?

**Answer**: Virtual Proxy defers object creation until needed (lazy loading). Protection Proxy controls access based on permissions. Remote Proxy represents an object in a different address space (e.g., gRPC client stub). When to use: any time you need a surrogate to control access, lifecycle, or location of a real subject. Pitfall: proxies add indirection — don't use them if direct access suffices.

```csharp
// Virtual Proxy — lazy loading
public class LazyImageProxy : IImage
{
    private HighResImage? _realImage;
    private readonly string _filePath;

    public LazyImageProxy(string filePath) => _filePath = filePath;

    public void Display()
    {
        _realImage ??= new HighResImage(_filePath);
        _realImage.Display();
    }
}

// Protection Proxy
public class AdminOnlyProxy : IService
{
    private readonly IService _real;
    public AdminOnlyProxy(IService real) => _real = real;
    public void DeleteAll()
    {
        if (!IsAdmin()) throw new UnauthorizedAccessException("Admins only");
        _real.DeleteAll();
    }

    private bool IsAdmin() => /* check current user role */;
}
```

---

**Question**: What is the Private Class Data pattern and why is it useful?

**Answer**: It encapsulates class attributes into a separate immutable data object, preventing the owning class from accidentally modifying its own state. When to use: when a class has write-once, read-many fields that should not change after construction (e.g., security credentials, configuration). Pitfall: can over-complicate simple classes — only apply when you need strict write protection.

```csharp
public sealed class ConnectionData
{
    public string Host { get; }
    public int Port { get; }
    public ConnectionData(string host, int port) => (Host, Port) = (host, port);
}

public class SecureConnection
{
    private readonly ConnectionData _data;
    public SecureConnection(string host, int port) => _data = new(host, port);
    // _data.Host and _data.Port are read-only even within SecureConnection
}
```

---

**Question**: How does the Strategy pattern enable interchangeable algorithms?

**Answer**: Strategy defines a family of algorithms, encapsulates each one behind a common interface, and makes them interchangeable at runtime. When to use: when you have multiple ways to perform an operation and want to select the algorithm dynamically (e.g., sorting strategies, payment gateways). Pitfall: clients must be aware of available strategies — consider combining with Factory to encapsulate selection logic.

```csharp
public interface ISortStrategy
{
    void Sort(int[] data);
}

public class QuickSort : ISortStrategy
{
    public void Sort(int[] data) { /* quicksort */ }
}

public class MergeSort : ISortStrategy
{
    public void Sort(int[] data) { /* mergesort */ }
}

public class Sorter(ISortStrategy strategy)
{
    public void Sort(int[] data) => strategy.Sort(data);
}
```

---

**Question**: How do you implement the Observer pattern with a publish-subscribe model?

**Answer**: The subject maintains a list of observers and notifies them via a method call (push) or by letting them pull state. In .NET, events provide idiomatic Observer support. When to use: when a change in one object requires updating others without tight coupling (e.g., UI event handlers, event emitters). Pitfall: observers that are never unsubscribed cause memory leaks — always use weak references or ensure explicit unsubscribe.

```csharp
public class NewsPublisher
{
    private readonly List<IObserver<string>> _observers = new();

    public void Subscribe(IObserver<string> observer) => _observers.Add(observer);
    public void Unsubscribe(IObserver<string> observer) => _observers.Remove(observer);

    public void PublishNews(string news)
    {
        foreach (var obs in _observers)
            obs.OnNext(news);
    }
}

// Alternatively with events
public class EventPublisher
{
    public event EventHandler<string>? NewsPublished;
    public void Publish(string news) => NewsPublished?.Invoke(this, news);
}
```

---

**Question**: How does the Command pattern support undo/redo operations?

**Answer**: Each operation is encapsulated as a Command object with `Execute()` and `Undo()` methods. An invoker stores a history stack, calling `Undo()` to reverse the last command. When to use: when you need to parameterize, queue, or replay operations (e.g., editor undo/redo, task queues). Pitfall: commands can accumulate state — implement a bounded history or snapshot size.

```csharp
public interface ICommand
{
    void Execute();
    void Undo();
}

public class TextEditor
{
    private readonly Stack<ICommand> _history = new();
    private string _text = "";

    public void ExecuteCommand(ICommand cmd)
    {
        cmd.Execute();
        _history.Push(cmd);
    }

    public void Undo()
    {
        if (_history.TryPop(out var cmd))
            cmd.Undo();
    }
}
```

---

**Question**: How do you build a middleware pipeline using Chain of Responsibility?

**Answer**: Each handler holds a reference to the next handler. A request is passed along the chain until a handler processes it or the chain ends. When to use: when multiple objects can handle a request and the handler is determined at runtime (e.g., ASP.NET Core middleware, logging levels, authentication filters). Pitfall: ensuring every request is handled — add a terminal handler at the end of the chain.

```csharp
public abstract class LogHandler
{
    protected LogHandler? _next;
    public LogHandler SetNext(LogHandler next) { _next = next; return next; }
    public virtual void Handle(string message, LogLevel level)
    {
        if (_next is not null)
            _next.Handle(message, level);
    }
}

public class InfoHandler : LogHandler
{
    public override void Handle(string message, LogLevel level)
    {
        if (level <= LogLevel.Info)
            Console.WriteLine($"INFO: {message}");
        else
            base.Handle(message, level);
    }
}

public class ErrorHandler : LogHandler
{
    public override void Handle(string message, LogLevel level)
    {
        if (level >= LogLevel.Error)
            Console.WriteLine($"ERROR: {message}");
        else
            base.Handle(message, level);
    }
}
```

---

**Question**: How does the State pattern implement a finite state machine?

**Answer**: The context delegates behavior to a current state object; when an event occurs, the state transitions to a new state by swapping the current state reference. When to use: when an object's behavior depends on its internal state and changes at runtime (e.g., order processing: New → Paid → Shipped → Delivered). Pitfall: state explosion — if states grow combinatorially, consider a state machine library instead.

```csharp
public interface IOrderState
{
    void Pay(OrderContext context);
    void Ship(OrderContext context);
}

public class NewState : IOrderState
{
    public void Pay(OrderContext context)
    {
        Console.WriteLine("Payment processed");
        context.SetState(new PaidState());
    }
    public void Ship(OrderContext context) =>
        Console.WriteLine("Cannot ship unpaid order");
}

public class PaidState : IOrderState
{
    public void Pay(OrderContext context) =>
        Console.WriteLine("Already paid");
    public void Ship(OrderContext context)
    {
        Console.WriteLine("Order shipped");
        context.SetState(new ShippedState());
    }
}

public class OrderContext
{
    private IOrderState _state = new NewState();
    public void SetState(IOrderState state) => _state = state;
    public void Pay() => _state.Pay(this);
    public void Ship() => _state.Ship(this);
}
```

---

**Question**: What are hook methods in the Template Method pattern?

**Answer**: Hook methods are optional steps in a skeleton algorithm that subclasses *may* override, whereas abstract methods *must* be overridden. When to use: when you want to define an algorithm's invariant structure while letting subclasses customize specific steps (e.g., data import: connect → parse → transform → save). Pitfall: excessive hooks make the template fragile — limit them to truly optional behavior.

```csharp
public abstract class DataImporter
{
    // Template Method — defines the skeleton
    public void Import(string path)
    {
        Open(path);
        Parse();
        Transform();   // Hook — default no-op
        Save();
    }

    private void Open(string path) { /* open file */ }
    protected abstract void Parse();
    protected virtual void Transform() { }   // Hook
    private void Save() { /* save to DB */ }
}

public class CsvImporter : DataImporter
{
    protected override void Parse() { /* parse CSV */ }
    protected override void Transform() { /* transform CSV rows */ }
}
```

---

**Question**: How does the Iterator pattern provide uniform traversal?

**Answer**: Iterator defines a common interface (`MoveNext()`, `Current`, `Reset()`) to traverse a collection without exposing its underlying representation. In C#, `foreach` works via `IEnumerable<T>` / `IEnumerator<T>`. When to use: when you need to iterate over different collection types with the same API, or when you need multiple concurrent traversals. Pitfall: modifying the collection during iteration throws `InvalidOperationException` — use snapshots or concurrent collections.

```csharp
public class TreeNode<T>
{
    public T Value { get; set; }
    public TreeNode<T>? Left { get; set; }
    public TreeNode<T>? Right { get; set; }
}

public class InOrderIterator<T> : IEnumerator<T>
{
    private readonly Stack<TreeNode<T>> _stack = new();
    private TreeNode<T>? _current;

    public InOrderIterator(TreeNode<T>? root)
    {
        PushLeft(root);
    }

    public bool MoveNext()
    {
        if (_stack.Count == 0) return false;
        _current = _stack.Pop();
        PushLeft(_current.Right);
        return true;
    }

    private void PushLeft(TreeNode<T>? node)
    {
        while (node is not null)
        {
            _stack.Push(node);
            node = node.Left;
        }
    }

    public void Reset() => throw new NotSupportedException("Reset is not supported for this iterator");

    public T Current => _current!.Value;
    // Dispose omitted for brevity
}
```

---

**Question**: How does the Mediator pattern reduce chaotic dependencies?

**Answer**: Components communicate through a central Mediator object instead of referencing each other directly, turning many-to-many interactions into one-to-many. When to use: when a set of objects communicate in complex ways and you want to centralize control logic (e.g., chat room, UI dialog coordinating widgets). Pitfall: the mediator can become a god object — split domain-specific mediators if it grows too large.

```csharp
public interface IChatMediator
{
    void SendMessage(string message, User sender);
}

public class ChatRoom : IChatMediator
{
    private readonly List<User> _users = new();
    public void AddUser(User user) => _users.Add(user);

    public void SendMessage(string message, User sender)
    {
        foreach (var user in _users.Where(u => u != sender))
            user.Receive(message);
    }
}

public class User
{
    private readonly IChatMediator _mediator;
    public string Name { get; }
    public User(string name, IChatMediator mediator) => (Name, _mediator) = (name, mediator);
    public void Send(string msg) => _mediator.SendMessage(msg, this);
    public void Receive(string msg) => Console.WriteLine($"{Name} received: {msg}");
}
```

---

**Question**: How does the Memento pattern implement snapshot/undo without breaking encapsulation?

**Answer**: The originator creates a Memento containing a snapshot of its internal state. A caretaker stores Mementos but never inspects them — restoring state is done by passing a Memento back to the originator. When to use: when you need undo/checkpoint functionality but don't want to expose internal state (e.g., editors, transaction rollbacks). Pitfall: large Mementos consume memory — use incremental snapshots or compression.

```csharp
public class EditorMemento
{
    internal string Content { get; }
    internal EditorMemento(string content) => Content = content;
}

public class Editor
{
    public string Content { get; set; } = "";

    public EditorMemento Save() => new(Content);
    public void Restore(EditorMemento memento) => Content = memento.Content;
}

public class History
{
    private readonly Stack<EditorMemento> _snapshots = new();
    public void Push(EditorMemento m) => _snapshots.Push(m);
    public EditorMemento Pop() => _snapshots.Pop();
}
```

---

**Question**: How does the Visitor pattern achieve double dispatch?

**Answer**: The Visitor pattern lets you define a new operation on a set of classes without changing them. The visited class accepts a visitor and calls the appropriate `Visit` overload based on its own type (double dispatch). When to use: when you need many unrelated operations across a stable object structure (e.g., AST traversal, exporting to XML/JSON). Pitfall: adding new element types requires updating all visitors — use only when the element hierarchy is stable.

```csharp
public interface IShape
{
    void Accept(IShapeVisitor visitor);
}

public class Circle : IShape
{
    public double Radius { get; set; }
    public void Accept(IShapeVisitor visitor) => visitor.Visit(this);
}

public class Rectangle : IShape
{
    public double Width { get; set; }
    public double Height { get; set; }
    public void Accept(IShapeVisitor visitor) => visitor.Visit(this);
}

public interface IShapeVisitor
{
    void Visit(Circle circle);
    void Visit(Rectangle rectangle);
}

public class AreaCalculator : IShapeVisitor
{
    public double TotalArea { get; private set; }
    public void Visit(Circle c) => TotalArea += Math.PI * c.Radius * c.Radius;
    public void Visit(Rectangle r) => TotalArea += r.Width * r.Height;
}
```

---

**Question**: How does the Active Object pattern decouple method execution from method invocation?

**Answer**: Active Object introduces a Scheduler and a Proxy — the Proxy converts method calls into requests queued on a Scheduler thread, which executes them asynchronously. When to use: when you need to simplify concurrent access to a single-threaded servant (e.g., GUI event queue, game loop). Pitfall: adds complexity — for simpler cases, use `Task.Run` or `Channel<T>` instead.

```csharp
public class ActiveObject<T>
{
    private readonly Channel<Func<T>> _queue = Channel.CreateUnbounded<Func<T>>();

    public ActiveObject(CancellationToken ct)
    {
        Task.Run(async () =>
        {
            await foreach (var work in _queue.Reader.ReadAllAsync(ct))
                work();
        }, ct);
    }

    public Task Enqueue(Func<T> work)
    {
        var tcs = new TaskCompletionSource<T>();
        _queue.Writer.TryWrite(() =>
        {
            try { tcs.SetResult(work()); }
            catch (Exception ex) { tcs.SetException(ex); }
            return tcs.Task;
        });
        return tcs.Task;
    }
}
```

---

**Question**: What is the Monitor Object pattern and how does it ensure thread safety?

**Answer**: A Monitor Object synchronizes all public methods using a lock (or `Monitor` in C#), ensuring only one thread executes at a time while others wait. When to use: when you need thread-safe access to a shared object's state without exposing locking details to callers. Pitfall: coarse locking hurts throughput — consider finer-grained locks or concurrent collections for high-contention scenarios.

```csharp
public class ThreadSafeCounter
{
    private int _count;
    private readonly object _lock = new();

    public int Increment()
    {
        lock (_lock)
        {
            return ++_count;
        }
    }

    public int GetCount()
    {
        lock (_lock)
        {
            return _count;
        }
    }
}
```

---

**Question**: How does a Thread Pool differ from creating individual threads?

**Answer**: A Thread Pool maintains a set of reusable worker threads, avoiding the overhead of thread creation and destruction. Work items are queued and dispatched to available threads. When to use: for short-lived, CPU-bound or I/O-bound tasks (e.g., ASP.NET request handling, parallel data processing). Pitfall: blocking a thread-pool thread long-term starves other tasks — use dedicated threads for long-running operations.

```csharp
// Using .NET ThreadPool (used implicitly by Task.Run)
Task.Run(() => Console.WriteLine("Runs on thread pool"));

// Explicit thread pool
ThreadPool.QueueUserWorkItem(state => Console.WriteLine(state), "Hello");

// Worker thread pattern via BackgroundService
public class WorkerService : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken ct)
    {
        while (!ct.IsCancellationRequested)
        {
            await DoWorkAsync();
            await Task.Delay(1000, ct);
        }
    }
}
```

---

**Question**: How do you implement a Read-Write Lock for shared read, exclusive write access?

**Answer**: `ReaderWriterLockSlim` allows multiple concurrent readers but blocks all readers when a writer holds the lock, ensuring exclusive writes without serializing reads. When to use: when reads vastly outnumber writes and the read operation isn't trivially fast (e.g., configuration cache, lookup tables). Pitfall: ReaderWriterLockSlim is prone to writer starvation if readers are continuous — consider `SemaphoreSlim` for simpler needs.

```csharp
public class ThreadSafeCache<TKey, TValue> where TKey : notnull
{
    private readonly Dictionary<TKey, TValue> _cache = new();
    private readonly ReaderWriterLockSlim _lock = new();

    public TValue? Get(TKey key)
    {
        _lock.EnterReadLock();
        try { return _cache.GetValueOrDefault(key); }
        finally { _lock.ExitReadLock(); }
    }

    public void Set(TKey key, TValue value)
    {
        _lock.EnterWriteLock();
        try { _cache[key] = value; }
        finally { _lock.ExitWriteLock(); }
    }
}
```

---

**Question**: How does the Repository pattern abstract data access?

**Answer**: Repository mediates between domain and data mapping layers, providing a collection-like interface for accessing domain objects. It hides querying, caching, and storage details. When to use: when you want to decouple business logic from persistence concerns and enable unit testing with mock repositories. Pitfall: repositories that expose `IQueryable` leak ORM details — consider returning `IEnumerable` or paginated DTOs instead.

```csharp
public interface IProductRepository
{
    Task<Product?> GetByIdAsync(Guid id);
    Task<IEnumerable<Product>> GetAllAsync();
    Task AddAsync(Product product);
    Task SaveChangesAsync();
}

public class ProductRepository : IProductRepository
{
    private readonly AppDbContext _db;
    public ProductRepository(AppDbContext db) => _db = db;

    public async Task<Product?> GetByIdAsync(Guid id) =>
        await _db.Products.FindAsync(id);

    public async Task<IEnumerable<Product>> GetAllAsync() =>
        await _db.Products.ToListAsync();

    public async Task AddAsync(Product product) =>
        await _db.Products.AddAsync(product);

    public async Task SaveChangesAsync() =>
        await _db.SaveChangesAsync();
}
```

---

**Question**: How does Unit of Work ensure transactional consistency?

**Answer**: Unit of Work tracks changes to objects during a business transaction and flushes all changes in a single database transaction when `Commit()` is called. When to use: when multiple repository operations must succeed or fail together (e.g., transferring funds between accounts). Pitfall: long-running units of work hold database connections and cause stale data — keep them scoped to a single request or use case.

```csharp
public interface IUnitOfWork : IDisposable
{
    IProductRepository Products { get; }
    IOrderRepository Orders { get; }
    Task<int> CommitAsync();
    Task RollbackAsync();
}

// Usage in a service
public async Task PlaceOrderAsync(OrderDto dto)
{
    using var uow = _unitOfWorkFactory.Create();
    var product = await uow.Products.GetByIdAsync(dto.ProductId);
    if (product is null || product.Stock < dto.Quantity)
        throw new InvalidOperationException("Insufficient stock");
    var order = new Order { ProductId = dto.ProductId, Quantity = dto.Quantity };
    await uow.Orders.AddAsync(order);
    await uow.CommitAsync(); // Single transaction
}
```

---

**Question**: How does the Data Mapper pattern differ from Active Record?

**Answer**: Data Mapper keeps domain objects completely unaware of the database — mapping logic lives in separate mapper classes (e.g., Entity Framework, Hibernate). Active Record has domain objects carry persistence logic themselves (e.g., `user.Save()`). When to use: Data Mapper is preferred in complex domains where persistence concerns should not pollute business logic. Pitfall: Data Mapper introduces more boilerplate than Active Record.

```csharp
// Data Mapper — domain objects are POCOs
public class User
{
    public Guid Id { get; set; }
    public string Name { get; set; } = "";
}

public class UserMapper
{
    private readonly DbConnection _conn;

    public async Task<User?> FindByIdAsync(Guid id)
    {
        await using var cmd = _conn.CreateCommand();
        cmd.CommandText = "SELECT Id, Name FROM Users WHERE Id = @id";
        // ... map result to User
    }
}

// Active Record — domain object handles persistence
public class ActiveRecordUser
{
    public Guid Id { get; set; }
    public string Name { get; set; } = "";
    public void Save() { /* INSERT or UPDATE */ }
    public static ActiveRecordUser Find(Guid id) { /* SELECT */ }
}
```

---

**Question**: How does the Service Layer pattern define an application's boundary?

**Answer**: Service Layer defines a set of operations available to clients and coordinates the application's response — it sits between the presentation layer and domain/model layer, encapsulating business logic and transaction management. When to use: when you have multiple clients (web API, background jobs, CLI) that need consistent business behavior. Pitfall: anemic domain models where all logic is pushed into services — keep domain objects behavior-rich.

```csharp
public class OrderService
{
    private readonly IOrderRepository _orders;
    private readonly IUnitOfWork _uow;
    private readonly IPaymentGateway _payment;

    public OrderService(IOrderRepository orders, IUnitOfWork uow, IPaymentGateway payment)
    {
        _orders = orders;
        _uow = uow;
        _payment = payment;
    }

    public async Task<OrderResult> PlaceOrderAsync(CreateOrderCommand cmd)
    {
        var order = Order.Create(cmd.CustomerId, cmd.Items);
        await _orders.AddAsync(order);
        var result = await _payment.ChargeAsync(order.Total);
        if (!result.Success)
            throw new PaymentFailedException(result.ErrorMessage);
        await _uow.CommitAsync();
        return new OrderResult(order.Id);
    }
}
```

---

**Question**: How do you implement the Pipeline pattern for sequential processing stages?

**Answer**: Each stage is a function or handler that receives data, processes it, and passes it to the next stage. Pipelines can be composed as a chain of decorators or as a linear list of steps executed in order. When to use: when you need to apply a sequence of transformation or validation steps (e.g., ETL pipelines, image filters, request validation). Pitfall: error handling becomes complex — consider a shared context object that accumulates errors as the request flows through.

```csharp
public class Pipeline<T>
{
    private readonly List<Func<T, T>> _stages = new();

    public Pipeline<T> AddStage(Func<T, T> stage)
    {
        _stages.Add(stage);
        return this;
    }

    public T Execute(T input)
    {
        var result = input;
        foreach (var stage in _stages)
            result = stage(result);
        return result;
    }
}

// Usage
var pipeline = new Pipeline<string>()
    .AddStage(s => s.Trim())
    .AddStage(s => s.ToUpperInvariant())
    .AddStage(s => s.Replace(" ", "_"));

string result = pipeline.Execute("  hello world  "); // "HELLO_WORLD"
```
