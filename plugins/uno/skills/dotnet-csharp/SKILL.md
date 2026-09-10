---
name: dotnet-csharp
description: ".NET C# best practices for building performant, secure, and maintainable applications. Use when: (1) Writing new .NET/C# code, (2) Reviewing code for performance issues, (3) Identifying security vulnerabilities, (4) Refactoring existing applications, (5) Architecting new systems, (6) Debugging production issues. Do NOT use for: XAML-only best practices (see winui-xaml), Uno Platform project setup (see uno-platform-agent), or service-layer extensions (see uno-extensions-services)."
license: "CC-BY-4.0 (patterns derived from Microsoft Learn documentation)"
metadata:
  version: "1.0.0"
---

# .NET C# Best Practices

Best practices for writing performant, secure, and maintainable .NET applications, covering the most impactful patterns that prevent production issues.

## Quick Reference - Top 5 Rules

1. **Prevent N+1 queries** - Use `Include()` for eager loading, never lazy-load in loops
2. **Eliminate async waterfalls** - Use `Task.WhenAll` for independent operations
3. **Never store secrets in code** - Use User Secrets locally, Key Vault in production
4. **Dispose resources properly** - Use `using` statements and `IHttpClientFactory`
5. **Use proper HTTP status codes** - Match status to outcome, document with `[ProducesResponseType]`

## Categories by Impact

| # | Category | Impact | Key Topics |
|---|----------|--------|------------|
| 1 | [Database Access](references/01-database-access-critical.md) | CRITICAL | N+1 queries, eager loading, AsNoTracking, split queries |
| 2 | [Async & Parallelism](references/02-async-parallelism-critical.md) | CRITICAL | Task.WhenAll, async waterfalls, ConfigureAwait |
| 3 | [Security & Identity](references/03-security-identity-critical.md) | CRITICAL | Secrets management, Key Vault, User Secrets |
| 4 | [API Design](references/04-api-design-high.md) | HIGH | Status codes, ProducesResponseType, CreatedAtAction |
| 5 | [Memory & Resources](references/05-memory-resources-high.md) | HIGH | IDisposable, IHttpClientFactory, using statements |

## Impact Levels

- **CRITICAL**: Can cause production outages, security breaches, or order-of-magnitude performance degradation
- **HIGH**: Noticeable performance degradation, resource exhaustion, or API contract violations
- **MEDIUM-HIGH**: Maintainability and testability issues that compound over time
- **MEDIUM**: Operational visibility gaps that slow incident response

## Pattern Structure

Each reference file follows this format:

```markdown
### Rule Name
**Impact**: Level (measurable description)
**Incorrect**: Anti-pattern with // ❌ BAD comment
**Correct**: Best practice with // ✅ GOOD comment
**Additional Context**: When to apply, edge cases
**Reference**: Microsoft Learn link
```

## Code Markers

- `// ❌ BAD:` - Anti-pattern to avoid
- `// ✅ GOOD:` - Recommended approach
- `// ✅ BETTER:` - Alternative or enhanced solution
- `// ✅ NOTE:` - Important clarification

## Common Production Issues

1. **N+1 queries** - Missing eager loading causes thousands of database round trips
2. **Async waterfalls** - Sequential awaits multiply latency (50%+ reduction possible)
3. **Secrets in code** - Hardcoded credentials lead to breaches
4. **Resource leaks** - Not disposing IDisposable causes memory leaks and socket exhaustion
5. **Wrong status codes** - API clients can't handle errors properly

## Technology Focus

- **.NET 8+** (LTS and STS versions)
- **ASP.NET Core** (Web APIs, Minimal APIs)
- **Entity Framework Core** (ORM)
- **C# 12+** (Language features)

## Related Skills

| Skill | Use instead when... |
|-------|-------------------|
| `uno-platform-agent` | Setting up Uno Platform projects, MVVM/MVUX, or platform-specific code |
| `winui-xaml` | Optimizing XAML layout, binding, or UI performance |
| `uno-extensions-services` | Configuring hosting, DI, authentication, or HTTP clients in Uno Platform |

## Detailed References

Read the reference file matching the task at hand:

- [references/01-database-access-critical.md](references/01-database-access-critical.md) - Read when writing EF Core queries, fixing N+1 problems, or optimizing database access
- [references/02-async-parallelism-critical.md](references/02-async-parallelism-critical.md) - Read when writing async code, parallelizing operations, or debugging deadlocks
- [references/03-security-identity-critical.md](references/03-security-identity-critical.md) - Read when handling secrets, credentials, or authentication configuration
- [references/04-api-design-high.md](references/04-api-design-high.md) - Read when designing REST APIs, choosing status codes, or documenting endpoints
- [references/05-memory-resources-high.md](references/05-memory-resources-high.md) - Read when managing IDisposable resources, HttpClient, or preventing memory leaks
