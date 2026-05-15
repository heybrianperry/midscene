<rule_activation id="bc7cafba-f3bb-4162-b3fe-c775f2ab9756" title="Adopt Thread-Safe Logging with Structured Context in Concurrent Operations: Components Implement Per" applies_to="**/*">
These rules are ACTIVE for all logging operations in concurrent execution contexts including web workers, event bridges, async handlers, and multi-threaded components.
</rule_activation>

### Rules

- **R-TSL-001** MAY: Components MAY implement per-thread or per-worker log buffers that are periodically flushed to a central logging sink.

### Verify

```bash
# Check for unsafe console.log in concurrent contexts
grep -r "console\.log" apps/chrome-extension/src/scripts/ packages/cli/src/ | grep -E "(worker|bridge|async)" && echo "FAIL: Found unsafe console.log in concurrent contexts" || echo "PASS: No unsafe console.log found"

# Verify all logs include context identifiers
grep -r "logger\.(info|error|warn|debug)" apps/chrome-extension/src/scripts/ | grep -v "contextId\|workerId\|correlationId" && echo "FAIL: Found logs without context" || echo "PASS: All logs include context"

# Run concurrent logging tests
npm test -- --grep "concurrent.*logging" && echo "PASS: Concurrent logging tests pass" || echo "FAIL: Concurrent logging tests failed"
```

**Accept when:**
- All logging operations in worker.ts, event-recorder-bridge.ts, bridgeConnector.ts, and config-factory.ts use a thread-safe logging library with structured context
- Grep verification confirms no unsafe console.log usage in concurrent contexts and all log entries include correlation/context identifiers
- Automated tests verify that log entries from concurrent operations can be correctly correlated and ordered by timestamp

<enforcement>
Claude Code MUST verify all logging operations in concurrent contexts comply with thread-safe structured logging requirements. Verification is mandatory before accepting changes to worker.ts, event-recorder-bridge.ts, bridgeConnector.ts, config-factory.ts, and any component executing in concurrent or asynchronous contexts.
</enforcement>