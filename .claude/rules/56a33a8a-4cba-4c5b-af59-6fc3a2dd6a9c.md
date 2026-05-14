<rule_activation id="56a33a8a-4cba-4c5b-af59-6fc3a2dd6a9c" title="Adopt Message-Based Communication for Chrome Extension Component Integration: Communication Between Chrome" applies_to="apps/chrome-extension/src/extension/**/*.{ts,tsx}">
These rules are ALWAYS ACTIVE for all Chrome extension component integrations involving recorder, confirm, and bridge modules. All message-passing implementations between extension contexts MUST comply with these rules.
</rule_activation>

### Rules

- **R-MSG-001** MUST: All communication between Chrome extension contexts (background, content script, popup, injected page) MUST use message-based communication via `chrome.runtime.sendMessage` or `chrome.tabs.sendMessage` APIs.

- **R-MSG-002** MUST: All recorder utility functions that communicate with other extension contexts MUST use message-based communication.

- **R-MSG-003** MUST: Confirmation UI components that need to send user decisions to background or content scripts MUST use message-based communication.

- **R-MSG-004** MUST: Bridge components that mediate between injected page context and extension contexts MUST use message-based communication.

- **R-MSG-005** MUST: Any cross-context state synchronization or command dispatch MUST use message-based communication.

- **R-MSG-006** MUST: Create a centralized message type registry (e.g., MessageTypes enum) to ensure consistent message type strings across all contexts.

- **R-MSG-007** MUST: Implement a typed message bus abstraction layer that wraps chrome.runtime APIs and provides type-safe send/receive methods.

- **R-MSG-008** MUST: Use TypeScript discriminated unions for message types to enable exhaustive pattern matching in message handlers.

- **R-MSG-009** MUST: Implement runtime message validation using schema validators (e.g., Zod, io-ts) at context boundaries.

- **R-MSG-010** MUST: Implement acknowledgment patterns, timeout handling, and state reconciliation mechanisms for message delivery failures.

- **R-MSG-011** SHOULD: Add logging/tracing for message flow to aid debugging; consider using Chrome DevTools' message inspection capabilities.

- **R-MSG-012** SHOULD: Document message contracts in a shared types package accessible to all extension contexts.

- **R-MSG-013** SHOULD: Implement message batching for high-frequency events to mitigate performance overhead.

### Verify

```bash
# Count chrome.runtime message API usage
grep -r 'chrome\.runtime\.sendMessage\|chrome\.tabs\.sendMessage' apps/chrome-extension/src/extension/ | wc -l

# Count message listener registrations
grep -r 'chrome\.runtime\.onMessage\.addListener' apps/chrome-extension/src/extension/ | wc -l

# Find TypeScript message type definitions with discriminated unions
grep -r 'type.*Message.*=.*{.*type:' apps/chrome-extension/src/extension/ --include='*.ts' --include='*.tsx'
```

**Accept when:**
- All cross-context communication in recorder, confirm, and bridge modules uses `chrome.runtime` message APIs
- Message handlers are registered with `chrome.runtime.onMessage.addListener` in each context
- TypeScript message type definitions exist with discriminated `type` or `action` fields
- Runtime message validation is implemented at context boundaries
- Message type registry is centralized and accessible to all extension contexts
- No direct cross-context function calls or shared state access patterns are detected

<enforcement>
Claude Code MUST NOT skip or defer verification. All message-based communication rules are mandatory for Chrome extension component integrations. Violations detected by automated code review checks, TypeScript compilation, integration tests, and ESLint rules will cause CI pipeline failure. Exception requests require explicit justification, security review, and architecture board approval.
</enforcement>