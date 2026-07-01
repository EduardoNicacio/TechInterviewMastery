# Testing & Quality – Interview Questions & Answers

**Question**: What is the test pyramid and when should you use unit, integration, or e2e tests?

**Answer**: The test pyramid recommends many unit tests, fewer integration tests, and even fewer e2e tests. Unit tests verify isolated logic quickly; integration tests validate interactions across boundaries (DB, API); e2e tests confirm full user workflows from UI to backend. Write a unit test for each business rule, an integration test for each data access path, and an e2e test only for critical happy paths.

---

**Question**: How does the "test trophy" (Kent C. Dodds) differ from the traditional test pyramid?

**Answer**: The test trophy replaces the pyramid's strict hierarchy with a focus on integration tests as the bulk of your suite. Kent C. Dodds argues that integration tests provide the best ROI because they test real interactions without the fragility of e2e tests, while static analysis catches type errors and unit tests cover pure logic. The trophy shape has static analysis at the top, then unit, then integration (largest), then e2e.

---

**Question**: How do you evaluate ROI when deciding what tests to write?

**Answer**: The cost of a bug found in production is exponentially higher than catching it in CI, so tests are an investment against that risk. Prioritize tests for critical paths, complex business logic, and frequently changing code where regression risk is highest. Avoid testing trivial code (simple getters, framework configuration) where the maintenance cost outweighs the bug-prevention benefit.

---

**Question**: What is the difference between smoke, sanity, regression, and acceptance tests?

**Answer**: Smoke tests verify basic functionality after a build ("does the app start?"). Sanity tests are a quick subset of regression tests run after minor changes. Regression tests re-verify existing features to catch unintended breaks. Acceptance tests validate the system meets business requirements, often written in business-readable language (Gherkin).

---

**Question**: How do you prioritize tests using risk-based testing?

**Answer**: Classify each feature by business criticality and failure likelihood. High-risk areas (payment processing, auth) get exhaustive coverage; low-risk areas (profile page layout) get minimal coverage. Use a risk matrix tool or even a simple spreadsheet to rank features and allocate test effort accordingly.

---

**Question**: What are the FIRST principles of unit testing?

**Answer**: **F**ast (milliseconds, no I/O), **I**solated (no shared state), **R**epeatable (same result every run), **S**elf-verifying (pass/fail boolean), **T**imely (written before or alongside code). Violating any principle erodes trust in the suite and slows down the feedback loop.

---

**Question**: What is the Arrange-Act-Assert pattern?

**Answer**: Arrange sets up the object under test and its dependencies; Act invokes the method being tested; Assert verifies the outcome against expectations. This structure keeps tests readable and forces a clear separation between setup, invocation, and verification.

```csharp
[Fact]
public void ApplyDiscount_WhenPremiumUser_ReturnsTenPercentOff()
{
    // Arrange
    var calculator = new PriceCalculator();
    var user = new User { IsPremium = true };

    // Act
    var result = calculator.ApplyDiscount(user, 100m);

    // Assert
    Assert.Equal(90m, result);
}
```

---

**Question**: What is the difference between mocks, stubs, fakes, dummies, and spies?

**Answer**: A **dummy** is passed but never used (e.g., null). A **stub** provides canned answers to calls made during the test. A **fake** has a working implementation but with shortcuts (e.g., in-memory database). A **mock** registers calls received and allows verifying interaction expectations. A **spy** wraps a real object and records calls for later verification. Stubs are about state verification; mocks are about behavior verification.

---

**Question**: What are isolated vs sociable unit tests?

**Answer**: Isolated (solitary) tests replace all collaborators with test doubles, verifying the SUT in complete isolation. Sociable tests use real implementations of collaborators, testing the SUT together with its immediate dependencies. Prefer sociable tests when collaborators are fast and deterministic (e.g., value objects); use isolated tests when collaborators involve I/O or nondeterministic behavior.

---

**Question**: What are parameterized/theory tests and why are they useful?

**Answer**: Parameterized tests run the same assertion logic against multiple input-output pairs, reducing duplication and increasing coverage per test method. They make it easy to test edge cases, boundary values, and equivalence partitions without copying and pasting test methods.

```csharp
[Theory]
[InlineData(2, true)]
[InlineData(3, true)]
[InlineData(4, false)]
[InlineData(1, false)]
public void IsPrime_ReturnsCorrectResult(int number, bool expected)
{
    var result = MathService.IsPrime(number);
    Assert.Equal(expected, result);
}
```

```java
@ParameterizedTest
@CsvSource({"2,true", "3,true", "4,false", "1,false"})
void isPrime_ReturnsCorrectResult(int number, boolean expected) {
    assertEquals(expected, MathService.isPrime(number));
}
```

---

**Question**: What should you look for in an assertion library beyond basic equality?

**Answer**: Fluent assertions (e.g., FluentAssertions, AssertJ) improve readability with chainable, natural-language assertions like `result.Should().BePositive().And.BeLessThan(100)`. They provide better failure messages (e.g., "Expected 'foo' to be 'bar' at index 2") and support collection, exception, and timing assertions out of the box.

```csharp
result.Should().Be(42);
list.Should().ContainSingle(x => x > 5);
action.Should().Throw<InvalidOperationException>().WithMessage("*lock*");
```

---

**Question**: How do you test database interactions without slowing down your suite?

**Answer**: Use Testcontainers to spin up lightweight disposable database instances via Docker, ensuring a real database environment without shared state. In-memory databases (e.g., SQLite in-memory, H2) are faster but risk false positives from engine-specific behavior differences. A pragmatic approach: use Testcontainers in CI and in-memory DBs for local development with an environment flag.

```csharp
// Testcontainers for .NET
var container = new PostgreSqlBuilder()
    .WithImage("postgres:16-alpine")
    .Build();
await container.StartAsync();
var connectionString = container.GetConnectionString();
```

```java
// Testcontainers for Java
@Container
PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine");
```

---

**Question**: What is API contract testing and how do Pact or Spring Cloud Contract help?

**Answer**: Contract testing verifies that a provider (API server) and consumer (client) agree on request/response shapes without running end-to-end. Pact follows a consumer-driven approach: the consumer generates a pact file, and the provider replays it to verify compliance. Spring Cloud Contract allows defining contracts in Groovy or YAML and generates both provider-side tests and client stubs.

```typescript
// Pact consumer test (Jest)
it('should return user by ID', async () => {
  await provider.addInteraction({
    state: 'user exists',
    uponReceiving: 'a request for user 42',
    withRequest: { method: 'GET', path: '/users/42' },
    willRespondWith: { status: 200, body: { id: 42, name: 'Alice' } },
  });
  const result = await api.getUser(42);
  expect(result.name).toBe('Alice');
});
```

---

**Question**: How do you use WireMock or MockServer for HTTP integration tests?

**Answer**: These tools run a local HTTP server that stubs external APIs, enabling tests to simulate timeouts, error codes, and specific payloads without hitting real endpoints. They also support request verification, so you can assert that your code sent the correct headers or body.

```java
// WireMock stub
stubFor(get(urlEqualTo("/api/payments"))
    .willReturn(aResponse()
        .withStatus(200)
        .withHeader("Content-Type", "application/json")
        .withBody("{\"status\":\"approved\"}")));
```

---

**Question**: How do you manage test data with fixtures and data factories?

**Answer**: Test fixtures are fixed data sets loaded before tests (e.g., JSON files, SQL scripts). Data factories generate objects with sensible defaults, allowing tests to override only relevant fields for readability and resilience to schema changes. Factories avoid the brittleness of shared static fixtures.

```csharp
public class UserFactory
{
    public static User Create(string? email = null) => new()
    {
        Id = Guid.NewGuid(),
        Email = email ?? "test@example.com",
        IsActive = true
    };
}

// Usage
var user = UserFactory.Create(email: "admin@test.com");
```

---

**Question**: Explain the red-green-refactor cycle of TDD.

**Answer**: **Red**: write a failing test for the desired behavior. **Green**: write the minimal code to make the test pass. **Refactor**: improve the code structure without changing behavior. Each cycle should last seconds to minutes. The discipline forces you to write testable code and prevents untested production logic.

```csharp
// Red – test fails because method doesn't exist yet
Assert.Equal("Fizz", FizzBuzz.Convert(3));

// Green – minimal implementation
public static string Convert(int n) => n % 3 == 0 ? "Fizz" : n.ToString();

// Refactor – extract magic number
private const int FizzDivisor = 3;
public static string Convert(int n) => n % FizzDivisor == 0 ? "Fizz" : n.ToString();
```

---

**Question**: What is the difference between London (mockist) and Detroit (classicist) TDD?

**Answer**: London-school TDD mocks all collaborators and drives design by verifying interactions between objects. Detroit-school TDD tests outcomes (state) with real implementations where possible, using mocks only for unavoidable boundaries (I/O). London promotes finer-grained design but creates brittle tests; Detroit produces fewer, more resilient tests but risks insufficient isolation.

---

**Question**: When does TDD add value and when does it not?

**Answer**: TDD excels for complex business logic, algorithms, and domains where correctness is critical (finance, healthcare). It adds little value for exploratory code (prototypes, data science), simple CRUD with no logic, or when the design is rapidly changing and tests would be constantly rewritten. For UI-heavy or legacy code, characterization tests (write tests after the fact) are more practical.

---

**Question**: What is property-based testing and how do tools like QuickCheck, jqwik, or Hypothesis work?

**Answer**: Instead of example-based tests, property-based testing defines invariants that must hold for a wide range of randomly generated inputs. The framework shrinks failing cases to a minimal reproducer. This catches edge cases you wouldn't think to write by hand.

```java
// jqwik – property-based for Java
@Property
boolean absoluteValueIsNonNegative(@ForAll int anyInt) {
    // Math.abs(Integer.MIN_VALUE) overflows, so guard against it
    return anyInt == Integer.MIN_VALUE || Math.abs(anyInt) > 0;
}
```

```python
# Hypothesis – property-based for Python
@given(st.integers())
def test_absolute_value_is_non_negative(n):
    assert abs(n) >= 0
```

---

**Question**: What is the difference between cyclomatic complexity and cognitive complexity?

**Answer**: Cyclomatic complexity counts linearly independent paths through code (based on branching: if, for, while, case). Cognitive complexity adjusts for how humans read code — a nested `if` adds more weight than a flat one, and structural decisions (like catching exceptions) are counted differently. Cognitive complexity is more useful for code review thresholds (e.g., warn at 15 vs 30).

---

**Question**: How do static analysis tools like SonarQube, CodeQL, and linters differ?

**Answer**: Linters (ESLint, StyleCop) enforce style rules and basic anti-patterns. SonarQube analyzes deeper code smells, duplications, coverage gaps, and computes the overall quality gate. CodeQL (from GitHub) models code as data and runs customizable security queries to find vulnerabilities (e.g., SQL injection, XSS paths). Use all three: linters in pre-commit, SonarQube in CI, CodeQL in security scan jobs.

---

**Question**: What are the pitfalls of using code coverage as a target or gate?

**Answer**: Teams can game the metric by writing shallow tests that exercise code without asserting meaningful behavior. Line coverage alone misses untested branches; even 100% branch coverage doesn't guarantee correct logic. Mutations survive when no test catches the wrong output. Use coverage as a discovery tool (find untested code) and a regression-prevention baseline, not a reward target.

---

**Question**: What is the difference between line, branch, and mutation coverage?

**Answer**: Line coverage measures whether each source line was executed. Branch coverage tracks whether both true/false paths in conditionals were taken. Mutation coverage introduces small code changes (e.g., flipping `>` to `<`) and checks if any test fails — surviving mutants indicate untested logic. Mutation testing is the most rigorous: if no test fails, your suite is blind to that behavior.

```csharp
// Source
if (x > 0) return "positive";

// Mutant
if (x >= 0) return "positive"; // changed > to >=

// Strong test: covers x=0 (not positive) and x=1 (positive) to kill this mutant
```

---

**Question**: What should a code review checklist cover for testing and quality?

**Answer**: **Correctness**: does the code handle edge cases (null, empty, overflow)? **Design**: does it follow SOLID and project patterns? **Testing**: are there unit/integration tests covering the change? **Naming**: do names reveal intent? **Security**: are inputs sanitized? **Performance**: are there obvious N+1 queries or allocations in hot paths? Each reviewer should spend the most time on correctness and design.

---

**Question**: How do you quantify technical debt (SQALE, SonarQube rating)?

**Answer**: SQALE (Software Quality Assessment based on Lifecycle Expectations) estimates the effort in person-hours to fix all code issues. SonarQube rates projects A–E based on remediation cost ratio (effort to fix divided by codebase size). A rating of A means <5% of the code needs fixing. Track the remediation cost trend: a sudden spike indicates debt accumulation from rushed commits.

---

**Question**: What is Gherkin syntax and how does Given-When-Then work?

**Answer**: Gherkin is a business-readable DSL: **Given** sets up preconditions, **When** describes the action, **Then** asserts outcomes. Each scenario is executable through tools like Cucumber or SpecFlow. Scenarios form living documentation that stays synchronized with the code since tests double as specs.

```gherkin
Feature: Order Discount
  Scenario: Premium user receives 10% discount
    Given a premium user named "Alice"
    When she places an order of $100
    Then the total should be $90
```

---

**Question**: How do Cucumber and SpecFlow translate Gherkin to executable tests?

**Answer**: Step definitions (annotated methods) match each Gherkin step to code. Cucumber (Ruby/Java/JS) uses regex or Cucumber expressions; SpecFlow (.NET) uses bindings with attributes like `[Given]`. The framework parses the feature file, matches steps at runtime, and reports pass/fail per scenario, producing HTML reports that stakeholders can read.

```csharp
[Binding]
public class OrderDiscountSteps
{
    [Given(@"a (.*) user named ""(.*)""")]
    public void GivenUser(string tier, string name) { /* ... */ }

    [When(@"she places an order of \$(.*)")]
    public void WhenPlacesOrder(decimal amount) { /* ... */ }

    [Then(@"the total should be \$(.*)")]
    public void ThenTotalShouldBe(decimal expected) { /* ... */ }
}
```

---

**Question**: What is living documentation and how do feature files enable it?

**Answer**: Living documentation is specification that stays in sync with the codebase because the same files power both human reading and automated tests. Each `.feature` file describes business rules in Gherkin, and CI runs them as tests. Tools like Pickles or SpecFlow LivingDoc generate HTML/PDF docs from feature files, so documentation never goes stale.

---

**Question**: How do you conduct load testing with k6, JMeter, or Locust?

**Answer**: Define user scenarios in code (k6 JavaScript, Locust Python) or a GUI (JMeter). Configure the target concurrency (virtual users), ramp-up pattern, and duration. Run against a pre-production environment and measure throughput, latency percentiles (p50, p95, p99), and error rate. Assert thresholds in CI (e.g., p95 < 500ms) to fail the build if performance degrades.

```javascript
// k6 load test
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '30s', target: 50 },
    { duration: '1m', target: 50 },
    { duration: '30s', target: 0 },
  ],
  thresholds: { http_req_duration: ['p(95)<500'] },
};

export default function () {
  const res = http.get('https://api.example.com/health');
  check(res, { 'status 200': (r) => r.status === 200 });
  sleep(1);
}
```

---

**Question**: What is the difference between load testing and stress testing?

**Answer**: Load testing verifies the system handles expected traffic within acceptable latency and error rates. Stress testing pushes the system beyond its breaking point to find the saturation limit (max concurrent users, request throughput) and observe recovery behavior (does it crash gracefully? auto-scale?). Stress testing answers "how much can it take?"; load testing answers "does it meet our SLA?"

---

**Question**: What tools would you use for CPU, memory, and I/O profiling in production?

**Answer**: For .NET, use `dotnet-trace` for CPU profiling, `dotnet-counters` for real-time metrics, and Visual Studio Diagnostic Tools. For Java, use async-profiler (CPU), JFR (memory/GC), and VisualVM. For Node.js, use the built-in `--prof` flag or clinic.js. Profile in staging with production-like traffic; always correlate high CPU with specific code paths via flame graphs.

---

**Question**: How do JMH (Java) and BenchmarkDotNet (.NET) differ for microbenchmarking?

**Answer**: Both handle warmup iterations, measure execution time precisely, and avoid common pitfalls like dead-code elimination. BenchmarkDotNet automates environment diagnostics (CPU info, JIT mode) and generates summary reports. JMH integrates with Maven/Gradle and is more feature-rich for Java (e.g., precise fork/warmup control, state scoping).

```csharp
[SimpleJob(RunStrategy.ColdStart, iterationCount: 5)]
public class StringBenchmarks
{
    [Benchmark]
    public string ConcatWithPlus() => "a" + "b" + "c";

    [Benchmark]
    public string ConcatWithBuilder() => new StringBuilder("a").Append("b").Append("c").ToString();
}
```

```java
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
public class StringBenchmarks {
    @Benchmark
    public String concatWithPlus() { return "a" + "b" + "c"; }

    @Benchmark
    public String concatWithBuilder() {
        return new StringBuilder("a").append("b").append("c").toString();
    }
}
```

---

**Question**: How do you implement test selection and test impact analysis in CI?

**Answer**: Test impact analysis uses a dependency graph to identify which tests cover code changed in a commit. Tools like NCRunch (C#), Infinitest (Java), or `jest --onlyChanged` run only affected tests, dramatically reducing feedback time in large monorepos. For deterministic impact analysis, maintain a mapping of source files to test files (e.g., using code coverage data from the last full run).

---

**Question**: How do you handle flaky tests in CI?

**Answer**: First, quarantine flaky tests by moving them to a separate CI job that doesn't block the build. Automatic retry (e.g., Jest's `--retryTimes` or `[Retry]` in xUnit v3) is a band-aid, not a fix. Root-cause approaches: stabilize async timing with deterministic waits, avoid shared mutable state, pin external service versions, and re-run failed tests with `dotnet test --filter` or `--rerun` and collect diagnostics. Track flaky rate per test and enforce a maximum allowability.

```csharp
// Flaky attribute for temporary quarantine
public class FlakyFactAttribute : FactAttribute
{
    public override string Skip { get; set; } = "Flaky — quarantined";
}
```

---

**Question**: How do you parallelize tests with sharding?

**Answer**: Sharding splits the test suite into N equal buckets, each running on a separate CI job. Test frameworks support built-in sharding: `dotnet test` with test filters and CI matrix strategies, Jest's `--shard=N/M`, and pytest's `pytest-xdist` for parallelization. Combined with test impact analysis, sharding ensures fast feedback even for thousand-test suites.

```yaml
# GitHub Actions matrix sharding
strategy:
  matrix:
    shard: [1, 2, 3]
steps:
  - run: dotnet test --filter "Category=Unit&Shard=${{ matrix.shard }}"
```

---

**Question**: How do you use Testcontainers in CI pipelines?

**Answer**: Testcontainers needs a Docker runtime available in CI (Docker-in-Docker with DinD or socket binding). Each test class can spin up disposable containers (Postgres, Redis, Kafka) via the Testcontainers API, and they auto-clean up after tests. To optimize, use `Ryuk` (resource reaper) for cleanup and shared container per class/module to reduce startup overhead.

```yaml
# GitHub Action with Testcontainers
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: dotnet test
```

---

**Question**: What is the role of mutation testing in evaluating test suite quality?

**Answer**: Mutation testing introduces faults (mutants) into the source code and checks whether the test suite detects them. A high mutation score means your tests actually verify behavior rather than just executing code. Pitest (Java) and Stryker (JS/.NET) are popular tools. Use mutation testing to identify untested branches and missing boundary checks after achieving nominal line coverage targets.

```bash
# Stryker for .NET
dotnet stryker --project "../src/MyProject.csproj"

# Pitest for Java (Maven)
mvn org.pitest:pitest-maven:mutationCoverage
```

---
