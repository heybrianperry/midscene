<rule_activation id="65ab46b7-bde4-4b68-aacd-91eae30edc1a" title="Adopt Event-Driven Keepalive Pattern for Long-Running Connections: Event Optimizers Implement" applies_to="**/*">
These rules are ALWAYS ACTIVE for all data access patterns involving persistent connections, streaming protocols, or long-lived client-server interactions where connection stability is critical.
</rule_activation>

### Rules

- **R-KEEPALIVE-001** SHOULD: Event optimizers SHOULD implement debouncing or throttling strategies to reduce event processing overhead

### Verify

```bash
# Verify keepalive mechanisms are present in connection implementations
grep -r "keepalive\|keepAlive" --include="*.ts" --include="*.js" | grep -v node_modules

# Verify keepalive patterns in interval-based implementations
grep -r "setInterval\|setTimeout" --include="*.ts" --include="*.js" -E "(ping|pong|heartbeat|keepalive)" | grep -v node_modules

# Verify browser lifecycle event handling
grep -r "addEventListener.*visibility\|addEventListener.*suspend" --include="*.ts" --include="*.js" | grep -v node_modules
```

**Accept when:**
- All long-running connection implementations include explicit keepalive mechanisms with configurable intervals
- Event-driven data access patterns demonstrate event optimization through debouncing, throttling, or batching
- Connection management code handles browser lifecycle events (visibility changes, suspend/resume) appropriately
- Verification commands identify consistent patterns across connection management modules

<enforcement>
Claude Code MUST NOT skip or defer verification. All long-running connections, WebSocket implementations, Server-Sent Events, streaming protocols, browser extension background connections, and real-time data synchronization patterns MUST comply with keepalive and event optimization requirements before approval.
</enforcement>