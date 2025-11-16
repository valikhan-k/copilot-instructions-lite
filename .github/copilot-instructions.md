# Guiding Principles

## Philosophy

**Pragmatism over dogma** — Empirical cleanliness beats theoretical purity. Make decisions based on what works in your context, not textbook ideals.

**Code that fits in your head** — Strive for code humans can reason about. If you can't hold the mental model in working memory, simplify. Reduce cognitive load through careful structuring and naming.

**Start simple, evolve deliberately** — Begin with the simplest solution. Add structure only when complexity demands it. Let actual pain points guide evolution. YAGNI (You Aren't Gonna Need It) is wisdom, not laziness.

**Duplication before abstraction** — Prefer duplication over wrong abstraction. Extract patterns only when they emerge organically across multiple locations (3+ is a good rule of thumb). Premature abstraction creates coupling harder to break than duplication.

**Readability is paramount** — Code is read far more than written. Optimize for clarity and maintenance, not cleverness. Make intent explicit.

---

## Security (Non-Negotiable)

- **Validate all external input** — HTTP requests, file uploads, message payloads
- **Use parameterized queries** — Never concatenate user input into SQL

---

## Code Quality Fundamentals

### Naming & Clarity
- Use intention-revealing names that communicate purpose
- Be specific, not generic (`GetOrderById` > `Get`)

### Modern C# Features
- **Embrace modern .NET and C#** — Use latest stable language features and framework capabilities
- Leverage features that improve clarity: records, pattern matching, collection expressions, nullable reference types, primary constructors
- Use `async/await` for I/O operations; never block on async code (`.Result`, `.Wait()`)
- Append `Async` suffix to async method names
- Stay current with language evolution — modern features often provide both clarity and performance benefits

### Method Design
- Keep methods focused and comprehensible (under 50 lines or below cyclomatic complexity of 10 as guideline)
- Reduce nesting (max 3 levels); use early returns and guard clauses
- Prefer pure functions when possible (no side effects) — easier to test and reason about
- Avoid temporal coupling — don't require methods to be called in specific order

---

## Architecture Guidance

Consider following layered architecture with clear separation of concerns:

- The solution must be organized into four main layers:
  - **Domain layer** - core business logic
  - **Application layer** - (use cases, services, commands, queries, interfaces for external services)
  - **Infrastructure layer** - (implementations for external services, database access, third-party integrations)
  - **Api layer** - (controllers, API endpoints, minimal API, request/response models)
  - Each layer should be either in seprate folders or separate projects depending on complexity.
- Tests must be in separate projects:
  - `tests/[Solution].UnitTests` (for Domain and Application)
  - `tests/[Solution].IntegrationTests` (for Infrastructure, Api)

### Dependencies Between Layers

- **Domain**: has no dependencies.
- **Application**: depends only on **Domain**.
- **Infrastructure**: depends on **Application** and **Domain**.
- **Api**: depends on **Application**. (But can reference anything it needs to bootstrap Composition Root)

### Folder and File Structure

- Use a **feature-oriented** folder structure in each layer (e.g., `Order/`, `Customer/`).
- Do **not** use technical names for top-level folders (Controllers, Repositories, etc.).

### Evolutionary Architecture

**Start simple, evolve when needed.** Premature architectural complexity is waste—you're solving problems you don't have yet. Begin with Stage 1 and let actual pain points (not imagined future ones) drive evolution. Each stage introduces overhead: more projects mean more ceremony, vertical slices add coordination complexity, microservices multiply operational challenges. Move to the next stage only when the current structure actively hinders your work. Architecture should emerge from real constraints, not anticipated scale. YAGNI applies to structure as much as code—the best architecture for tomorrow is the simplest one that works today.

#### Stage 1: Single Project with Folder-Based Layers
- All layers (Domain, Application, Infrastructure, Api) as folders within one `.csproj`
- Boundaries manifested through folder structure only
- Use for: new projects, MVPs, small microservices

#### Stage 2: Multi-Project Layer Separation
- Extract each layer into separate `.csproj` files
- Compiler-enforced layer dependencies
- Features remain as folders within each layer project
- Use when: enforcing architectural boundaries becomes important

#### Stage 3: Modular Monolith (Vertical Slices)
- Features become top-level organizational unit (swap places with layers)
- Each feature contains all layers (as folders or projects depending on size)
- Extract shared infrastructure into `Lib/Shared/Common` project
- Features are bounded contexts, communicate through contracts
- Use when: distinct business domains, independent feature evolution needed

#### Stage 4: Microservices (Strangler Pattern)
- Extract features/modules into separate deployable services
- Each service can use Stage 1 or Stage 2 structure internally
- Gradually migrate using strangler pattern: extract → deploy alongside → route traffic → remove from monolith
- Use when: independent deployment and scaling required

**Note**: Stages 1-2 are also appropriate for microservices from the start if service scope is clear.

---

## API Layer Design
- Keep presentation layer thin — delegate to application layer for orchestration and use case logic
- Co-locate related code (Controller or endpoint, DTOs, validators) — feature cohesion over layer cohesion
- Use dependency injection for services and configuration

---

## Testing Approach

**Test behavior, not implementation** — Focus on observable outcomes through public APIs, not internal mechanics. Tests verify **what** the system does, not **how** it does it.

**The refactoring anti-pattern** — Good tests should survive refactoring. If renaming methods or reorganizing classes breaks tests without changing observable behavior, you're coupled to implementation details.

**Outside-in testing:**
- Start with integration tests (API endpoints → database) — broad coverage, coarse-grained assertions
- Add unit tests for application and domain logic — narrow scope, fine-grained assertions
- **Mock only out-of-process dependencies** (HTTP APIs, message brokers, file systems) — things that cross process boundaries
- Use real collaborators for in-process dependencies — avoid mocking domain objects or application services

**Three pillars of good tests:**
1. **Protection against regressions** — Catch bugs when behavior changes
2. **Resistance to refactoring** — Don't break when structure changes (if behavior doesn't)
3. **Fast feedback** — Run quickly to enable rapid iteration

**Test structure (AAA pattern):**
- **Arrange** — Set up test data and dependencies
- **Act** — Execute the behavior under test
- **Assert** — Verify observable outcomes (one logical assertion per test)

**Test naming:** Use plain English that describes the scenario (`Order_with_insufficient_stock_returns_error`). Communicate business intent, not technical implementation.

**Test data:** Start with simple inline creation. Introduce Test Data Builders and Test Fixtures when duplication becomes painful.

**Avoid over-specification** — Assert only what matters for the scenario. Exact method call counts and argument matching create brittle tests. Focus on observable end state.

---

## Common Patterns

### Favor
- **Dependency Injection** — Constructor injection by default; configure at Composition Root
- **Options Pattern** — Strongly-typed configuration
- **Repository Pattern** — Only when abstracting data access provides real value; avoid generic repositories
- **Structured Logging** — Semantic, searchable log messages
- **Pure functions** — No side effects; easier to test, compose, and reason about

### Question (Challenge Before Using)
- **Mediator/CQRS** — Only when complexity justifies the overhead; don't cargo-cult patterns
- **Service Locator** — Anti-pattern; use constructor injection instead
- **Base classes** — Favor composition over inheritance
- **Generic repositories** — Often hide more than they reveal; prefer specific repositories with specific operations
- **Elaborate abstractions** — Every abstraction has a cost; does it earn its keep?
- **Anemic domain models** — If models are just data bags with all logic in services, consider moving behavior into domain

---

## When in Doubt

- Choose the **simpler** solution
- Favor **explicit** over clever
- Prefer **readable** over concise
- Value **working** over theoretically perfect
- **Refer to Microsoft conventions** — When this guide is silent, use [Microsoft Learn](https://learn.microsoft.com/dotnet/) for .NET and C# conventions
- **Ask questions** — clarity beats assumptions

---

## Anti-Patterns to Avoid

**Security & Correctness:**
- Hardcoded secrets and credentials
- SQL injection (always parameterize queries)
- Blocking on async code (`.Result`, `.Wait()`)

**Design:**
- Deep inheritance hierarchies (favor composition)
- Premature optimization (measure first)
- Over-engineering with unnecessary ceremony
- Abstractions before pain points emerge (YAGNI)
- **Temporal coupling** — Methods that must be called in specific order
- **Feature envy** — Methods using more of another class's data than their own
- **Primitive obsession** — Using primitives instead of domain types for business concepts
- **Control Freak** — Classes that micromanage their dependencies (tell, don't ask)

**Testing:**
- Testing private methods or internal state (test through public API)
- **Fragile tests** — Tests that break on refactoring without behavior change
- **Overspecified mocks** — Verifying every interaction instead of observable outcomes
- **Test-induced design damage** — Changing good design just to make testing easier

---

## Bridging Gaps

**When this guide is silent** — Use Microsoft's official documentation as your reference for topics not covered here:

- **[Microsoft Learn (.NET)](https://learn.microsoft.com/dotnet/)** — Official .NET documentation and tutorials
- **[C# Programming Guide](https://learn.microsoft.com/dotnet/csharp/)** — Language conventions and best practices
- **[.NET API Reference](https://learn.microsoft.com/dotnet/api/)** — Official API documentation
- **[ASP.NET Core Fundamentals](https://learn.microsoft.com/aspnet/core/fundamentals/)** — Web development patterns

**Microsoft conventions for:**
- Naming conventions and code formatting
- Framework-specific patterns and APIs
- Language feature usage
- Performance and optimization guidance

This guide takes precedence for architectural philosophy and design principles. Microsoft documentation fills in the implementation details.

---

## Principles Summary

**SOLID Principles (applied pragmatically):**
- **Single Responsibility** — Class has one reason to change
- **Open/Closed** — Open for extension, closed for modification (via abstractions)
- **Liskov Substitution** — Subtypes must be substitutable for base types
- **Interface Segregation** — Many specific interfaces > one general interface
- **Dependency Inversion** — Depend on abstractions, not concretions

**Other Guiding Principles:**
- **Separation of Concerns** — Each module handles one aspect
- **Don't Repeat Yourself (DRY)** — But only after duplication reveals true patterns
- **YAGNI** — You Aren't Gonna Need It (defer until necessary)
- **KISS** — Keep It Simple, Stupid
- **Command-Query Separation** — Methods do or return, not both
- **Tell, Don't Ask** — Objects should make decisions based on their own state
- **Postel's Law** — Be conservative in what you send, liberal in what you accept
