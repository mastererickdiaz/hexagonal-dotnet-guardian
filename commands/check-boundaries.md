---
description: Scan the .NET solution for Hexagonal Architecture boundary violations
allowed-tools: Glob, Grep, Read, Agent
---

# Check Architecture Boundaries

Perform a quick architecture boundary scan of this .NET solution.

## Steps

1. **Find all .csproj files** using Glob to identify solution projects.

2. **Classify each project** by layer:
   - `Domain` → Domain layer
   - `Application` → Application layer
   - `*Infrastructure` → Infrastructure layer
   - The main API project (contains `Controllers/`) → Presentation layer
   - `*Tests` → Skip (test projects are exempt)

3. **Check project references** in each `.csproj`:
   - Domain must reference NO other projects
   - Application must reference ONLY Domain
   - Infrastructure must reference ONLY Domain
   - Presentation can reference Application and Domain

4. **Scan for forbidden `using` directives**:
   - In `Domain/`: search for `using.*Infrastructure`, `using MongoDB`, `using Dapper`, `using StackExchange`, `using Confluent`, `using Amazon`, `using System.Net.Http`
   - In `Application/`: search for `using.*Infrastructure`, `using MongoDB`, `using Dapper`, `using StackExchange`, `using Confluent`, `using Amazon`, `using System.Net.Http`, `using Polly`
   - In `*Infrastructure/`: search for `using.*Application\.`, `using.*Case;`

5. **Output a summary table**:

```
Layer           | Project                  | Violations | Status
----------------|--------------------------|------------|-------
Domain          | Domain                   | 0          | PASS
Application     | Application              | 0          | PASS
Infrastructure  | MongodbInfrastructure     | 0          | PASS
...
```

6. **For each violation found**, output:
   - File path and line number
   - The offending `using` or `<ProjectReference>`
   - Which rule it violates
   - Suggested fix

Be concise. If no violations are found, congratulate the team and suggest any optional improvements.
