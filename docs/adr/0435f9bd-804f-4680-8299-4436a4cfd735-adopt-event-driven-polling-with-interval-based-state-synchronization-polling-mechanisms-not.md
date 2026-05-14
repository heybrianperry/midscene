# Adopt Event-Driven Polling with Interval-Based State Synchronization: Polling Mechanisms Not

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all data access patterns involving periodic state synchronization, event optimization, and keepalive mechanisms across the codebase.

## Context

- The codebase exhibits a recurring pattern of interval-based polling mechanisms for maintaining state synchronization and connection liveness across distributed components
- Multiple modules (eventOptimizer, mjpeg-hub, keepalive) independently implement timer-based data access patterns suggesting a common architectural approach to periodic state management
- Browser extension and real-time streaming contexts require continuous background processes to maintain connection state and optimize event delivery without blocking the main thread
- The pattern emerges from the need to balance responsiveness with resource efficiency in long-running client-side applications that must maintain persistent connections or periodic updates

## Problem Statement

How should the system implement periodic data access and state synchronization in client-side contexts where continuous polling, keepalive signals, and event optimization are required without degrading performance or exhausting system resources?

## Decision

1. MUST_NOT: Polling mechanisms MUST NOT execute synchronous blocking operations that could freeze the UI thread

## Policy Block

- MUST_NOT Polling mechanisms MUST NOT execute synchronous blocking operations that could freeze the UI thread

In scope:
- Browser extension background scripts and service workers requiring keepalive signals
- Real-time streaming components (MJPEG hubs, WebSocket managers) maintaining connection state
- Event optimization utilities that aggregate or throttle high-frequency user interactions
- Any client-side module performing periodic health checks or state synchronization

Out of scope:
- Server-side polling or cron job implementations
- One-time data fetches or request-response patterns without periodic repetition
- WebSocket or Server-Sent Events (SSE) push-based architectures where the server initiates updates
- Native mobile applications using platform-specific background task APIs

Exceptions:
- EXC-001: Critical real-time systems require sub-millisecond polling intervals where timer precision is insufficient
- EXC-002: Legacy third-party libraries impose incompatible polling patterns that cannot be refactored

## Rationale

- Pattern detected across 3 independent modules (eventOptimizer, mjpeg-hub, keepalive) with 90.87% confidence indicates a deliberate architectural choice rather than coincidental implementation
- Interval-based polling provides predictable resource usage and allows the JavaScript event loop to process other tasks between polling cycles, maintaining UI responsiveness
- Browser extension and streaming contexts have strict requirements for background process management where improper polling can lead to extension suspension or memory leaks
- Standardizing this pattern enables code reuse, consistent debugging approaches, and predictable performance characteristics across the application

## Consequences

Positive:
- Consistent polling behavior across all modules reduces cognitive load and makes debugging timing-related issues more predictable
- Proper cleanup of intervals prevents memory leaks and resource exhaustion in long-running client applications
- Configurable intervals allow runtime tuning for different deployment environments without code changes
- Event batching and throttling reduce unnecessary network requests and improve overall application performance

Negative:
- Timer-based polling introduces minimum latency equal to the polling interval, which may be unacceptable for ultra-low-latency requirements
- Multiple independent polling mechanisms can create thundering herd problems if intervals align and fire simultaneously
- Browser timer throttling in background tabs can cause unpredictable polling behavior and stale state
- Increased complexity in testing due to time-dependent behavior requiring mock timers and careful test orchestration

## Alternatives

- Use WebSocket or Server-Sent Events (SSE) for push-based real-time updates instead of client-side polling (rejected)
  Rejected because: Not applicable for browser extension keepalive scenarios where no server connection exists, and adds infrastructure complexity for simple periodic tasks
  When valid: Valid for server-client communication where the server has state changes to push and can maintain persistent connections
- Implement recursive setTimeout pattern instead of setInterval for self-adjusting polling (deferred)
  Rejected because: Adds complexity without clear benefit for current use cases, but may be valuable for adaptive polling scenarios
  When valid: Valid when polling intervals need to adjust dynamically based on response times or error rates
- Use requestAnimationFrame for high-frequency polling tied to display refresh rate (rejected)
  Rejected because: Couples polling to rendering pipeline which is inappropriate for background tasks and wastes resources when visual updates are not needed
  When valid: Valid only for animation-related state updates that must synchronize with screen refresh

## Risks

- Browser timer throttling in background tabs may cause keepalive mechanisms to fail, leading to connection drops or extension suspension
  Mitigation: Implement visibility change detection and use chrome.alarms API for critical keepalive tasks in extensions; document minimum interval requirements
  Owner: Frontend Engineering Team
- Synchronized polling intervals across multiple components could create resource contention spikes
  Mitigation: Introduce random jitter (±10-20%) to polling intervals to distribute load; implement centralized polling coordinator for related tasks
  Owner: Performance Engineering Team
- Memory leaks from uncleaned intervals in single-page applications with dynamic component lifecycles
  Mitigation: Enforce cleanup in component unmount hooks; implement automated leak detection in CI; use WeakRef patterns where appropriate
  Owner: Engineering Team

## Implementation Notes

- Create reusable polling utility functions that encapsulate interval creation, cleanup, and error handling to avoid duplication across modules
- Use React useEffect cleanup functions, Vue onUnmounted hooks, or similar lifecycle methods to ensure intervals are cleared when components are destroyed
- Store interval IDs in component state or class properties to enable programmatic cancellation and debugging
- Consider implementing a PollingManager class that provides start/stop/pause capabilities and centralizes interval configuration
- Add telemetry to track polling frequency, success rates, and timing to identify performance issues in production

## Continuation Context


Verify commands:
- grep -r 'setInterval\|setTimeout' --include='*.ts' --include='*.js' | grep -v 'clearInterval\|clearTimeout' | wc -l
- grep -r 'setInterval' --include='*.ts' --include='*.tsx' -A 10 | grep -c 'clearInterval'
- npm run test -- --grep 'polling\|interval\|keepalive' --reporter json | jq '.tests[] | select(.title | contains("cleanup"))'

Accept when:
- All setInterval and setTimeout calls have corresponding clearInterval/clearTimeout in cleanup paths (component unmount, error handlers, connection close)
- Polling intervals are defined as named constants or configuration values rather than inline magic numbers
- Unit tests exist for polling mechanisms that verify cleanup behavior using fake timers (jest.useFakeTimers or sinon.useFakeTimers)

## Enforcement

- Verified by: ESLint custom rules detecting setInterval/setTimeout without corresponding cleanup
- Verified by: Code review checklist requiring verification of timer cleanup in component lifecycle methods
- Verified by: Automated memory leak detection tests in CI pipeline using heap snapshots
- Verified by: Static analysis tools scanning for timer-related patterns in pull requests
- Violation handling: CI pipeline fails if ESLint rules detect uncleaned timers in new code
- Violation handling: Code review blocks merge until cleanup handlers are added and tested
- Violation handling: Memory leak detection failures trigger automatic rollback and incident review
- Violation handling: Quarterly audits identify and remediate legacy violations with tracked technical debt tickets
- Exception process: Developer submits exception request with justification and alternative mitigation strategy
- Exception process: Tech lead reviews and approves/rejects based on risk assessment and architectural alignment
- Exception process: Approved exceptions are documented in code comments with ADR reference and expiration date
- Exception process: Exception registry is reviewed quarterly to ensure temporary exceptions are resolved