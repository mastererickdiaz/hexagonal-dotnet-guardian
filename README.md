# hexagonal-dotnet-guardian

A Claude Code plugin that validates and enforces **Hexagonal (Ports & Adapters) Architecture** boundaries in .NET projects.

Prevents layer violations, detects wrong-layer dependencies, and guides correct code placement — automatically.

## Components

| Component | Type | Description |
|---|---|---|
| `boundary-awareness` | **Skill** | Auto-detects when you're writing code in the wrong layer and suggests the correct placement |
| `architecture-validator` | **Agent** | Full project scan — analyzes all `.csproj` references and `using` directives for violations |
| `PreToolUse` hook | **Hook** | Blocks file writes that introduce forbidden `using` directives in Domain or Application layers |
| `/check-boundaries` | **Command** | Quick boundary scan with a summary table of all projects and their violation status |

## Installation

### From marketplace

```bash
# Add the marketplace (once)
/plugin marketplace add mastererickdiaz/hexagonal-dotnet-guardian

# Install the plugin
/plugin install hexagonal-dotnet-guardian
```

### From Git directly

```bash
claude plugin install --git https://github.com/mastererickdiaz/hexagonal-dotnet-guardian
```

## Usage

### Automatic (Skill + Hook)

Once installed, the plugin works automatically:

- **Skill:** When you create or edit `.cs` files, Claude is aware of layer boundaries and will warn you if code is misplaced
- **Hook:** If you try to write a file in `Domain/` with `using MongoDB.Driver;`, the hook **blocks the write** and explains why

### Manual

```bash
# Quick boundary scan
/check-boundaries

# Full architecture audit (invokes the agent)
# Ask: "@architecture-validator validate my project architecture"
```

## Rules Enforced

```
Domain (innermost)     → Zero external dependencies
Application            → References only Domain (no infra packages)
Infrastructure         → References only Domain (implements ports)
Presentation           → References Application + Domain via DI
```

### Forbidden imports by layer

| Package | Domain | Application | Infrastructure | Presentation |
|---|---|---|---|---|
| `MongoDB.Driver` | NO | NO | YES | NO |
| `Dapper` | NO | NO | YES | NO |
| `StackExchange.Redis` | NO | NO | YES | NO |
| `Confluent.Kafka` | NO | NO | YES | NO |
| `AWSSDK.*` | NO | NO | YES | NO |
| `System.Net.Http` | NO | NO | YES | NO |

## Example output

```
/check-boundaries

Layer           | Project                  | Violations | Status
----------------|--------------------------|------------|-------
Domain          | Domain                   | 0          | PASS
Application     | Application              | 0          | PASS
Infrastructure  | MongodbInfrastructure     | 0          | PASS
Infrastructure  | RedisInfrastructure       | 0          | PASS
Infrastructure  | SqlServerInfrastructure   | 0          | PASS
Presentation    | SystemAPI                 | 0          | PASS

All clean! No boundary violations found.
```

## License

MIT
