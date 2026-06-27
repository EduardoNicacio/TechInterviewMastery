# Architecture – Interview Questions & Answers

**Question**: What are the layers in Clean Architecture and what is the dependency rule?

**Answer**: Clean Architecture consists of four layers: Enterprise Business Rules (Entities), Application Business Rules (Use Cases), Interface Adapters (Controllers, Gateways), and Frameworks & Drivers (DB, UI, Web). The dependency rule states that source code dependencies can only point inward - outer layers depend on inner layers, never the reverse.

```csharp
// Inner layer – Enterprise Business Rules
public class Order
{
    public decimal Total { get; private set; }
    public void ApplyDiscount(decimal percentage) => Total -= Total * (percentage / 100m);
}

// Outer layer – Interface Adapters
public class OrderController
{
    private readonly IPlaceOrderUseCase _useCase;
    public OrderController(IPlaceOrderUseCase useCase) => _useCase = useCase;
}
```

---

**Question**: How does Vertical Slicing differ from Layered Architecture?

**Answer**: Layered Architecture groups code by technical concern (controllers, services, repositories), while Vertical Slicing groups by feature (place-order, cancel-order). Vertical Slicing reduces coupling across features, makes it easier to add or modify one feature without affecting others, and aligns better with Domain-Driven Design.

```
// Layered
Controllers/OrderController.cs
Services/OrderService.cs
Repositories/OrderRepository.cs

// Vertical Slices
Features/PlaceOrder/PlaceOrderController.cs
Features/PlaceOrder/PlaceOrderHandler.cs
Features/PlaceOrder/PlaceOrderValidator.cs
Features/CancelOrder/CancelOrderController.cs
```

---

**Question**: What are the key microservices architecture patterns?

**Answer**: Key patterns include API Gateway for unified entry points, Service Discovery for dynamic routing, Circuit Breaker for fault tolerance, Saga for distributed transactions, Event Sourcing for audit trails, CQRS for read/write separation, and Strangler Fig for incremental migration from monoliths.

```yaml
# docker-compose excerpt showing service discovery pattern
services:
  service-a:
    image: service-a:latest
    environment:
      - CONSUL_HTTP_ADDR=consul:8500
  consul:
    image: consul:latest
    ports:
      - "8500:8500"
```

---

**Question**: Explain the difference between Entities, Value Objects, Aggregates, and Repositories in DDD.

**Answer**: Entities have identity and mutable state (e.g., `User` with `UserId`). Value Objects are immutable and defined by their attributes (e.g., `Address`). Aggregates are clusters of entities and value objects treated as a single unit with a root entity (e.g., `Order` as aggregate root containing `OrderLine` items). Repositories provide collection-like access to aggregates for retrieval and persistence.

```csharp
// Entity
public class User
{
    public Guid Id { get; private set; }
    public string Name { get; set; }
}

// Value Object
public record Address(string Street, string City, string ZipCode);

// Aggregate Root
public class Order
{
    public Guid OrderId { get; private set; }
    private List<OrderLine> _lines = new();
    public IReadOnlyList<OrderLine> Lines => _lines.AsReadOnly();
    public void AddLine(OrderLine line) => _lines.Add(line);
}

// Repository
public interface IOrderRepository
{
    Task<Order?> GetByIdAsync(Guid orderId);
    Task SaveAsync(Order order);
}
```

---

**Question**: What is CQRS and when should you use it?

**Answer**: CQRS (Command Query Responsibility Segregation) separates read operations (queries) from write operations (commands), often using separate models and data stores. Use it when read and write workloads differ significantly, such as read-heavy dashboards with complex queries, or when different security boundaries are needed for reads vs writes.

```csharp
// Command – write model
public record PlaceOrderCommand(Guid CustomerId, List<OrderLineDto> Items);

public class PlaceOrderHandler : IRequestHandler<PlaceOrderCommand, Guid>
{
    public async Task<Guid> Handle(PlaceOrderCommand request, CancellationToken ct)
    {
        var order = new Order(request.CustomerId);
        // ... apply domain logic
        return order.Id;
    }
}

// Query – read model (denormalized)
public record GetOrderSummaryQuery(Guid OrderId);

public class GetOrderSummaryHandler : IRequestHandler<GetOrderSummaryQuery, OrderSummaryDto>
{
    public async Task<OrderSummaryDto> Handle(GetOrderSummaryQuery request, CancellationToken ct)
    {
        // Direct SQL or read-optimized store
        return await _db.QueryAsync<OrderSummaryDto>("SELECT * FROM OrderSummaries WHERE Id = @OrderId", request);
    }
}
```

---

**Question**: What is Event Sourcing and how does it differ from traditional persistence?

**Answer**: Event Sourcing stores state changes as an append-only sequence of events rather than the current state. To get the current state, you replay all events. This provides a complete audit trail, enables temporal queries, and supports rebuilding projections, but increases storage and replay complexity compared to CRUD.

```csharp
// Event interface for filtering
public interface IAggregateEvent
{
    Guid AggregateId { get; }
}

// Concrete events
public record OrderCreated(Guid AggregateId, Guid CustomerId, DateTime OccurredAt) : IAggregateEvent;
public record OrderShipped(Guid AggregateId, DateTime ShippedAt) : IAggregateEvent;

// Repository appends events, never updates
public class EventStore
{
    private readonly List<IAggregateEvent> _events = new();
    public void Append(IAggregateEvent @event) => _events.Add(@event);
    public IEnumerable<IAggregateEvent> GetEvents(Guid aggregateId)
        => _events.Where(e => e.AggregateId == aggregateId);
}
```

---

**Question**: Compare choreography vs orchestration in the Saga Pattern.

**Answer**: In choreography, each service publishes events and reacts to others' events - no central coordinator exists, which reduces coupling but makes failure handling harder to trace. In orchestration, a central coordinator (orchestrator) tells each service what to do and handles compensations explicitly, adding a single point of control and failure.

```csharp
// Orchestration Saga with a coordinator
public class OrderOrchestrator
{
    public async Task ProcessOrderAsync(PlaceOrderCommand cmd)
    {
        try
        {
            await _inventory.ReserveAsync(cmd.Items);
            await _payment.ChargeAsync(cmd.CustomerId, cmd.Total);
            await _shipping.ScheduleAsync(cmd.OrderId);
            await _order.MarkConfirmedAsync(cmd.OrderId);
        }
        catch (Exception)
        {
            await _inventory.ReleaseAsync(cmd.Items);   // Compensate
            await _payment.RefundAsync(cmd.CustomerId); // Compensate
            await _order.MarkFailedAsync(cmd.OrderId);
        }
    }
}
```

---

**Question**: Explain the Strangler Fig Pattern.

**Answer**: It incrementally replaces a monolithic system by gradually building a new microservices-based system alongside it. A routing layer intercepts calls and directs them to either the old monolith or new services. Over time, features are migrated until the monolith can be decommissioned.

```csharp
// Routing middleware that gradually migrates traffic
public class StranglerMiddleware
{
    public async Task InvokeAsync(HttpContext context)
    {
        if (_featureToggles.UseNewCheckout(context.Request.Path))
        {
            var requestMessage = new HttpRequestMessage
            {
                Method = new HttpMethod(context.Request.Method),
                RequestUri = new Uri($"https://new-checkout-service{context.Request.Path}{context.Request.QueryString}")
            };
            await _httpClient.SendAsync(requestMessage);
        }
        else
        {
            await _next(context); // Forward to old monolith
        }
    }
}
```

---

**Question**: What is the Circuit Breaker Pattern and when would you use it?

**Answer**: The Circuit Breaker prevents an application from repeatedly calling a failing remote service. It has three states: Closed (normal operation), Open (fail fast after threshold), and Half-Open (allow test requests). Use it to prevent cascading failures, give downstream services time to recover, and degrade gracefully.

```csharp
public class CircuitBreaker
{
    private int _failureCount;
    private readonly int _threshold = 5;
    private readonly TimeSpan _openToHalfOpenTimeout = TimeSpan.FromSeconds(30);
    private CircuitState _state = CircuitState.Closed;
    private DateTime _openedAt;

    public async Task<T> ExecuteAsync<T>(Func<Task<T>> action)
    {
        if (_state == CircuitState.Open)
        {
            if (DateTime.UtcNow - _openedAt >= _openToHalfOpenTimeout)
                _state = CircuitState.HalfOpen;
            else
                throw new CircuitBreakerOpenException("Service unavailable");
        }

        try
        {
            var result = await action();
            _failureCount = 0;
            if (_state == CircuitState.HalfOpen) _state = CircuitState.Closed;
            return result;
        }
        catch
        {
            _failureCount++;
            if (_failureCount >= _threshold)
            {
                _state = CircuitState.Open;
                _openedAt = DateTime.UtcNow;
            }
            throw;
        }
    }
}
```

---

**Question**: How does an API Gateway differ from a Backend for Frontend (BFF)?

**Answer**: An API Gateway is a single entry point for all clients that handles routing, authentication, rate limiting, and aggregation. A BFF is a dedicated backend per client type (mobile, web, IoT) that tailors APIs specifically to each client's needs, avoiding the problem of a one-size-fits-all gateway that forces mobile and web to share the same payload.

```
                   ┌──────────────┐
Client A ─────────►│              │
Client B ─────────►│  API Gateway │──► Services
Client C ─────────►│              │
                   └──────────────┘

                   ┌──────────┐
Mobile App ────────►│ BFF-Mobile │──► Services
                   └──────────┘
                   ┌────────┐
Web Browser ───────►│ BFF-Web │──► Services
                   └────────┘
```

---

**Question**: What is a Service Mesh and what problems does it solve?

**Answer**: A Service Mesh is a dedicated infrastructure layer (e.g., Istio, Linkerd) that handles inter-service communication via sidecar proxies. It provides observability (metrics, tracing), security (mTLS encryption), and reliability (retries, circuit breaking) without modifying application code, offloading these concerns from developers.

```yaml
# Kubernetes annotation to inject sidecar proxy
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    sidecar.istio.io/inject: "true"
spec:
  template:
    spec:
      containers:
        - name: service
          image: my-service:latest
```

---

**Question**: What is Event-Driven Architecture and what are its benefits?

**Answer**: Event-Driven Architecture uses asynchronous events to communicate between decoupled services. Producers emit events (e.g., `OrderPlaced`) to an event broker, and consumers react independently. Benefits include loose coupling, scalability, resilience (consumers can fail without affecting producers), and real-time processing capabilities.

```csharp
// Producer
public class OrderService
{
    private readonly IEventBus _eventBus;
    public async Task PlaceOrderAsync(Order order)
    {
        await _repository.SaveAsync(order);
        await _eventBus.PublishAsync(new OrderPlacedEvent(order.Id, order.CustomerId));
    }
}

// Consumer (in a different service)
public class NotificationService : IIntegrationEventHandler<OrderPlacedEvent>
{
    public async Task HandleAsync(OrderPlacedEvent @event)
    {
        await _emailService.SendOrderConfirmationAsync(@event.CustomerId);
    }
}
```

---

**Question**: What is the difference between a Message Queue and an Event Bus?

**Answer**: A Message Queue (e.g., RabbitMQ, SQS) uses point-to-point delivery - one message is consumed by one consumer, supporting work queues and competing consumers. An Event Bus (e.g., Kafka, Event Grid) uses pub/sub - one event can be consumed by multiple subscribers. Event Buses typically support longer retention and replay, while message queues emphasize guaranteed delivery and load leveling.

---

**Question**: Explain the CAP Theorem.

**Answer**: CAP Theorem states a distributed data store can only provide two of three guarantees: Consistency (every read receives the most recent write), Availability (every request gets a non-error response), and Partition Tolerance (system continues despite network splits). In practice, partitions are inevitable, so designers choose between CP (consistency) and AP (availability).

---

**Question**: Compare ACID and BASE.

**Answer**: ACID (Atomicity, Consistency, Isolation, Durability) guarantees strong consistency and is used in relational databases for transactions. BASE (Basically Available, Soft state, Eventual consistency) relaxes consistency for availability and partition tolerance, common in NoSQL systems. ACID is suitable for financial systems; BASE for social feeds or analytics where immediate consistency isn't critical.

---

**Question**: When would you use Two-Phase Commit vs a Saga?

**Answer**: Two-Phase Commit (2PC) provides strong atomicity across resources but requires a coordinator and locks resources, making it unsuitable for long-running transactions across microservices. Sagas break a transaction into compensatable steps with rollback logic, trading strong consistency for eventual consistency and higher availability across distributed services.

```
2PC:             Saga (Orchestration):
Phase 1: Prepare ├── Reserve Inventory
  A: Prepare OK  ├── Charge Payment (fails here → compensate)
  B: Prepare OK  ├── Compensate: Release Inventory
Phase 2: Commit  └── Saga completes with failure
  A: Commit OK
  B: Commit OK
```

---

**Question**: Compare Database per Service vs Shared Database.

**Answer**: Database per Service gives each microservice its own database, enforcing bounded contexts and independent schema evolution, but adds complexity for cross-service queries. Shared Database provides simpler joins and transactions but creates tight coupling - a schema change in one service can break others, undermining microservice independence.

---

**Question**: What is Polyglot Persistence?

**Answer**: Polyglot Persistence means using different database technologies for different use cases within the same system. For example, PostgreSQL for transactional data, Elasticsearch for full-text search, Redis for caching, and Neo4j for graph relationships. Each service chooses the storage engine best suited to its data access patterns.

```yaml
# Microservice-specific databases
services:
  orders:
    image: orders:latest
    depends_on:
      - postgres-orders
  catalog:
    image: catalog:latest
    depends_on:
      - mongo-catalog
  search:
    image: search:latest
    depends_on:
      - elasticsearch
  cart:
    image: cart:latest
    depends_on:
      - redis-cart
```

---

**Question**: Why is idempotency important in distributed systems?

**Answer**: Idempotency ensures that processing the same request multiple times produces the same result, preventing duplicate side effects like double charges. It's critical in messaging systems where at-least-once delivery can cause retries. Idempotency keys or version fields allow servers to safely detect and ignore duplicates.

```csharp
public class PaymentService
{
    private readonly HashSet<Guid> _processedIds = new();

    public async Task<PaymentResult> ProcessPaymentAsync(Guid idempotencyKey, decimal amount)
    {
        if (_processedIds.Contains(idempotencyKey))
            return PaymentResult.Duplicate; // Already processed

        await _gateway.ChargeAsync(amount);
        _processedIds.Add(idempotencyKey);
        return PaymentResult.Success;
    }
}
```

---

**Question**: What is Distributed Tracing and how does it work?

**Answer**: Distributed Tracing tracks a single request as it flows across multiple services by propagating a trace ID and span IDs in request headers (e.g., W3C Trace-Context). Tools like Jaeger, Zipkin, or OpenTelemetry collect span data to build a timeline of the request, helping identify latency bottlenecks and error sources.

```csharp
// Using OpenTelemetry automatic instrumentation
// Trace ID propagates across HTTP calls via headers
services.AddOpenTelemetry()
    .WithTracing(tracer => tracer
        .AddAspNetCoreInstrumentation()
        .AddHttpClientInstrumentation()
        .AddJaegerExporter());

// You can create custom spans
using var span = tracer.StartActiveSpan("ProcessOrder");
span.SetAttribute("order.id", orderId);
// ... work ...
```

---

**Question**: What is the difference between health checks and readiness probes?

**Answer**: Liveness health checks indicate whether the application is running (if it's dead, restart it). Readiness probes indicate whether the application can handle traffic (if not ready, remove from load balancer). Startup probes check if the application has finished initializing. Separating these allows graceful startup and shutdown without disrupting traffic.

```csharp
// ASP.NET Core health checks
builder.Services.AddHealthChecks()
    .AddCheck("liveness", _ => HealthCheckResult.Healthy(), tags: new[] { "live" })
    .AddDbContextCheck<AppDbContext>("readiness", tags: new[] { "ready" });

app.MapHealthChecks("/health/live", new() { Predicate = r => r.Tags.Contains("live") });
app.MapHealthChecks("/health/ready", new() { Predicate = r => r.Tags.Contains("ready") });
```

```yaml
# Kubernetes probes
livenessProbe:
  httpGet:
    path: /health/live
    port: 8080
readinessProbe:
  httpGet:
    path: /health/ready
    port: 8080
```

---

**Question**: What is graceful degradation?

**Answer**: Graceful degradation means a system continues to provide partial functionality when a dependency fails, rather than failing completely. For example, showing cached product recommendations when the recommendation service is down, or serving a simpler UI without personalization. This improves user experience and system resilience.

```csharp
public class ProductPageService
{
    public async Task<ProductPageViewModel> GetProductPageAsync(int productId)
    {
        var product = await _productRepo.GetByIdAsync(productId);
        List<Recommendation> recs;

        try
        {
            recs = await _recommendationService.GetAsync(productId);
        }
        catch (Exception)
        {
            recs = new List<Recommendation>(); // Graceful degradation: empty recommendations
        }

        return new ProductPageViewModel(product, recs);
    }
}
```

---

**Question**: Explain the Bulkhead Pattern.

**Answer**: The Bulkhead Pattern isolates resources into separate pools so that a failure in one pool doesn't bring down the entire system. Named after ship bulkheads that prevent flooding from spreading, it's typically implemented with separate thread pools, connection pools, or process boundaries per service or consumer.

```csharp
// Separate thread pools for different consumer types
public class BulkheadExecutor
{
    private readonly SemaphoreSlim _paymentSemaphore = new(5);
    private readonly SemaphoreSlim _notificationSemaphore = new(10);

    public async Task ExecutePaymentAsync(Func<Task> action)
    {
        await _paymentSemaphore.WaitAsync();
        try { await action(); }
        finally { _paymentSemaphore.Release(); }
    }

    public async Task ExecuteNotificationAsync(Func<Task> action)
    {
        await _notificationSemaphore.WaitAsync();
        try { await action(); }
        finally { _notificationSemaphore.Release(); }
    }
}
```

---

**Question**: What is the difference between rate limiting and throttling?

**Answer**: Rate limiting restricts the number of requests a client can make within a time window (e.g., 100 req/min). Throttling applies dynamic backpressure by reducing throughput based on server capacity, often by returning 429 Too Many Requests or queuing requests. Rate limiting is usually fixed; throttling adapts to system load.

```csharp
// Fixed window rate limiter middleware
public class RateLimitingMiddleware
{
    private readonly ConcurrentDictionary<string, (int Count, DateTime WindowStart)> _windows = new();
    private readonly int _limit = 100;
    private readonly TimeSpan _windowDuration = TimeSpan.FromMinutes(1);

    public async Task InvokeAsync(HttpContext context)
    {
        var clientIp = context.Connection.RemoteIpAddress.ToString();
        var now = DateTime.UtcNow;

        var window = _windows.AddOrUpdate(clientIp,
            _ => (1, now),
            (_, existing) =>
            {
                if (now - existing.WindowStart >= _windowDuration)
                    return (1, now);
                return (existing.Count + 1, existing.WindowStart);
            });

        if (window.Count > _limit)
        {
            context.Response.StatusCode = 429;
            await context.Response.WriteAsync("Rate limit exceeded");
            return;
        }

        await _next(context);
    }
}
```

---

**Question**: What is the Cache-Aside Pattern?

**Answer**: The application checks the cache first. On a cache hit, it returns data directly. On a miss, it loads data from the database, stores it in the cache, and returns it. This keeps the cache populated lazily and avoids stale data issues when used with appropriate expiration policies. It's the most common caching pattern.

```csharp
public class ProductService
{
    public async Task<Product?> GetProductAsync(int id)
    {
        // Check cache first
        var cached = await _cache.GetAsync<Product>($"product:{id}");
        if (cached is not null) return cached;

        // Cache miss - load from DB
        var product = await _db.Products.FindAsync(id);
        if (product is null) return null;

        // Populate cache
        await _cache.SetAsync($"product:{id}", product, TimeSpan.FromMinutes(10));
        return product;
    }
}
```

---

**Question**: Compare Write-Through and Write-Behind Cache strategies.

**Answer**: Write-Through writes to both cache and database synchronously, ensuring consistency but adding latency on every write. Write-Behind (Write-Back) writes to cache immediately and asynchronously persists to the database in batches, offering lower write latency but risking data loss if the cache fails before flushing.

```csharp
// Write-Through
public async Task WriteThroughAsync(string key, Product value)
{
    await _cache.SetAsync(key, value);
    await _db.Products.UpsertAsync(value); // Synchronous DB write
}

// Write-Behind
public async Task WriteBehindAsync(string key, Product value)
{
    await _cache.SetAsync(key, value);
    _writeQueue.Enqueue((key, value)); // Background worker flushes to DB
}
```

---

**Question**: What are CDN strategies for static and dynamic content?

**Answer**: For static content (assets, images), CDNs cache at edge locations with long TTLs using versioned filenames. For dynamic content, edge caching can be used with Cache-Control headers (private for user-specific, public for shared). Strategies include cache invalidation via purge APIs, origin shielding to reduce load on the origin server, and geo-distribution.

```http
# Static asset with long cache
GET /static/js/app.a1b2c3.js
Cache-Control: public, max-age=31536000, immutable

# Dynamic API response with short cache
GET /api/products/popular
Cache-Control: public, max-age=300, s-maxage=600
```

---

**Question**: When should you design stateless vs stateful services?

**Answer**: Stateless services don't store session data between requests, making them horizontally scalable, resilient to failures, and simpler to deploy. Stateful services maintain in-memory state (e.g., WebSocket connections, shopping carts) and require session affinity, distributed caches, or external state stores. Prefer stateless by default; use stateful only when necessary.

```csharp
// Stateless - any instance can handle any request
public class StatelessPaymentService
{
    public async Task<PaymentResult> ProcessAsync(PaymentRequest request)
    {
        return await _paymentGateway.ChargeAsync(request);
    }
}

// Stateful - uses session affinity
public class SessionCartService
{
    public async Task AddToCartAsync(string sessionId, CartItem item)
    {
        var cart = await _sessionStore.GetAsync<Cart>(sessionId);
        cart.Items.Add(item);
        await _sessionStore.SetAsync(sessionId, cart);
    }
}
```

---

**Question**: What are the Twelve-Factor App methodology principles?

**Answer**: The Twelve-Factor App is a methodology for building SaaS applications: 1) Codebase - one codebase tracked in version control, 2) Dependencies - explicitly declare and isolate, 3) Config - store config in environment variables, 4) Backing Services - treat as attached resources, 5) Build, Release, Run - strictly separate stages, 6) Processes - execute as stateless processes, 7) Port Binding - export services via port binding, 8) Concurrency - scale out via process model, 9) Disposability - fast startup and graceful shutdown, 10) Dev/Prod Parity - keep environments similar, 11) Logs - treat as event streams, 12) Admin Processes - run as one-off processes.

```yaml
# Twelve-Factor compliant configuration
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    spec:
      containers:
        - env:
            - name: DATABASE_URL  # Factor 3: Config in env
              value: "postgres://..."
            - name: REDIS_URL
              value: "redis://..."
```

---

**Question**: What is Conway's Law and how does it affect architecture?

**Answer**: Conway's Law states that organizations design systems that mirror their communication structures. If a company has three teams (frontend, backend, database), the resulting system will likely have three corresponding layers. This means architecture should align with team boundaries - cross-team communication overhead drives the need for well-defined service contracts and bounded contexts.

---

**Question**: Compare Modular Monolith vs Microservices.

**Answer**: A Modular Monolith is a single deployment unit with well-defined module boundaries, offering simpler deployment, testing, and data consistency while still enforcing separation of concerns. Microservices deploy each module independently, providing independent scaling and team autonomy but adding network overhead, distributed transaction complexity, and operational burden. Start with a modular monolith; extract microservices when needed.

```csharp
// Modular Monolith - one solution with separate project modules
// Modules/Orders/OrdersModule.cs
public class OrdersModule : IModule
{
    public IServiceCollection RegisterModules(IServiceCollection services)
    {
        services.AddScoped<IOrderRepository, OrderRepository>();
        return services;
    }
}

// Modules/Billing/BillingModule.cs
public class BillingModule : IModule
{
    // Separate module with its own database schema
}
```

---

**Question**: What is a Bounded Context in DDD?

**Answer**: A Bounded Context is a logical boundary within which a particular domain model applies consistently. Each context has its own ubiquitous language, entities, and invariants. For example, "Product" in the Inventory context has stock levels, while in the Catalog context it has descriptions and prices. Contexts communicate through mappings, not shared models.

```csharp
// Bounded Context: Inventory
namespace Inventory
{
    public class Product
    {
        public int StockLevel { get; private set; }
        public void AdjustStock(int quantity) => StockLevel += quantity;
    }
}

// Bounded Context: Catalog
namespace Catalog
{
    public class Product
    {
        public string Name { get; set; }
        public decimal Price { get; set; }
        public string Description { get; set; }
    }
}
```

---

**Question**: What is Ubiquitous Language in DDD?

**Answer**: Ubiquitous Language is a shared, rigorous language used by developers, domain experts, and stakeholders throughout the project - in code, conversations, documentation, and tests. It ensures that the code's model accurately reflects business concepts. For example, if the business says "Reserve Stock," the code should use `reserveStock`, not `inventoryLock`.

---

**Question**: What is Event Storming?

**Answer**: Event Storming is a collaborative workshop technique for exploring complex business domains. Participants use sticky notes on a wall: orange for domain events (past tense), blue for commands, green for aggregates, red for external systems, and pink for read models. The process reveals bounded contexts, aggregates, and business rules through group discussion and visualization.

```
[OrderPlaced] ──→ [ReserveStock] ──→ [StockReserved] ──→ [ChargePayment]
  (Event)           (Command)            (Event)            (Command)
```

---

**Question**: What is Hexagonal Architecture (Ports and Adapters)?

**Answer**: Hexagonal Architecture places the core domain logic in the center, surrounded by ports (interfaces) and adapters (implementations). Ports define use cases or repository interfaces; adapters connect to external systems (databases, HTTP, message queues). This isolates domain logic from infrastructure concerns, making it testable without external dependencies.

```csharp
// Core port
public interface IOrderRepository
{
    Task<Order?> GetByIdAsync(Guid id);
    Task SaveAsync(Order order);
}

// Infrastructure adapter
public class PostgresOrderRepository : IOrderRepository
{
    private readonly AppDbContext _db;
    public async Task<Order?> GetByIdAsync(Guid id)
        => await _db.Orders.FindAsync(id);
}
```

---

**Question**: What is Onion Architecture?

**Answer**: Onion Architecture inverts dependencies toward the domain core, similar to Clean Architecture. It uses concentric layers: Domain Model (innermost), Domain Services, Application Services, Infrastructure (outermost). Each layer can only depend on layers more central. It emphasizes dependency injection and interfaces to keep the domain pure and framework-agnostic.

```
Onion Layers (innermost to outermost):
1. Domain Model - Entities, Value Objects
2. Domain Services - Business logic
3. Application Services - Use cases, DTOs
4. Infrastructure - DB, File System, APIs
5. UI / Tests - Outer ring
```

---

**Question**: How do SOLID principles apply to software architecture?

**Answer**: SOLID principles guide system-level design. Single Responsibility ensures each module has one reason to change. Open/Closed allows adding features via extension without modifying existing code. Liskov Substitution ensures service contracts are interchangeable. Interface Segregation prevents fat service interfaces. Dependency Inversion keeps high-level policies independent of low-level details.

```csharp
// Dependency Inversion at architectural level
// High-level policy depends on abstraction, not concrete implementation
public class PlaceOrderUseCase
{
    private readonly IOrderRepository _repo;     // Abstraction in core layer
    private readonly IPaymentGateway _payment;    // Abstraction in core layer

    public PlaceOrderUseCase(IOrderRepository repo, IPaymentGateway payment)
    {
        _repo = repo;
        _payment = payment;
    }
}
```

---

**Question**: How does Dependency Injection differ from Inversion of Control?

**Answer**: Inversion of Control (IoC) is a broad principle where the framework controls the flow of the program rather than the application code. Dependency Injection (DI) is a specific technique to achieve IoC: dependencies are provided (injected) into a class from the outside rather than created internally. IoC is the "what," DI is the "how."

```csharp
// Without DI - class creates its own dependencies
public class OrderService
{
    private readonly IOrderRepository _repo = new PostgresOrderRepository();
}

// With DI - dependencies injected
public class OrderService
{
    private readonly IOrderRepository _repo;
    public OrderService(IOrderRepository repo) => _repo = repo;
}
```

---

**Question**: What is the Repository Pattern and what problem does it solve?

**Answer**: The Repository Pattern abstracts data access logic behind an interface, making the domain model unaware of the persistence mechanism. It centralizes query logic, simplifies unit testing, and allows swapping storage backends without changing business code. It's commonly used with Domain-Driven Design to provide collection-like access to aggregates.

```csharp
public interface IProductRepository
{
    Task<Product?> GetByIdAsync(int id);
    Task<IEnumerable<Product>> GetByCategoryAsync(string category);
    Task AddAsync(Product product);
    void Update(Product product);
    Task DeleteAsync(int id);
}

public class ProductRepository : IProductRepository
{
    private readonly AppDbContext _db;
    public async Task<Product?> GetByIdAsync(int id)
        => await _db.Products.FindAsync(id);
}
```

---

**Question**: What is the Unit of Work Pattern?

**Answer**: The Unit of Work pattern maintains a list of objects affected by a business transaction and coordinates the writing out of changes. It ensures all changes are committed together (atomicity) or rolled back. In Entity Framework, `DbContext` is the Unit of Work - `SaveChangesAsync()` persists all tracked changes in a single transaction.

```csharp
public class OrderService
{
    private readonly IUnitOfWork _uow;
    private readonly IOrderRepository _orders;
    private readonly ICustomerRepository _customers;

    public async Task PlaceOrderAsync(PlaceOrderCommand cmd)
    {
        var customer = await _customers.GetByIdAsync(cmd.CustomerId);
        var order = new Order(customer.Id);
        _orders.Add(order);
        customer.IncrementOrderCount();

        await _uow.SaveChangesAsync(); // One transaction for all changes
    }
}
```

---

**Question**: When would you use the Factory Pattern in architecture design?

**Answer**: The Factory Pattern encapsulates object creation logic, useful when creation involves complex decisions, configuration, or polymorphic types. In architecture, it's commonly used to create different service implementations based on runtime configuration (e.g., creating a `StripePaymentGateway` vs `PayPalPaymentGateway` based on region).

```csharp
public interface IPaymentGatewayFactory
{
    IPaymentGateway Create(PaymentMethod method);
}

public class PaymentGatewayFactory : IPaymentGatewayFactory
{
    public IPaymentGateway Create(PaymentMethod method) => method switch
    {
        PaymentMethod.CreditCard => new StripeGateway(),
        PaymentMethod.PayPal => new PayPalGateway(),
        PaymentMethod.Crypto => new CoinbaseGateway(),
        _ => throw new ArgumentException($"Unknown method: {method}")
    };
}
```

---

**Question**: How does the Strategy Pattern improve architectural flexibility?

**Answer**: The Strategy Pattern defines a family of interchangeable algorithms encapsulated behind a common interface. It allows selecting algorithms at runtime without changing the client code. In architecture, it's used for tax calculation, pricing rules, authentication providers, or data export formats - each strategy can be developed and tested independently.

```csharp
public interface ITaxCalculationStrategy
{
    decimal CalculateTax(Order order);
}

public class UsTaxStrategy : ITaxCalculationStrategy
{
    public decimal CalculateTax(Order order) => order.Total * 0.08m;
}

public class EuTaxStrategy : ITaxCalculationStrategy
{
    public decimal CalculateTax(Order order) => order.Total * 0.20m;
}

// Client
public class CheckoutService
{
    private readonly ITaxCalculationStrategy _taxStrategy;
    public CheckoutService(ITaxCalculationStrategy taxStrategy) => _taxStrategy = taxStrategy;
}
```

---

**Question**: What is the Observer Pattern and how is it used in distributed systems?

**Answer**: The Observer Pattern defines a one-to-many dependency where when one object changes state, all dependents are notified automatically. In distributed systems, it's implemented via event buses, message queues, or pub/sub systems - services emit events and multiple consumers react independently without coupling to the producer.

```csharp
// In-process observer pattern (event-based)
public class OrderService
{
    public event EventHandler<OrderPlacedEventArgs>? OrderPlaced;

    public async Task PlaceOrderAsync(Order order)
    {
        // ... business logic ...
        OrderPlaced?.Invoke(this, new OrderPlacedEventArgs(order.Id));
    }
}

// Distributed observer (via message broker)
// Same concept, but across service boundaries using Kafka/RabbitMQ
```

---

**Question**: How does the Decorator Pattern apply to cross-cutting concerns?

**Answer**: The Decorator Pattern wraps an object with additional behavior without modifying its interface. In architecture, it's ideal for cross-cutting concerns like logging, caching, validation, or retry logic around service operations. Multiple decorators can be stacked, enabling composable middleware.

```csharp
// Base component
public interface IOrderService
{
    Task<Order> GetOrderAsync(Guid id);
}

// Decorator for caching
public class CachedOrderService : IOrderService
{
    private readonly IOrderService _inner;
    private readonly ICache _cache;

    public async Task<Order> GetOrderAsync(Guid id)
    {
        return await _cache.GetOrCreateAsync($"order:{id}", () => _inner.GetOrderAsync(id));
    }
}

// Decorator for logging
public class LoggingOrderService : IOrderService
{
    private readonly IOrderService _inner;
    private readonly ILogger _logger;

    public async Task<Order> GetOrderAsync(Guid id)
    {
        _logger.LogInformation("Fetching order {Id}", id);
        return await _inner.GetOrderAsync(id);
    }
}
```

---

**Question**: What is the Proxy Pattern and when is it used in architecture?

**Answer**: The Proxy Pattern provides a surrogate or placeholder for another object to control access to it. In distributed architecture, proxies handle lazy initialization, access control, remote communication (e.g., gRPC proxy), or caching. API Gateways and reverse proxies (e.g., Nginx, Envoy) are architectural-level implementations of this pattern.

```csharp
// Virtual proxy - lazy loading of expensive resource
public class LazyCustomerRepositoryProxy : ICustomerRepository
{
    private ICustomerRepository? _realRepo;
    private readonly Func<ICustomerRepository> _factory;

    public LazyCustomerRepositoryProxy(Func<ICustomerRepository> factory) => _factory = factory;

    public async Task<Customer?> GetByIdAsync(Guid id)
    {
        _realRepo ??= _factory(); // Lazily initialize real repository
        return await _realRepo.GetByIdAsync(id);
    }
}
```

---

**Question**: What are Feature Toggles and how should they be managed?

**Answer**: Feature Toggles (feature flags) allow enabling or disabling features at runtime without code deploys. They enable trunk-based development, canary releases, and A/B testing. They should be short-lived for release toggles but can persist for operational or permission toggles. Use centralized configuration services and clean up stale toggles to avoid technical debt.

```csharp
public class CheckoutFeature
{
    private readonly IFeatureManager _featureManager;

    public async Task<IActionResult> CheckoutAsync(CheckoutRequest request)
    {
        if (await _featureManager.IsEnabledAsync("NewCheckoutFlow"))
        {
            return await _newCheckoutHandler.HandleAsync(request);
        }
        return await _legacyCheckoutHandler.HandleAsync(request);
    }
}
```

```json
{
  "FeatureManagement": {
    "NewCheckoutFlow": true,
    "RecommendationEngine": {
      "EnabledFor": [
        { "Name": "Microsoft.Percentage", "Parameters": { "Value": 10 } }
      ]
    }
  }
}
```

---

**Question**: What is Blue-Green Deployment?

**Answer**: Blue-Green Deployment runs two identical production environments (Blue and Green). At any time, only one serves live traffic. When deploying a new version, it goes to the idle environment (e.g., Green), where it's tested. After verification, the router switches traffic from Blue to Green. This enables zero-downtime deployments and instant rollback by switching back.

```yaml
# Kubernetes service switching between blue and green
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp
    version: green  # Switch between blue/green here
  ports:
    - port: 80
```

---

**Question**: What are Canary Releases?

**Answer**: Canary Releases gradually route a small percentage of traffic to a new version while serving most traffic on the old version. If the canary shows no errors or performance degradation, traffic is incrementally increased. This reduces blast radius and provides real-world validation before full rollout. It's more sophisticated than blue-green for risk-sensitive deployments.

```yaml
# Istio VirtualService for canary routing
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
spec:
  hosts:
    - myapp
  http:
    - route:
        - destination:
            host: myapp
            subset: stable
          weight: 90
        - destination:
            host: myapp
            subset: canary
          weight: 10
```

---

**Question**: What are the three pillars of observability?

**Answer**: The three pillars are Logs (structured, searchable records of discrete events), Metrics (aggregated numerical data over time - latency, error rate, throughput), and Traces (end-to-end request paths across distributed services). Together they provide a complete picture: metrics alert you to problems, logs help you debug, and traces show you the path through services.

```csharp
// OpenTelemetry configuration covering all three pillars
services.AddOpenTelemetry()
    .WithMetrics(metrics => metrics
        .AddAspNetCoreInstrumentation()
        .AddRuntimeInstrumentation()
        .AddPrometheusExporter())
    .WithTracing(tracing => tracing
        .AddAspNetCoreInstrumentation()
        .AddHttpClientInstrumentation()
        .AddZipkinExporter());
// Structured logging via Serilog
Log.Logger = new LoggerConfiguration()
    .WriteTo.Console()
    .WriteTo.Seq("http://seq:5341")
    .CreateLogger();
```

---

**Question**: Explain SLA, SLO, and SLI.

**Answer**: SLI (Service Level Indicator) is a specific metric like request latency at p99. SLO (Service Level Objective) is a target value for that SLI, e.g., 99.9% of requests under 200ms. SLA (Service Level Agreement) is a contractual commitment to a SLO, with penalties for breaches. SLIs measure what we actually get; SLOs define what we aim for; SLAs formalize what we promise.

```
SLA: 99.9% uptime (contractual, with penalty)
SLO: 99.95% uptime (internal target, stricter than SLA)
SLI: Actual uptime = 99.97% over trailing 30 days
```
