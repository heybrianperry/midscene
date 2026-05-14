# Adopt Event-Driven Logging Pattern for Asynchronous Operations: Event Logging Not

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all event-driven components that perform asynchronous operations requiring observability.

## Context

- The codebase contains event-driven components (launcher, scrcpy-stream, io-client) that handle asynchronous operations and require runtime observability
- Event-driven architectures create non-linear execution flows where traditional sequential logging is insufficient for debugging and monitoring
- Multiple packages (playground, playground-app, web-integration) independently implement similar logging patterns around event boundaries, indicating an emergent architectural need
- The pattern appears in 3 files with 91.97% confidence, suggesting a consistent approach to handling observability in event-driven contexts

## Problem Statement

Event-driven systems with asynchronous operations lack visibility into execution flow, state transitions, and error conditions without structured logging at event boundaries. Traditional sequential logging approaches fail to capture the temporal relationships and causal chains in event-driven architectures, making debugging and operational monitoring difficult.

## Decision

1. MUST_NOT: Event logging MUST NOT block the event loop or introduce synchronous I/O operations that degrade performance

## Policy Block

- MUST_NOT Event logging MUST NOT block the event loop or introduce synchronous I/O operations that degrade performance

In scope:
- Event emitters and event listeners in asynchronous components
- Stream processing pipelines with event-driven data flow
- Bridge and integration layers that mediate between event-driven systems
- Launcher and orchestration components managing lifecycle events

Out of scope:
- Synchronous function calls without event-driven characteristics
- Pure computational logic without external I/O or state changes
- Static configuration loading and initialization code
- Unit test fixtures and mock implementations

Exceptions:
- EXC-001: Performance-critical hot paths where logging overhead exceeds 5% of execution time
- EXC-002: Third-party event emitters where logging cannot be injected without modifying external libraries

## Rationale

- The pattern was detected across 3 independent files in different packages (playground, playground-app, web-integration), indicating convergent evolution toward a common solution for event-driven observability
- Event-driven architectures inherently create temporal and causal complexity that requires structured logging to maintain operational visibility and debuggability
- The 91.97% confidence score suggests this is a well-established pattern in the codebase, not an experimental or inconsistent approach
- Logging at event boundaries provides natural instrumentation points that align with the architectural style while minimizing invasiveness

## Consequences

Positive:
- Improved debuggability of asynchronous operations through correlation of events across temporal boundaries
- Enhanced operational monitoring with visibility into event flow, state transitions, and error conditions
- Consistent logging approach across event-driven components reduces cognitive load for developers
- Structured event logs enable automated analysis, alerting, and performance profiling

Negative:
- Additional logging code increases component complexity and maintenance burden
- Logging overhead may impact performance in high-throughput event processing scenarios
- Excessive logging can generate large volumes of log data requiring storage and management
- Developers must maintain discipline to log consistently at all event boundaries

## Alternatives

- Use distributed tracing framework (OpenTelemetry) instead of custom event logging (rejected)
  Rejected because: Adds significant dependency weight and complexity for a pattern that is already organically established in the codebase; may be overkill for current scale
  When valid: Consider when scaling to microservices architecture or when cross-service tracing becomes critical
- Implement aspect-oriented programming (AOP) to inject logging automatically at event boundaries (rejected)
  Rejected because: TypeScript/JavaScript ecosystem has limited mature AOP support; adds metaprogramming complexity that may obscure code behavior
  When valid: Revisit if a mature, well-supported AOP framework emerges for TypeScript
- No standardized logging pattern; allow each component to implement observability independently (rejected)
  Rejected because: Leads to inconsistent observability across components, making system-wide debugging and monitoring difficult; the detected pattern shows convergence is already happening naturally
  When valid: Never valid for production systems requiring operational support

## Risks

- Performance degradation in high-frequency event processing due to logging overhead
  Mitigation: Implement configurable log levels and sampling for high-volume events; use asynchronous logging backends; profile and optimize hot paths
  Owner: Engineering team with performance monitoring
- Log volume explosion leading to storage costs and difficulty finding relevant information
  Mitigation: Implement log level filtering, retention policies, and structured logging to enable efficient querying; use log aggregation tools with filtering capabilities
  Owner: DevOps and platform engineering team
- Inconsistent implementation across teams leading to gaps in observability coverage
  Mitigation: Provide logging utility libraries and templates; include event logging requirements in code review checklists; document pattern with examples
  Owner: Architecture team and tech leads

## Implementation Notes

- Create a shared logging utility module that provides event-aware logging functions with built-in correlation ID management
- Establish log level conventions: DEBUG for detailed event flow, INFO for lifecycle events, WARN for recoverable issues, ERROR for failures
- Include event type, timestamp, component name, and correlation ID as standard metadata in all event logs
- Consider implementing a logging wrapper or decorator pattern to reduce boilerplate in event handler implementations
- Document the logging pattern with code examples from the detected files (launcher.ts, scrcpy-stream.ts, io-client.ts) as reference implementations

## Continuation Context


Verify commands:
- grep -r "addEventListener\|on(\|emit(" packages/ | wc -l > /tmp/events.txt && grep -r "log\|logger\|console" packages/ | wc -l > /tmp/logs.txt && echo 'Check ratio of logging to events'
- find packages/ -name '*.ts' -exec grep -l 'emit\|addEventListener' {} \; | xargs grep -L 'log\|logger\|console' | wc -l
- npm run lint -- --rule 'no-console: off' && npm test -- --grep 'logging' --reporter json > test-results.json

Accept when:
- All event-driven components identified in scope have logging at event emission and reception points
- Code review checklist includes verification of event boundary logging for new event-driven code
- Automated tests verify that critical events generate expected log entries with required metadata
- Log analysis tools can successfully trace event flows across component boundaries using correlation identifiers

## Enforcement

- Verified by: Code review process with specific checklist item for event-driven logging compliance
- Verified by: Static analysis tools configured to detect event emitters/listeners without associated logging statements
- Verified by: Integration tests that verify log output for critical event flows
- Violation handling: Code review feedback requesting addition of logging at event boundaries before merge approval
- Violation handling: CI pipeline warnings for components with event-driven patterns lacking logging instrumentation
- Violation handling: Quarterly architecture review to identify gaps in observability coverage and remediate
- Exception process: Developer submits exception request with performance profiling data or technical justification
- Exception process: Architecture review board evaluates exception against policy criteria (EXC-001, EXC-002)
- Exception process: Approved exceptions documented in component README with alternative observability approach
- Exception process: Exceptions reviewed annually to determine if conditions have changed