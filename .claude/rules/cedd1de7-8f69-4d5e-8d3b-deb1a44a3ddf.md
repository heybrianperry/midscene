<rule_activation id="cedd1de7-8f69-4d5e-8d3b-deb1a44a3ddf" title="Adopt Event-Driven Concurrency with Debouncing for Data Access Operations: Event Driven Data" applies_to="**/*">
These rules are ALWAYS ACTIVE for all data access operations involving event-driven patterns, debouncing, and concurrency control mechanisms.
</rule_activation>

### Rules

- **R-EDDCA-001** MUST: Event-driven data access patterns MUST use asynchronous concurrency models with proper promise handling and error boundaries.

### Scope

**In scope:**
- Event-driven data access operations in browser extensions and web applications
- Real-time data streaming components (e.g., MJPEG hubs, WebSocket handlers)
- Keepalive mechanisms and persistent connection management
- User input handlers that trigger backend API calls
- System event listeners that perform data synchronization

**Out of scope:**
- One-time initialization data fetches
- Batch processing operations with explicit scheduling
- Server-side data access patterns with different concurrency models
- Database transaction management within backend services
- Static data loading during application bootstrap

**Exceptions:**
- EXC-001: Critical real-time events that require immediate data access without debouncing (e.g., emergency alerts, security events)
- EXC-002: Legacy components undergoing gradual migration where immediate refactoring would introduce regression risk

### Verify

```bash
# Count debounce/throttle usage
grep -r "debounce\|throttle" --include="*.ts" --include="*.tsx" apps/ packages/ | wc -l

# Count setTimeout/setInterval without cleanup
grep -r "setTimeout\|setInterval" --include="*.ts" --include="*.tsx" apps/ packages/ | grep -v "clearTimeout\|clearInterval" | wc -l

# Count concurrency-related tests
npm run test -- --grep "debounce|throttle|concurrency" --reporter json | jq '.tests | length'
```

**Accept when:**
- All event-driven data access components implement debouncing or throttling with documented intervals
- No setTimeout/setInterval calls exist without corresponding cleanup in component lifecycle or error handlers
- Test coverage for concurrency patterns exceeds 80% with specific tests for cleanup and race conditions

### Implementation Guidance

- Create shared utility modules (e.g., eventOptimizer.ts) that export reusable debounce and throttle functions with consistent interfaces
- Use TypeScript generics to ensure type safety when wrapping event handlers with debouncing logic
- Implement cleanup tracking using AbortController or similar patterns to ensure all pending operations can be cancelled on component unmount
- Document debounce intervals as named constants with rationale (e.g., USER_INPUT_DEBOUNCE_MS = 300) rather than magic numbers
- Consider implementing adaptive debouncing that adjusts intervals based on event frequency patterns
- Establish standard debounce intervals for common use cases: user input 300ms, system events 1000ms, keepalive 30s

<enforcement>
Claude Code MUST NOT skip or defer verification. All event-driven data access patterns MUST comply with R-EDDCA-001. CI pipeline MUST fail if event handlers lack debouncing and no exception is documented. Code review MUST require explicit justification for synchronous data access in event handlers.
</enforcement>