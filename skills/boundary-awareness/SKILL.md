# Boundary Awareness

Automatically detects and prevents Hexagonal Architecture layer violations in .NET projects.

## When to activate

Activate this skill when:
- The user is creating or editing C# files (.cs) in a .NET project
- The user is adding `using` directives or NuGet references
- The user asks to create a new class, interface, service, or repository
- Code is being moved between projects/layers

## Architecture Layers (Inner to Outer)

```
Domain (innermost) → Application → Infrastructure / Presentation (outermost)
```

### Domain Layer
- **Contains:** Entities, Value Objects, Port interfaces (`I*Port`, `I*Repository`, `I*Query`)
- **Allowed dependencies:** None (zero external references)
- **Forbidden:** `using` any Infrastructure, Application, or Presentation namespace. No NuGet packages except pure domain libraries.

### Application Layer
- **Contains:** Use Cases (`*Case.cs`), DTOs/Adapters (`*Adapter.cs`), Validators, Result types
- **Allowed dependencies:** Domain only
- **Forbidden:** `using` any Infrastructure namespace (`*.Infrastructure*`, `MongoDB.*`, `Dapper`, `StackExchange.*`, `Confluent.*`, `Amazon.*`, `System.Net.Http`). No direct database drivers, HTTP clients, or cloud SDKs.

### Infrastructure Layer
- **Contains:** Adapters implementing Domain ports, external service clients
- **Allowed dependencies:** Domain (for port interfaces), external NuGet packages
- **Forbidden:** `using` Application namespace (infrastructure must not know about use cases)

### Presentation Layer (API/Controllers)
- **Contains:** Controllers, Middleware, Startup configuration
- **Allowed dependencies:** Application (for use case interfaces), Domain (for port registration)
- **Forbidden:** Direct `using` of Infrastructure implementation classes in controller logic (only via DI)

## Violation Detection Rules

### Rule 1: Domain must have zero outward dependencies
```csharp
// VIOLATION in Domain project:
using MongoDB.Driver;           // Infrastructure leak
using Application.Adapters;     // Application leak
using System.Net.Http;          // Infrastructure concern

// CORRECT in Domain project:
namespace Domain.Interfaces;
public interface IUserRepository
{
    Task<User> FindByIdAsync(string id, CancellationToken ct = default);
}
```

### Rule 2: Application must not reference Infrastructure
```csharp
// VIOLATION in Application project:
using MongodbInfrastructure.Repositories;  // Direct infra reference
using StackExchange.Redis;                  // Cache driver in use case
using Confluent.Kafka;                      // Messaging driver

// CORRECT in Application project:
using Domain.Interfaces;  // Only reference Domain ports
public class GetUserCase
{
    private readonly IUserRepository _userRepository;  // Port, not adapter
}
```

### Rule 3: Infrastructure must not reference Application
```csharp
// VIOLATION in Infrastructure project:
using Application.UseCases;     // Infra calling use cases
using Application.Adapters;     // Infra using app DTOs

// CORRECT in Infrastructure project:
using Domain.Interfaces;        // Implement domain ports
using Domain.Entities;          // Use domain entities
```

### Rule 4: No circular dependencies
No project may reference another project that directly or transitively references it back.

## Forbidden Package-to-Layer Matrix

| Package / Namespace | Domain | Application | Infrastructure | Presentation |
|---|---|---|---|---|
| `MongoDB.Driver` | NO | NO | YES | NO |
| `Dapper` / `System.Data` | NO | NO | YES | NO |
| `StackExchange.Redis` | NO | NO | YES | NO |
| `Confluent.Kafka` | NO | NO | YES | NO |
| `Amazon.S3` / `AWSSDK.*` | NO | NO | YES | NO |
| `System.Net.Http` | NO | NO | YES | NO |
| `Polly` | NO | NO | YES | NO |
| `FluentValidation` | NO | YES | NO | NO |
| `Microsoft.AspNetCore.*` | NO | NO | NO | YES |

## Response Behavior

When a violation is detected:

1. **Identify** the violation clearly: which file, which `using`, which rule
2. **Explain** why it violates hexagonal boundaries
3. **Suggest** the correct approach (e.g., define a port interface in Domain, implement in Infrastructure)
4. **Show** a corrected code example

When the user asks where to place new code:

1. Identify if it's a domain concept, business logic, external integration, or API concern
2. Recommend the correct layer and project
3. If it needs a port, suggest the interface in Domain and adapter in Infrastructure
