# Guiding Principles

## Philosophy

**Pragmatism over dogma** — Empirical cleanliness beats theoretical purity. Make decisions based on what works in your context, not textbook ideals.

**Code that fits in your head** — Strive for code humans can reason about. If you can't hold the mental model in working memory, simplify. Reduce cognitive load through careful structuring and naming.

**Start simple, evolve deliberately** — Begin with the simplest solution. Add structure only when complexity demands it. Let actual pain points guide evolution. YAGNI (You Aren't Gonna Need It) is wisdom, not laziness.

**Duplication before abstraction** — Prefer duplication over wrong abstraction. Extract patterns only when they emerge organically across multiple locations (3+ is a good rule of thumb). Premature abstraction creates coupling harder to break than duplication.

**Readability is paramount** — Code is read far more than written. Optimize for clarity and maintenance, not cleverness. Make intent explicit.

---

## Security (Non-Negotiable)

- **Never hardcode secrets** — Use configuration providers (User Secrets, environment variables, key vaults)
- **Validate all external input** — HTTP requests, file uploads, message payloads
- **Use parameterized queries** — Never concatenate user input into SQL
- **Don't log sensitive data** — No passwords, tokens, PII, or connection strings in logs
- **Authenticate and authorize early** — Validate identity and permissions before processing

---

## Code Quality Fundamentals

### Naming & Clarity
- Use intention-revealing names that communicate purpose
- Avoid abbreviations unless universally understood
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
- **Command-Query Separation** — If feasible, methods should either do something (command) or return something (query), not both. But, stay pragmatic and prefer empirical cleanliness.
- Avoid temporal coupling — don't require methods to be called in specific order

### Error Handling
- **Use Result pattern for expected failures** — Validation, business rule violations (make failure explicit in return type)
- **Reserve exceptions for unexpected errors** — Programming errors, infrastructure failures, exceptional circumstances
- Don't expose internal implementation details in error messages
- Consider functional error handling patterns (Railway Oriented Programming) for composable operations

---

## Architecture Guidance

### Separation of Concerns & Dependency Management
Isolate business logic from infrastructure concerns. Core business rules should have minimal dependencies. This is **Ports and Adapters (Hexagonal Architecture)** — domain at center, infrastructure at edges.

**Dependency Inversion Principle** — High-level modules should not depend on low-level modules. Both depend on abstractions.

**Dependency flow:**
- Domain layer → no external dependencies (pure business logic)
- Application layer → depends on domain, defines abstractions (interfaces/ports)
- Infrastructure → implements application abstractions (adapters)
- API/Presentation → orchestrates, composes dependencies (Composition Root)

**Composition Root** — Wire up dependencies in one place at application entry point. Keep DI configuration centralized, close to `Main`.

### Structural Evolution
Let project structure emerge from evidence, not prediction:

**Default to single .csproj** unless you have concrete reasons to separate:
- Exploring or prototyping
- Total scope fits comfortably in one project
- Boundaries are still fluid

**Start with multiple .csproj when you know:**
- End architecture is predictable from requirements or past experience
- Early separation helps contributors understand the vision (prevents broken windows)
- Clear deployment, team, or domain boundaries exist today

**Evolve the structure** as the codebase reveals its true shape. Balance two risks:
- Creating structure before understanding (premature boundaries)
- Deferring structure past the point of clarity (accumulated debt)

Stay ahead of complexity without over-engineering for imagined futures.

#### Organization Patterns
Choose organization based on what's hardest to navigate, not rigid rules:

**Start with layer folders** (e.g., `/Api`, `/Application`, `/Domain`, `/Infrastructure`):
- Simple mental model for most projects
- Move away when you find yourself jumping between layers constantly

**Add feature-based grouping** when layers get crowded:
- Group related files by feature (e.g., `/Orders` contains order-related files)
- Or nest within layers (e.g., `/Application/Orders`, `/Application/Customers`)
- Use when "where does this belong?" has obvious feature-based answers

**Vertical slice architecture** when features benefit from independence:
- Self-contained slices own their full stack (applicable at any scale)
- Different features change at different rates or by different teams
- Creates natural seams for future extraction (strangler pattern to microservices)
- Choose when slice isolation provides real value, not theoretical modularity

These are examples, not mandates. Let navigation pain and team mental models guide structure. VSA can work beautifully in small projects and provides clean boundaries when it's time to distribute.

### API Design
- Keep endpoints/controllers thin — delegate to services for orchestration
- Co-locate related code (endpoint, DTOs, validators) — feature cohesion over layer cohesion
- Use dependency injection for services and configuration
- **Humble Object pattern** — Keep infrastructure adapters simple, push logic into testable domain/application layers

---

## Testing Approach

**Test behavior, not implementation** — Focus on observable outcomes through public APIs, not internal mechanics. Tests verify **what** the system does, not **how** it does it.

**The refactoring anti-pattern** — Good tests should survive refactoring. If renaming methods or reorganizing classes breaks tests without changing observable behavior, you're coupled to implementation details.

**Outside-in testing:**
- Start with integration tests (API endpoints → database) — broad coverage, coarse-grained assertions
- Add unit tests for complex domain logic — narrow scope, fine-grained assertions
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

---

## Influences & Further Learning

These principles draw from:
- **Mark Seemann** — "Dependency Injection Principles, Practices, and Patterns", "Code That Fits in Your Head"
- **Vladimir Khorikov** — "Unit Testing Principles, Practices, and Patterns"
- **Martin Fowler** — "Refactoring", evolutionary design
- **Kent Beck** — Test-Driven Development, simple design
- **Robert C. Martin** — SOLID principles, Clean Code/Architecture
- **Scott Wlaschin** — Functional domain modeling, Railway Oriented Programming

**Recommended reading:**
- "Code That Fits in Your Head" by Mark Seemann
- "Unit Testing Principles, Practices, and Patterns" by Vladimir Khorikov  
- "Dependency Injection Principles, Practices, and Patterns" by Mark Seemann & Steven van Deursen
- "Domain Modeling Made Functional" by Scott Wlaschin

