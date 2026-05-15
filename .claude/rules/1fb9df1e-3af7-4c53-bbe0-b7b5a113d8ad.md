<rule_activation id="1fb9df1e-3af7-4c53-bbe0-b7b5a113d8ad" title="Adopt Event-Driven Logging with Buffered Batch Processing for Data Access Operations: Event Buffers Implement" applies_to="**/*">
These rules are ALWAYS ACTIVE for all files matching `**/*`. Event-driven logging with buffered batch processing MUST be applied to data access operations in performance-sensitive contexts including browser extensions, real-time streaming, keepalive mechanisms, and performance-critical data retrieval layers.
</rule_activation>

### Rules

- **R-EVLOG-001** SHOULD: Event buffers SHOULD implement configurable flush intervals and size thresholds to balance latency and throughput.

### Verify

```bash
# Count occurrences of event buffer and logging patterns
grep -r "eventOptimizer\|EventBuffer\|LogBuffer" --include="*.ts" --include="*.js" | wc -l

# Count flush and batch logging patterns
grep -r "flush.*log\|batch.*log" --include="*.ts" --include="*.js" | grep -v "node_modules" | wc -l

# Run logging and buffer related tests
npm test -- --grep "logging.*buffer\|event.*batch" 2>&1 | grep -E "passing|failing"
```

**Accept when:**
- All data access operations in performance-critical paths use buffered logging mechanisms rather than synchronous writes
- Event buffer implementations include configurable flush intervals and maximum size limits
- Log events include structured metadata (timestamps, correlation IDs, operation context) sufficient for distributed tracing
- Tests verify that logging does not block data access operations and that buffers flush correctly on lifecycle events

<enforcement>
Claude Code MUST NOT skip or defer verification. Violations of buffered logging requirements in performance-critical paths MUST trigger CI build warnings, code review rejection, and performance regression alerts. Exceptions require documented justification, performance impact analysis, and approval from both component owner and platform architecture team.
</enforcement>