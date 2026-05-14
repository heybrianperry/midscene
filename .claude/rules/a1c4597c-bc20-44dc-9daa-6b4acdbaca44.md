<rule_activation id="a1c4597c-bc20-44dc-9daa-6b4acdbaca44" title="Adopt Event-Driven Data Access with Buffering and Retry Mechanisms: Data Access Implementations" applies_to="**/*">
These rules are ALWAYS ACTIVE for all files matching network-based data access operations, event handling systems, AI model connectivity, real-time data streaming components, and MCP tool integrations.
</rule_activation>

### Rules

- **R-EVDA-001** MUST: Data access implementations MUST handle connection lifecycle events (connect, disconnect, reconnect) explicitly.

### Scope

In scope:
- All network-based data access operations (HTTP, WebSocket, streaming protocols)
- Event handling systems in browser extensions and web integrations
- AI model connectivity and API interactions
- Real-time data streaming components (MJPEG, video, telemetry)
- MCP tool integrations and external service communications

Out of scope:
- Synchronous, in-memory data access within a single process
- Database query patterns using connection pools (covered by separate ADRs)
- File system I/O operations
- Static configuration loading

Exceptions:
- EXC-001: Real-time systems with strict latency requirements where buffering would violate SLA constraints
- EXC-002: Prototype or experimental features explicitly marked as non-production

### Verify

```bash
# Count retry, backoff, and buffer implementations
grep -r "retry\|backoff\|buffer" --include="*.ts" --include="*.js" packages/ apps/ | grep -v node_modules | wc -l

# Count event-driven patterns
grep -r "EventEmitter\|Observable\|Subject" --include="*.ts" packages/ apps/ | grep -v node_modules | wc -l

# Run tests for retry, buffer, and reconnect scenarios
npm test -- --grep "retry|buffer|reconnect" 2>&1 | grep -E "passing|failing"
```

**Accept when:**
- All network-based data access components implement retry logic with exponential backoff (verified by code review)
- Event-driven components include buffering mechanisms with defined overflow policies (verified by implementation inspection)
- Test suites include scenarios for connection failures, retries, and buffer overflow conditions (verified by test execution)
- Connection lifecycle event handlers (connect, disconnect, reconnect) are explicitly implemented in all in-scope components
- Buffer depth, retry count, and failure rate metrics are integrated with observability infrastructure
- Configurable retry policies with sensible defaults are documented and tested

<enforcement>
Claude Code MUST NOT skip or defer verification. All network-based data access implementations MUST be reviewed for compliance with R-EVDA-001 before merge. Architecture review is required for new components involving network communication or event processing. Violations trigger PR comments requesting addition of retry/buffering mechanisms and connection lifecycle handling.
</enforcement>