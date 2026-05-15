# Adopt Message Queue Logging Pattern for Asynchronous Event Processing: Message Queue Send

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all components that process asynchronous events through message queues, including Chrome extension workers, event recorder bridges, and timeline recording systems.

## Context

- The system processes asynchronous events across multiple boundaries including Chrome extension workers, event recorder bridges, and timeline recording components
- Message queues facilitate communication between isolated contexts (e.g., service workers, content scripts, UI components) where direct function calls are not possible
- Debugging asynchronous message flows requires visibility into message lifecycle events including send, receive, processing, and error states
- The pattern was detected across 3 files with 90.83% confidence, indicating consistent implementation of message queue logging at architectural boundaries
- Chrome extension architecture imposes strict isolation between contexts, making message-based communication and associated logging critical for operational visibility

## Problem Statement

Asynchronous message-based communication across architectural boundaries creates observability challenges where traditional synchronous logging is insufficient. Without structured logging at message queue boundaries, debugging message flow failures, tracking event processing latency, and diagnosing communication breakdowns between isolated contexts becomes extremely difficult, leading to increased mean time to resolution (MTTR) for production issues.

## Decision

1. MUST: All message queue send operations MUST log the message type, destination, and timestamp before transmission

## Policy Block

- MUST All message queue send operations MUST log the message type, destination, and timestamp before transmission

In scope:
- Chrome extension service workers and background scripts
- Event recorder bridge components that mediate between contexts
- Timeline recording systems that process event streams
- Any component that sends or receives messages across architectural boundaries
- Message queue implementations including postMessage, chrome.runtime.sendMessage, and custom queue abstractions

Out of scope:
- Synchronous function calls within the same execution context
- Direct DOM event handlers that do not cross architectural boundaries
- Internal state management that does not involve message passing
- Third-party library logging (unless wrapping message queue operations)

Exceptions:
- EXC-001: High-frequency heartbeat or keepalive messages that would generate excessive log volume
- EXC-002: Messages containing sensitive user data that cannot be logged for privacy/security reasons

## Rationale

- Pattern detected with 90.83% confidence across 3 critical files (worker.ts, event-recorder-bridge.ts, RecordTimeline.tsx) indicates this is an established architectural practice
- Message queue boundaries represent the highest-risk points for communication failures in distributed or isolated-context architectures
- Structured logging at these boundaries enables correlation of events across architectural components, reducing debugging time from hours to minutes
- The facet 'boundaries.message_queues' explicitly identifies this as a boundary-crossing concern requiring special observability treatment

## Consequences

Positive:
- Significantly improved debugging capability for asynchronous message flow issues across architectural boundaries
- Enables end-to-end tracing of events through the system using correlation IDs
- Provides performance metrics for identifying message processing bottlenecks
- Creates audit trail for message-based communication useful for compliance and security analysis

Negative:
- Increased log volume may impact storage costs and log aggregation system performance
- Additional logging code increases maintenance surface area and potential for logging-related bugs
- Performance overhead from logging operations, particularly for high-frequency message queues
- Risk of accidentally logging sensitive data if payload logging is not carefully controlled

## Alternatives

- Implement distributed tracing framework (e.g., OpenTelemetry) instead of custom message queue logging (rejected)
  Rejected because: Adds significant dependency overhead for Chrome extension environment; existing pattern is lightweight and proven effective
  When valid: Consider for backend services or when standardized tracing across multiple systems is required
- Use browser DevTools Performance API exclusively without explicit logging (rejected)
  Rejected because: Performance API does not capture message semantics, error conditions, or provide production observability
  When valid: Useful as complementary tool for development-time performance profiling
- Implement centralized message bus with built-in observability (deferred)
  Rejected because: Would require significant architectural refactoring; current pattern provides immediate value
  When valid: Consider for future architecture evolution if message complexity grows significantly

## Risks

- Excessive logging volume in high-throughput message queues may degrade performance or overwhelm log storage
  Mitigation: Implement sampling strategies for high-volume queues; monitor log volume metrics; use log level filtering in production
  Owner: Engineering team
- Accidental logging of sensitive user data in message payloads could create privacy/security violations
  Mitigation: Implement payload sanitization; use allowlist approach for loggable fields; conduct security review of logging code
  Owner: Security team
- Inconsistent logging implementation across components reduces effectiveness of pattern
  Mitigation: Create shared logging utilities; enforce through code review; add linting rules to detect missing message queue logs
  Owner: Engineering team

## Implementation Notes

- Create shared logging utility functions (e.g., logMessageSend, logMessageReceive) to ensure consistent format across all message queue implementations
- Use structured logging with consistent field names: messageType, source, destination, timestamp, correlationId, duration, error
- Implement correlation ID generation at message origin and propagate through all hops to enable end-to-end tracing
- Configure log levels appropriately: INFO for normal message flow, WARN for retries/timeouts, ERROR for failures
- Consider implementing a message queue wrapper/decorator that automatically adds logging to reduce boilerplate

## Continuation Context


Verify commands:
- grep -r "postMessage\|sendMessage\|addEventListener.*message" --include="*.ts" --include="*.tsx" | xargs -I {} sh -c 'grep -l "log\|console" {}'
- grep -r "class.*Queue\|interface.*Queue" --include="*.ts" | xargs -I {} sh -c 'grep -c "log" {} || echo "0"'
- npm run test -- --grep "message.*log" 2>/dev/null || echo "No message logging tests found"

Accept when:
- All files containing message queue operations (postMessage, sendMessage, message event listeners) include corresponding log statements
- Log statements include minimum required fields: message type, timestamp, and source/destination context
- Error handling blocks for message processing include error logging with full context
- Code review checklist includes verification of message queue logging compliance

## Enforcement

- Verified by: Code review process with specific checklist item for message queue logging
- Verified by: Static analysis linting rules to detect message queue operations without corresponding logs
- Verified by: Automated tests verifying log output for message send/receive operations
- Violation handling: Code review blocks merge if message queue operations lack required logging
- Violation handling: Linter warnings escalated to errors in CI pipeline for message queue logging violations
- Violation handling: Post-deployment monitoring alerts on components with suspiciously low log volume
- Exception process: Developer submits exception request via GitHub issue with justification and alternative monitoring approach
- Exception process: Engineering lead reviews and approves/rejects within 2 business days
- Exception process: Approved exceptions documented in code comments with reference to exception ID and expiration date
- Exception process: Exceptions reviewed quarterly for continued validity