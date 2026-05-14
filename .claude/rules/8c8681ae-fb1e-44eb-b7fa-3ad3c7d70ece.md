<rule_activation id="8c8681ae-fb1e-44eb-b7fa-3ad3c7d70ece" title="Implement Cache Layer for Integration Pattern Data Access: Cache Implementations Provide" applies_to="**/*">
These rules are ALWAYS ACTIVE for all files matching integration patterns, backend client integrations, build pipeline scripts, and Electron packaging workflows.
</rule_activation>

### Rules

- **R-CACHE-001** SHOULD: Cache implementations SHOULD provide metrics and observability hooks to monitor hit rates, miss rates, and cache effectiveness.

### Scope

**In scope:**
- Backend client integrations (RDP, API clients, remote service connectors)
- Build pipeline scripts that access or generate artifacts
- Electron packaging and distribution workflows
- Any integration component with network I/O or expensive computation
- Third-party API integrations with rate limits or quota constraints

**Out of scope:**
- In-memory data structures used within a single function scope
- Database query result caching (covered by database-specific ADRs)
- Browser-side HTTP caching (handled by HTTP cache headers)
- User session state management (covered by session management ADRs)

**Exceptions:**
- EXC-001: Real-time data requirements mandate fresh data on every access with latency SLA < 50ms
- EXC-002: Security or compliance requirements prohibit data caching due to sensitivity

### Verify

```bash
# Check for cache implementations in integration patterns
grep -r "cache" packages/computer/src/rdp/ apps/studio/scripts/ --include="*.ts" --include="*.mjs" | grep -E "(Map|LRU|TTL|invalidate)"

# Find backend client integrations with cache implementations
find . -name "*.ts" -o -name "*.mjs" | xargs grep -l "backend.*client" | xargs grep -l "cache"

# Verify cache-related tests
npm test -- --grep "cache" --reporter json | jq '.tests[] | select(.title | contains("cache"))'
```

**Accept when:**
- All integration components accessing remote data sources implement cache layers with documented TTL or invalidation strategies
- Cache implementations include unit tests verifying cache hit/miss behavior and invalidation logic
- Code review checklist includes verification of cache key uniqueness and invalidation strategy appropriateness
- Cache implementations provide metrics and observability hooks for monitoring effectiveness
- Cache invalidation logic is comprehensively tested with conservative TTL values
- LRU or size-based eviction policies are implemented to prevent unbounded cache growth
- Request coalescing or stale-while-revalidate patterns are implemented to mitigate cache stampede

<enforcement>
Claude Code MUST verify all integration components implement cache layers with documented strategies and observability hooks. Violations MUST be flagged in code review with performance analysis. Architecture review board escalation is required for repeated violations or missing cache strategies. Exception requests MUST include performance analysis and security team approval for data sensitivity concerns.
</enforcement>