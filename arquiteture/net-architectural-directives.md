# ARCHITECTURE DIRECTIVES AND CODE STANDARDS (CLEAN ARCHITECTURE)

## Mode & Scope
- Surgery Mode: ENABLED (surgical changes in minimal code snippets).
- Prose: DISABLED (direct responses, no introductions, no politeness, only minimal technical explanations if necessary).
- Token Economy: MAXIMIZED.

---

## Directives

### 1. General Constraints
- **Framework Target:** .NET 10 via `dotnet CLI` (`dotnet new sln`, `dotnet new classlib`, etc.).
- **License Locking:** Mandatory `Mediator` version locked to MIT (< 2.0.0). No commercial Mediator versions allowed.
- **Vogen Config:** Generate static abstracts globally in `AssemblyInfo.cs` for Vogen structs:
  `[assembly: VogenDefaults(staticAbstractsGeneration: StaticAbstractsGeneration.MostCommon | StaticAbstractsGeneration.InstanceMethodsAndProperties)]`

---

### 2. Solution & Project Structure (Vertical Slice / Ardalis Clean Architecture)
- **Projects:**
  - `Core`: Aggregates, Entities, Value Objects, Domain Events, Specs, Domain Services, Core Interfaces. Zero external IO dependencies.
  - `UseCases`: Commands, Queries, Handlers, DTOs, Query Interfaces. Depends on `Core`.
  - `Infrastructure`: EF Core, DbContext, Configurations, Repositories, External Services, Concrete Query Services. Depends on `UseCases`.
  - `Web`: FastEndpoints, Swagger/Scalar, Dependency Injection, Middleware, Extensions. Depends on `Infrastructure` and `UseCases`.
  - `AspireHost` / `ServiceDefaults`: Distributed application orchestration and OpenTelemetry defaults.

---

### 3. Domain Model (Core)
- **Strongly-Typed IDs / Value Objects:**
  - Create using `Vogen` struct types.
  - Required validation inside Vogen definitions via `private static Validation Validate(...)`.
- **Entities & Aggregates:**
  - Inherit from `EntityBase<TEntity, TId>` and implement `IAggregateRoot` on the Root.
  - Encapsulate collections (`private readonly List<T> _items`, expose `IReadOnlyList<T>`).
  - Modify state exclusively via domain methods (e.g., `UpdateName`, `AddItem`).
- **Domain Events:**
  - Inherit from `DomainEventBase`.
  - Register inside Domain Entities using `this.RegisterDomainEvent(...)`.
  - Event Handlers implement `INotificationHandler<TEvent>` and live in `Core` or `UseCases`.

---

### 4. Application Logic (Use Cases & CQRS)
- **Framework:** Source-Generated AOT Mediator (`Mediator` package).
- **Commands & Queries:**
  - Commands: `public record MyCommand(...) : ICommand<Result<TResponse>>;`
  - Queries: `public record MyQuery(...) : IQuery<Result<TResponse>>;`
- **Handlers:**
  - Implement `ICommandHandler<TCommand, Result<TResponse>>` or `IQueryHandler<TQuery, Result<TResponse>>`.
  - Handlers must consume `IRepository<T>` (Writes) or direct Query Services / `IReadRepository<T>` (Reads).
- **Result Pattern:**
  - Standardize method returns with `Ardalis.Result<T>`.
  - Use `Result.Success()`, `Result.NotFound()`, `Result.Invalid(validationErrors)`.

---

### 5. Data Access & Persistence (Infrastructure)
- **Specifications:**
  - Complex queries/includes inside `Core` inheriting from `Specification<T>`.
- **Repository:**
  - Consume generic `IRepository<T>` and `IReadRepository<T>` backed by `Ardalis.Specification.EntityFrameworkCore`.
- **EF Core Configurations:**
  - Implement `IEntityTypeConfiguration<T>` in `Infrastructure/Data/Config`.
  - Value Object Conversions: Add `[EfCoreConverter<TId>]` in a central `VogenEfCoreConverters.cs` file.
  - Dispatch Domain Events post-save via `SaveChangesInterceptor` (`EventDispatchInterceptor`).
- **Read Operations (CQRS Query Services):**
  - Read-heavy queries bypass domain specifications: define interface in `UseCases`, implement optimized SQL/EF project projection in `Infrastructure`.

---

### 6. Web API Endpoints (Web)
- **Framework:** `FastEndpoints` (No Controllers, No Minimal API lambdas).
- **Endpoint Design (REPR Pattern):**
  - Structure folders by feature (e.g., `CartFeatures/AddToCart/`).
  - Class extends `FastEndpoints.Endpoint<TRequest, Results<Ok<TResponse>, NotFound, ...>, TMapper>`.
  - Implement `Configure()`: Define route, HTTP verb, `AllowAnonymous()`, `Summary()`, `Tags()`.
  - Implement `ExecuteAsync()`: Send command/query via `IMediator` and return typed results (`TypedResults.Ok()`, `TypedResults.NotFound()`).
- **Validation:**
  - Implement `FastEndpoints.Validator<TRequest>` using `FluentValidation`.
- **Mappings & DTOs:**
  - Implement `FastEndpoints.Mapper<TRequest, TResponse, TDto>` for clean endpoint-to-domain transformations.
- **Global Result Extension:**
  - Map `Ardalis.Result` to `TypedResults` using custom extension methods (`ToGetByIdResult`, `ToCreatedResult`, `ToDeleteResult`).
