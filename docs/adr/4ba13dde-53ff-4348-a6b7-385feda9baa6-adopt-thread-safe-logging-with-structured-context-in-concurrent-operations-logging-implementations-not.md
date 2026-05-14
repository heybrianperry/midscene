# Adopt Thread-Safe Logging with Structured Context in Concurrent Operations: Logging Implementations Not

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all logging operations in concurrent execution contexts including web workers, event bridges, async handlers, and multi-threaded components.

## Context

- The codebase contains concurrent execution contexts including Chrome extension workers, event recorder bridges, bridge connectors, and CLI configuration factories that operate asynchronously
- Pattern signature 826a21fc9a491b735eb535a9b494bddb was detected across 4 files with 91.22% confidence, indicating a consistent approach to logging in concurrent scenarios
- Concurrent operations such as web workers, message passing bridges, and async configuration loading require coordinated logging to maintain observability without race conditions
- The facet 'paradigm.concurrency_model' indicates this pattern specifically addresses how logging integrates with the system's concurrency architecture
- Without proper thread-safe logging mechanisms, concurrent operations can produce interleaved, corrupted, or lost log entries that hinder debugging and monitoring

## Problem Statement

How do we ensure logging operations remain consistent, thread-safe, and contextually meaningful when executed across concurrent contexts such as web workers, event bridges, and asynchronous handlers, while maintaining observability and debuggability without introducing race conditions or performance bottlenecks?

## Decision

1. MUST: Logging implementations MUST NOT block concurrent execution flows; all log operations MUST be non-blocking or use async buffering mechanisms

## Policy Block

- MUST Logging implementations MUST NOT block concurrent execution flows; all log operations MUST be non-blocking or use async buffering mechanisms

In scope:
- Chrome extension web workers and service workers
- Event recorder bridges and message passing interfaces
- Bridge connector components handling async communication
- CLI configuration factories with async initialization
- Any component executing in a concurrent or asynchronous context
- Logging operations that span multiple execution contexts

Out of scope:
- Synchronous, single-threaded logging in non-concurrent contexts
- Debug console output during local development (unless explicitly testing concurrency)
- Performance profiling logs that intentionally measure timing without context propagation
- Third-party library logging that cannot be modified

Exceptions:
- EXC-001: Emergency diagnostic logging during critical failures where thread safety cannot be guaranteed
- EXC-002: Prototype or experimental code in isolated feature branches not intended for production

## Rationale

- Pattern signature 826a21fc9a491b735eb535a9b494bddb detected across 4 files (worker.ts, event-recorder-bridge.ts, bridgeConnector.ts, config-factory.ts) with 91.22% confidence demonstrates a consistent architectural approach to concurrent logging
- The facet 'paradigm.concurrency_model' indicates this pattern is fundamental to how the system handles concurrent operations, making standardized logging critical for system observability
- Chrome extension architecture with web workers and event bridges requires coordinated logging to trace operations across isolated execution contexts
- Thread-safe structured logging with context propagation enables effective debugging and monitoring in production environments where concurrent operations are the norm

## Consequences

Positive:
- Improved debuggability: Structured context identifiers enable tracing of operations across concurrent boundaries, making it easier to diagnose issues in production
- Prevented data corruption: Thread-safe logging mechanisms eliminate race conditions that could produce corrupted or interleaved log entries
- Enhanced observability: Consistent logging patterns across concurrent components provide reliable monitoring and alerting capabilities
- Better performance: Non-blocking logging operations prevent concurrent execution flows from being delayed by I/O-bound logging operations

Negative:
- Increased complexity: Implementing thread-safe logging with context propagation requires additional infrastructure and developer awareness
- Memory overhead: Buffering mechanisms and context storage add memory consumption, particularly in high-throughput concurrent scenarios
- Learning curve: Developers must understand concurrency implications and properly initialize/propagate logging context
- Potential latency: Async buffering and queuing mechanisms may introduce slight delays in log availability for real-time monitoring

## Alternatives

- Use simple console.log() without thread safety or context propagation (rejected)
  Rejected because: Console.log in concurrent contexts produces interleaved output that is difficult to parse, lacks context for tracing across execution boundaries, and provides no guarantees about atomicity or ordering
  When valid: Only acceptable for trivial single-threaded scripts or temporary debugging in local development
- Implement per-worker isolated logging with no cross-context correlation (rejected)
  Rejected because: Isolated logging prevents tracing operations that span multiple workers or bridges, making it impossible to debug complex workflows that involve coordination between concurrent components
  When valid: Could be used for completely independent workers that never interact with other system components
- Use a centralized logging service with message queuing and context injection (accepted)
  When valid: This is the recommended approach as it provides thread safety, context propagation, and non-blocking operations through async queuing

## Risks

- Log buffer overflow in high-throughput scenarios could lead to dropped log entries or memory exhaustion
  Mitigation: Implement bounded buffers with backpressure mechanisms, monitor buffer utilization, and configure appropriate buffer sizes based on load testing
  Owner: Platform Engineering Team
- Context propagation failures could result in logs without proper correlation IDs, breaking traceability
  Mitigation: Implement automated testing that verifies context propagation across concurrent boundaries, use TypeScript types to enforce context passing, and add runtime validation
  Owner: Engineering Team
- Performance degradation if logging operations become synchronous bottlenecks in critical paths
  Mitigation: Conduct performance profiling of logging operations, ensure all logging is truly non-blocking, and implement sampling for high-frequency log events
  Owner: Performance Engineering Team

## Implementation Notes

- Use a logging library that provides native support for concurrent contexts (e.g., pino, winston with async transports) rather than implementing custom thread-safety mechanisms
- Initialize logging context at the entry point of each concurrent component (worker creation, bridge initialization) and store in a context object that is passed through the call chain
- For Chrome extension workers, use chrome.runtime.sendMessage or postMessage to send log entries to a centralized logging coordinator in the background script
- Implement correlation ID generation using UUIDs or timestamp-based identifiers at the start of each user-initiated operation, and propagate through all concurrent operations
- Add TypeScript interfaces that enforce logging context parameters, making it a compile-time error to log without required context in concurrent components

## Continuation Context


Verify commands:
- grep -r "console\.log" apps/chrome-extension/src/scripts/ packages/cli/src/ | grep -E "(worker|bridge|async)" && echo "FAIL: Found unsafe console.log in concurrent contexts" || echo "PASS: No unsafe console.log found"
- grep -r "logger\.(info|error|warn|debug)" apps/chrome-extension/src/scripts/ | grep -v "contextId\|workerId\|correlationId" && echo "FAIL: Found logs without context" || echo "PASS: All logs include context"
- npm test -- --grep "concurrent.*logging" && echo "PASS: Concurrent logging tests pass" || echo "FAIL: Concurrent logging tests failed"

Accept when:
- All logging operations in worker.ts, event-recorder-bridge.ts, bridgeConnector.ts, and config-factory.ts use a thread-safe logging library with structured context
- Grep verification confirms no unsafe console.log usage in concurrent contexts and all log entries include correlation/context identifiers
- Automated tests verify that log entries from concurrent operations can be correctly correlated and ordered by timestamp

## Enforcement

- Verified by: Automated CI pipeline runs grep-based verification commands to detect unsafe logging patterns
- Verified by: Code review checklist includes verification of logging context propagation in concurrent components
- Verified by: Integration tests validate log correlation across concurrent boundaries
- Verified by: Static analysis tools (ESLint rules) flag console.log usage in files matching concurrent patterns
- Violation handling: CI pipeline fails if verification commands detect unsafe logging patterns in concurrent contexts
- Violation handling: Code review blocks merge if logging context is missing from concurrent operations
- Violation handling: Runtime warnings are logged if context propagation validation detects missing correlation IDs
- Violation handling: Quarterly audits review logging patterns and identify components requiring refactoring
- Exception process: Developer submits exception request via GitHub issue with justification and impact assessment
- Exception process: Engineering lead reviews exception request and approves/rejects within 2 business days
- Exception process: Approved exceptions are documented in ADR amendments with expiration dates
- Exception process: All exceptions are reviewed quarterly and must be re-justified or remediated