# Adopt Message-Based Communication for Chrome Extension Component Integration: Components Implement Message

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all Chrome extension component integrations involving recorder, confirm, and bridge modules. All message-passing implementations between extension contexts MUST comply with these rules.

## Context

- Chrome extensions operate in multiple isolated execution contexts (background scripts, content scripts, popup UI, injected pages) that cannot directly share memory or function calls
- The recorder, confirm, and bridge components need to coordinate user interactions and data flow across these isolated contexts
- Message queues provide asynchronous, decoupled communication that aligns with Chrome's extension architecture and security model
- Pattern detected across 3 critical extension files (recorder/utils.ts, confirm/index.tsx, bridge/index.tsx) with 91.63% confidence, indicating consistent architectural approach
- The boundaries.message_queues facet suggests intentional use of message-passing as an integration boundary pattern

## Problem Statement

Chrome extension components operate in isolated execution contexts with no shared memory, requiring a reliable, type-safe mechanism to coordinate actions, share state, and trigger behaviors across the recorder, confirmation UI, and bridge layers without tight coupling or blocking operations.

## Decision

1. MAY: Components MAY implement message queuing or buffering for scenarios where the receiving context may not be immediately available

## Policy Block

- MAY Components MAY implement message queuing or buffering for scenarios where the receiving context may not be immediately available

In scope:
- All recorder utility functions that communicate with other extension contexts
- Confirmation UI components that need to send user decisions to background or content scripts
- Bridge components that mediate between injected page context and extension contexts
- Any cross-context state synchronization or command dispatch

Out of scope:
- Communication within a single execution context (e.g., React component props, function calls within the same script)
- External API calls to backend services (use standard HTTP/WebSocket patterns)
- Storage operations using chrome.storage APIs (separate persistence pattern)
- DOM manipulation within content scripts (use direct DOM APIs)

Exceptions:
- EXC-001: Performance-critical operations require shared memory via SharedArrayBuffer in controlled contexts

## Rationale

- Chrome's extension security model enforces context isolation, making message-passing the only viable communication mechanism across contexts
- Pattern detected with 91.63% confidence across 3 key integration points (recorder, confirm, bridge) demonstrates this is an established architectural standard in the codebase
- Message-based communication provides loose coupling, enabling independent evolution of components and easier testing through message mocking
- Asynchronous message-passing aligns with Chrome's event-driven architecture and prevents blocking operations that could degrade user experience

## Consequences

Positive:
- Clear separation of concerns with well-defined integration boundaries between extension contexts
- Components can be developed, tested, and deployed independently with minimal coupling
- Type-safe message contracts enable compile-time validation of cross-context communication
- Asynchronous communication prevents UI blocking and improves perceived performance

Negative:
- Message serialization overhead adds latency compared to direct function calls (typically 1-5ms per message)
- Debugging cross-context communication is more complex than debugging synchronous function calls
- Requires careful error handling for message delivery failures and timeout scenarios
- TypeScript type safety is lost at runtime; messages must be validated at context boundaries

## Alternatives

- Use chrome.storage for state sharing between contexts (rejected)
  Rejected because: Storage APIs are designed for persistence, not real-time communication; introduces unnecessary I/O overhead and lacks command/action semantics
  When valid: Appropriate for persisting user preferences or cached data, not for real-time coordination
- Implement shared worker for centralized state management (rejected)
  Rejected because: Adds architectural complexity and is not supported in all Chrome extension contexts; message-passing is simpler and more universally supported
  When valid: Could be considered for complex multi-tab coordination scenarios with heavy computational requirements
- Use window.postMessage for injected page communication (accepted)
  When valid: Specifically for communication between content scripts and injected page scripts where chrome.runtime APIs are not available

## Risks

- Message delivery failures due to context lifecycle (e.g., popup closed, content script unloaded) could result in lost commands or inconsistent state
  Mitigation: Implement acknowledgment patterns, timeout handling, and state reconciliation mechanisms; use chrome.storage for critical state persistence
  Owner: Extension engineering team
- Type safety is not enforced at runtime; malformed messages could cause runtime errors or security vulnerabilities
  Mitigation: Implement runtime message validation using schema validators (e.g., Zod, io-ts) at context boundaries; add comprehensive error handling
  Owner: Extension engineering team
- Message-passing overhead could accumulate in high-frequency scenarios (e.g., recording rapid user interactions)
  Mitigation: Implement message batching for high-frequency events; profile performance and optimize message payload size
  Owner: Performance engineering team

## Implementation Notes

- Create a centralized message type registry (e.g., MessageTypes enum) to ensure consistent message type strings across all contexts
- Implement a typed message bus abstraction layer that wraps chrome.runtime APIs and provides type-safe send/receive methods
- Use TypeScript discriminated unions for message types to enable exhaustive pattern matching in message handlers
- Add logging/tracing for message flow to aid debugging; consider using Chrome DevTools' message inspection capabilities
- Document message contracts in a shared types package accessible to all extension contexts

## Continuation Context


Verify commands:
- grep -r 'chrome\.runtime\.sendMessage\|chrome\.tabs\.sendMessage' apps/chrome-extension/src/extension/ | wc -l
- grep -r 'chrome\.runtime\.onMessage\.addListener' apps/chrome-extension/src/extension/ | wc -l
- grep -r 'type.*Message.*=.*{.*type:' apps/chrome-extension/src/extension/ --include='*.ts' --include='*.tsx'

Accept when:
- All cross-context communication in recorder, confirm, and bridge modules uses chrome.runtime message APIs
- Message handlers are registered with chrome.runtime.onMessage.addListener in each context
- TypeScript message type definitions exist with discriminated 'type' or 'action' fields

## Enforcement

- Verified by: Automated code review checks for chrome.runtime API usage patterns
- Verified by: TypeScript compilation enforces message type contracts at build time
- Verified by: Integration tests verify message flow between extension contexts
- Verified by: ESLint rules prohibit direct cross-context function calls or shared state access
- Violation handling: CI pipeline fails if direct cross-context communication patterns are detected
- Violation handling: Code review requires explicit justification for any non-message-based integration
- Violation handling: Runtime errors logged and reported when message validation fails
- Violation handling: Quarterly architecture reviews audit extension communication patterns
- Exception process: Submit exception request to architecture review board with performance data or technical justification
- Exception process: Document alternative approach and security implications in ADR amendment
- Exception process: Obtain approval from extension security lead and architecture owner
- Exception process: Add exception to policy_exceptions with clear scope and expiration criteria