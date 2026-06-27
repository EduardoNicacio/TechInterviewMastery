# Technical Debt – Interview Questions & Answers

**Question**: What is the metaphor of technical debt and who popularized it?

**Answer**: Ward Cunningham coined the term to explain that shipping flawed code is like taking out a financial loan - you get speed now but must pay "interest" later through slower development, increased bugs, and higher maintenance costs. Just like financial debt, if never repaid, the compounding interest can eventually stall all progress.

---

**Question**: What is the difference between intentional and unintentional technical debt?

**Answer**: Intentional debt is a conscious trade-off to meet a deadline, knowing you'll refactor later. Unintentional debt accumulates due to ignorance, poor practices, or lack of skill - the team doesn't realize they're creating problems. Intentional debt can be strategic; unintentional debt is always harmful.

---

**Question**: Explain Martin Fowler's technical debt quadrant - prudent vs reckless.

**Answer**: Fowler's quadrant crosses intent (deliberate vs inadvertent) with action quality (prudent vs reckless). Prudent-deliberate: "We'll ship now and fix later." Reckless-deliberate: "We don't have time for design." Prudent-inadvertent: "We learned something new." Reckless-inadvertent: "We didn't know better." The most dangerous is reckless-deliberate.

---

**Question**: What are code smells that commonly indicate technical debt?

**Answer**: Common smells include large classes/methods (God objects), shotgun surgery (one change ripples everywhere), duplicated code, long parameter lists, feature envy, and switch statements over types. These make the codebase harder to modify, increasing the cost of every new feature.

---

**Question**: What is the Boy Scout Rule in software engineering?

**Answer**: The rule states: "Always leave the codebase cleaner than you found it." When touching a file for any reason, make a small improvement - rename a misleading variable, extract a method, add a test. Over time, this compounds into significant quality gains without dedicated refactoring sprints.

---

**Question**: When is a big rewrite preferable over incremental refactoring?

**Answer**: A big rewrite is rarely advisable - it risks losing domain knowledge embedded in the existing code and often takes longer than expected. Prefer incremental refactoring unless the current system is fundamentally unmaintainable (e.g., no tests, monolithic, wrong architecture) and the business can tolerate a long period of dual maintenance.

---

**Question**: Explain the Strangler Fig pattern for migrating legacy systems.

**Answer**: Named after the fig tree that grows around a host, this pattern gradually replaces legacy components by routing specific functionality to new services while keeping the old system running. Over time, more features are moved until the legacy system can be safely decommissioned.

```csharp
// Legacy controller remains - new requests are intercepted and routed
public class OrderController
{
    // Old implementation for orders not yet migrated
    public IActionResult GetOrder(int id)
    {
        throw new NotImplementedException();
    }
}

// New implementation - once coverage is complete, cut over entirely
public class NewOrderController
{
    public async Task<IActionResult> GetOrder(int id)
    {
        var order = await _newOrderService.GetOrderAsync(id);
        return Ok(order);
    }
}
```

---

**Question**: When does technical debt become critical?

**Answer**: It becomes critical when the cost of adding a single feature exceeds the value that feature delivers, or when the team spends more time fixing bugs than building new functionality. Another threshold: onboarding a new developer takes weeks instead of days because the codebase is incomprehensible.

---

**Question**: How do you measure technical debt quantitatively?

**Answer**: Tools like SonarQube compute a "Debt Ratio" by dividing estimated remediation time by development time. Key metrics include cyclomatic complexity (methods over 10–15 are suspects), code coverage (below 60–70% is risky), code duplication percentage, and the number of code smells per 1,000 lines.

---

**Question**: How should technical debt be managed on a product roadmap?

**Answer**: Treat high-interest debt as a first-class work item - size it, estimate its ROI, and prioritize it alongside features. Use a "debt backlog" with clear labels (e.g., "performance debt," "testing debt") and allocate a fixed percentage of each sprint to reducing it so it never gets forgotten.

---

**Question**: What percentage of sprint capacity should be allocated to reducing technical debt?

**Answer**: Industry consensus suggests 20–30% of each sprint for debt reduction and refactoring. This prevents debt from accumulating while still delivering features. Some teams use the "Rule of Thirds": one-third new features, one-third improvements/debt, one-third bug fixes and operations.

---

**Question**: What is the cost of delay when ignoring technical debt?

**Answer**: The cost grows exponentially - every deferred refactor compounds future changes because new features must work around increasingly tangled code. This is the "debt trap": teams slow down, deadlines slip, more shortcuts are taken, and the cycle repeats. A fix that takes 1 hour today could take 1 week six months later.

---

**Question**: What is testing debt and how does it manifest?

**Answer**: Testing debt is the absence of adequate unit, integration, or end-to-end tests, forcing manual regression testing for every change. It manifests as fear of refactoring, production bugs in unchanged code, and long manual QA cycles. Teams should enforce coverage gates in CI to prevent further accumulation.

```csharp
// Untested code - every change risks regression
public class InvoiceCalculator
{
    public decimal CalculateTotal(IEnumerable<LineItem> items)
    {
        decimal total = 0;
        foreach (var item in items)
            total += item.Quantity * item.UnitPrice; // No discount logic - and no tests
        return total;
    }
}

// With tests, refactoring is safe
[Fact]
public void CalculateTotal_sums_all_items()
{
    var items = new[] { new LineItem { Quantity = 100, UnitPrice = 10 } };
    var result = new InvoiceCalculator().CalculateTotal(items);
    Assert.Equal(1000, result); // matches implementation
}
```

---

**Question**: What is documentation debt and when should it concern you?

**Answer**: Documentation debt occurs when APIs, architectural decisions, or setup steps lack written records, forcing developers to reverse-engineer the system. It becomes critical when knowledge exists only in one person's head (bus-factor risk). However, code should be self-documenting via clear naming and structure - documentation debt is secondary to code debt.

---

**Question**: What is architecture debt and how does it differ from code-level debt?

**Answer**: Architecture debt involves wrong or outdated structural decisions at the system level - incorrect abstractions, inappropriate coupling between modules, or a mismatched pattern (e.g., using microservices when a monolith suffices). Unlike code-level debt (messy methods), architecture debt requires significant rework across multiple components.

---

**Question**: What is dependency debt and how do you manage it?

**Answer**: Dependency debt arises from using outdated libraries or frameworks with known vulnerabilities, breaking API changes, or abandoned packages. Manage it with automated dependency scanning (Dependabot, Renovate), regular upgrade cycles (every 2–3 minor versions), and a policy to never let a dependency fall more than one major version behind.

---

**Question**: What is build and deployment debt?

**Answer**: This includes slow CI pipelines (30+ minutes), manual deployment steps, unreliable scripts, and lack of infrastructure-as-code. It manifests as "works on my machine" issues and failed releases. Invest in reproducible builds, automated pipeline optimizations (caching, parallel stages), and fully automated deployments.

---

**Question**: What is performance debt?

**Answer**: Performance debt is the accumulation of unoptimized code - N+1 database queries, missing indexes, no caching strategy, inefficient algorithms, and large payload serialization. It degrades user experience silently and often only surfaces under load. Address it with profiling tools and set performance budgets (e.g., "API must respond under 200ms at p95").

```csharp
// Performance debt: N+1 queries
var orders = await _context.Orders.ToListAsync();
foreach (var order in orders)
{
    var customer = await _context.Customers.FindAsync(order.CustomerId); // N queries!
}

// Fixed: eager loading
var orders = await _context.Orders
    .Include(o => o.Customer)
    .ToListAsync();
```

---

**Question**: What is security debt?

**Answer**: Security debt is the accumulation of unaddressed vulnerabilities - unpatched dependencies, missing input validation, weak authentication, exposed secrets, and insufficient logging/auditing. Unlike other debt, security debt carries immediate business risk (breaches, compliance fines). Treat critical and high-severity vulnerabilities as P0 incidents, not backlog items.

---

**Question**: How does technical debt manifest in microservices architectures?

**Answer**: Common forms include tight service coupling (synchronous calls where async should be used), shared databases that break bounded contexts, inconsistent API versioning, lack of observability (distributed tracing, centralized logging), and duplicated logic across services. Each of these makes independent deployability - the core benefit of microservices - impossible.

---

**Question**: What strategies exist for migrating a legacy database?

**Answer**: The expand-migrate-contract pattern (aka Parallel Run) works well: add new tables alongside old ones, dual-write to both, backfill historical data, validate consistency, then cut over reads. For schema changes, use versioned migrations (Flyway, EF Core Migrations) and never mutate production data without a rollback plan.

```sql
-- Phase 1: Expand - add new column alongside old
ALTER TABLE Orders ADD TotalAmount decimal(18,2);
-- Phase 2: Migrate - dual-write and backfill
UPDATE Orders SET TotalAmount = Subtotal + Tax + Shipping;
-- Phase 3: Contract - drop old column after verification
ALTER TABLE Orders DROP COLUMN LegacyTotal;
```

---

**Question**: How do feature flags enable safe refactoring of legacy code?

**Answer**: Feature flags wrap new code paths behind a toggle so you can deploy refactored logic to production without affecting users. If the new implementation fails, flip the flag to instantly revert to the old path. This enables continuous delivery of refactoring work without long-lived branches or risky cutovers.

```csharp
public class PaymentProcessor
{
    private readonly IFeatureFlagService _flags;

    public async Task<PaymentResult> ProcessPayment(PaymentRequest request)
    {
        if (_flags.IsEnabled("NewPaymentGateway"))
        {
            return await NewPaymentFlow(request); // Refactored path
        }
        return await LegacyPaymentFlow(request); // Existing path
    }
}
```

---

**Question**: How do coding standards and linting prevent technical debt?

**Answer**: Automated linting (ESLint, StyleCop, ruff) and formatting enforce consistent code style, catch common anti-patterns, and prevent debt from being introduced in the first place. Enforce them as pre-commit hooks and CI gates so that any PR violating standards is automatically rejected, shifting quality left.

---

**Question**: How do code reviews act as technical debt prevention?

**Answer**: Reviews catch design flaws, missing tests, and unclear logic before they merge into the main branch - preventing debt at the point of introduction. They also spread domain knowledge across the team, reducing bus-factor risk. A good review checklist includes: "Does this change introduce any new debt?" or "Is the code at least as clean as it was before?"

---

**Question**: How does automated testing serve as a safety net for refactoring?

**Answer**: A comprehensive test suite (unit + integration + contract tests) lets developers refactor with confidence, knowing that any behavioral change will be caught by a failing test. Without this safety net, teams avoid touching legacy code, letting debt rot. Aim for high coverage in core business logic and enforce it with coverage gates.

```csharp
// Refactoring this method is safe because tests protect behavior
public decimal CalculateDiscount(decimal amount, CustomerTier tier)
{
    // Original: convoluted if-else chain
    // Refactored: strategy pattern or lookup table
    return _discountStrategyProvider.GetStrategy(tier).Apply(amount);
}
```

---

**Question**: What is collective code ownership and how does it reduce technical debt?

**Answer**: Collective ownership means any team member can modify any part of the codebase, not just "their" module. This prevents knowledge silos, encourages consistent standards, and ensures that refactoring isn't blocked by a single gatekeeper. It pairs well with code reviews and strong automated testing.

---

**Question**: How should you onboard a new developer to a codebase with high technical debt?

**Answer**: Pair the new developer with a senior who documents the worst debt areas, the "why" behind awkward code, and the team's refactoring strategy. Give them small, low-risk debt-reduction tasks (rename variables, add tests) to build familiarity without frustration. Maintain an "architectural decision log" explaining past trade-offs.

---

**Question**: How do you calculate the ROI of paying down technical debt?

**Answer**: Estimate the "interest rate" - time lost per feature due to the debt (e.g., 20% slower development). Compare that to the cost of refactoring (engineering hours). If a refactor costing 40 hours saves 10 hours per sprint indefinitely, the payback period is 4 sprints with infinite ROI thereafter. Tools like SonarQube provide estimated remediation cost vs. time saved.

---

**Question**: What is the difference between technical debt and architectural runway in SAFe?

**Answer**: Technical debt is unplanned, accumulated suboptimal code that slows future work. Architectural runway in SAFe is the existing set of infrastructure, platform components, APIs, and patterns that supports upcoming features. Building runway is proactive Enabler work; addressing technical debt is reactive remediation. They are complementary but distinct concepts.

---

**Question**: How do you communicate the need to address technical debt to non-technical stakeholders?

**Answer**: Use financial metaphors they understand: "Every feature now takes 30% longer because of shortcuts made last year. Investing two weeks to clean this up will recover that lost velocity." Translate technical debt into business metrics: slower time-to-market, higher bug counts, increased operational risk. Avoid jargon - talk about cost, schedule, and quality.

---

**Question**: What is the difference between refactoring and rewriting, and when is each appropriate?

**Answer**: Refactoring changes internal structure without altering external behavior, done in small safe steps. Rewriting starts from scratch. Refactoring is almost always safer; rewrites should be reserved for cases where the codebase is too brittle to refactor (zero tests, no modularity, unsafe to deploy) or the technology is fundamentally obsolete.

---

**Question**: How can you quantify the "interest" being paid on technical debt in a sprint?

**Answer**: Track metrics like "time spent debugging legacy code," "failed deployments due to fragile tests," and "context-switching overhead from spaghetti dependencies." If a team spends 40% of each sprint on unplanned work caused by existing debt, that's the interest rate. Use retrospective trend data to measure improvement after refactoring.

---

**Question**: What is the role of static analysis in managing technical debt?

**Answer**: Static analysis tools (SonarQube, Roslyn Analyzers, Coverity) automatically detect code smells, security vulnerabilities, and maintainability issues. Integrate them into CI to enforce quality gates - fail builds when debt ratio exceeds a threshold or when new code introduces smells. This prevents new debt from entering the codebase.

---

**Question**: How do you handle technical debt introduced by third-party library upgrades?

**Answer**: Treat library upgrades as first-class work items with testing and migration plans. Use semantic versioning awareness and automated dependency updates (Dependabot, Renovate). When a library changes its public API, wrap it behind an abstraction so future migrations affect only the adapter code, not the whole codebase.

```csharp
// Adapter pattern isolates third-party dependency debt
public interface IPaymentGateway
{
    Task<PaymentResult> ChargeAsync(decimal amount, string token);
}

public class StripePaymentGateway : IPaymentGateway
{
    public async Task<PaymentResult> ChargeAsync(decimal amount, string token)
    {
        // Stripe-specific SDK calls - changes are isolated here
        var charge = await _stripeClient.Charges.CreateAsync(new ChargeCreateOptions
        {
            Amount = (long)(amount * 100),
            Source = token,
            Currency = "usd"
        });
        return new PaymentResult { Success = charge.Status == "succeeded" };
    }
}
```

---

**Question**: What is "broken windows theory" as applied to software?

**Answer**: The theory says that visible signs of disorder (messy code, commented-out blocks, inconsistent formatting) encourage further decay - if a window is broken and left unrepaired, soon all windows will be broken. In code, a single unmaintained module signals that quality isn't valued, so new contributions skip standards. Fix broken windows immediately.

---

**Question**: How do you decide which technical debt to pay down first?

**Answer**: Prioritize by risk and frequency of touch: debt in heavily modified code paths with high bug rates should be addressed first. Use the "hot-spot analysis" - files that change most often and have the highest complexity are prime candidates. Also prioritize security debt (P0) and debt blocking a specific feature before general cleanup.
