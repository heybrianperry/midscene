<rule_activation id="ce50776c-7ebf-4425-aefc-ebeee0773ad7" title="Adopt Message Queue Logging Pattern for Asynchronous Event Processing: Message Queue Logs" applies_to="**/*">
These rules are ALWAYS ACTIVE for all components that process asynchronous events through message queues, including Chrome extension workers, event recorder bridges, and timeline recording systems.
</rule_activation>

### Rules

- **R-MQL-001** SHOULD: Message queue logs SHOULD include correlation IDs to enable end-to-end tracing across architectural boundaries.

### Scope

**In scope:**
- Chrome extension service workers and background scripts
- Event recorder bridge components that mediate between contexts
- Timeline recording systems that process event streams
- Any component that sends or receives messages across architectural boundaries
- Message queue implementations including postMessage, chrome.runtime.sendMessage, and custom queue abstractions

**Out of scope:**
- Synchronous function calls within the same execution context
- Direct DOM event handlers that do not cross architectural boundaries
- Internal state management that does not involve message passing
- Third-party library logging (unless wrapping message queue operations)

**Exceptions:**
- EXC-001: High-frequency heartbeat or keepalive messages that would generate excessive log volume
- EXC-002: Messages containing sensitive user data that cannot be logged for privacy/security reasons

### Verify

```bash
# Detect message queue operations and verify logging presence
grep -r "postMessage\|sendMessage\|addEventListener.*message" --include="*.ts" --include="*.tsx" | xargs -I {} sh -c 'grep -l "log\|console" {}'

# Check for Queue class/interface implementations with logging
grep -r "class.*Queue\|interface.*Queue" --include="*.ts" | xargs -I {} sh -c 'grep -c "log" {} || echo "0"'

# Run message logging tests
npm run test -- --grep "message.*log" 2>/dev/null || echo "No message logging tests found"
```

**Accept when:**
- All files containing message queue operations (postMessage, sendMessage, message event listeners) include corresponding log statements
- Log statements include minimum required fields: message type, timestamp, and source/destination context
- Error handling blocks for message processing include error logging with full context
- Code review checklist includes verification of message queue logging compliance
- Structured logging uses consistent field names: messageType, source, destination, timestamp, correlationId, duration, error
- Correlation IDs are generated at message origin and propagated through all hops
- Log levels are configured appropriately: INFO for normal flow, WARN for retries/timeouts, ERROR for failures

<enforcement>
Clause Code MUST verify message queue logging compliance through code review, static analysis linting rules, and automated tests. Violations block merge and escalate to CI pipeline errors. Exceptions require GitHub issue justification and engineering lead approval within 2 business days, with quarterly review.
</enforcement>