# Adopt Event-Driven Process Lifecycle Management for External Services: Process Managers Provide

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all integration components that manage external process lifecycles (servers, display managers, browser automation). Agents MUST follow these patterns when implementing or modifying integration boundaries.

## Context

- The codebase manages multiple external processes including test servers, demo servers, virtual display managers (Xvfb), and browser automation tools (Playwright)
- These processes require coordinated lifecycle management with proper startup, health checking, graceful shutdown, and error handling
- Integration boundaries between the application and external services need consistent patterns for event-driven communication and state management
- Process management failures can lead to resource leaks, zombie processes, port conflicts, and unreliable test/development environments
- The pattern appears across 4 files with 91.65% confidence, indicating a deliberate architectural approach to external service integration

## Problem Statement

How should the system manage the lifecycle of external processes and services to ensure reliable startup, monitoring, and cleanup while maintaining clear integration boundaries and event-driven communication patterns?

## Decision

1. MUST: Process managers MUST provide health checking mechanisms to verify service readiness before declaring the process available

## Policy Block

- MUST Process managers MUST provide health checking mechanisms to verify service readiness before declaring the process available

In scope:
- Test server lifecycle management (start-server.ts)
- Demo and development server orchestration (demo/server.ts)
- Virtual display manager integration (xvfb.ts)
- Browser automation process management (playwright/page.ts)
- Any external process that requires coordinated startup and shutdown
- Integration boundaries between application code and system services

Out of scope:
- Internal application service lifecycle (in-process services)
- Simple synchronous command execution without lifecycle management
- Static configuration loading
- Pure data transformation functions
- Direct library imports that don't spawn processes

Exceptions:
- EXC-001: Quick-and-dirty scripts or one-off utilities where process cleanup is handled by OS termination
- EXC-002: Legacy integration code scheduled for deprecation within current quarter

## Rationale

- The pattern detected across 4 files (start-server.ts, server.ts, xvfb.ts, page.ts) demonstrates a consistent architectural approach to external process management
- Event-driven lifecycle management provides clear separation of concerns between process orchestration and business logic, improving testability and reliability
- Explicit health checking and readiness signals prevent race conditions where dependent code attempts to use services before they're fully initialized
- Graceful shutdown patterns prevent resource leaks and ensure clean test/development environment teardown, critical for CI/CD reliability

## Consequences

Positive:
- Consistent, predictable behavior across all external service integrations reduces cognitive load and debugging time
- Event-driven patterns enable reactive architectures with better error handling and recovery mechanisms
- Proper lifecycle management prevents resource leaks, zombie processes, and port conflicts in test and development environments
- Clear integration boundaries improve testability through mocking and stubbing of external dependencies
- Asynchronous APIs align with Node.js/TypeScript ecosystem conventions and enable concurrent operations

Negative:
- Increased complexity in simple use cases where synchronous process execution would suffice
- Additional boilerplate code required for each external service integration
- Potential for timeout configuration issues if not tuned appropriately for different service startup times
- Learning curve for developers unfamiliar with event-driven process management patterns
- Debugging async lifecycle issues can be more complex than synchronous alternatives

## Alternatives

- Synchronous process spawning with blocking waits (rejected)
  Rejected because: Blocks event loop in Node.js, prevents concurrent operations, and doesn't align with async ecosystem patterns. Would cause performance degradation in test suites and development workflows.
  When valid: Only appropriate for simple CLI tools with no concurrency requirements
- Fire-and-forget process spawning without lifecycle management (rejected)
  Rejected because: Leads to resource leaks, zombie processes, and unreliable test environments. No mechanism to verify service readiness or coordinate shutdown.
  When valid: Never appropriate in production or test code; only acceptable for throwaway scripts
- Container-based service orchestration (Docker Compose, Kubernetes) (deferred)
  Rejected because: Adds infrastructure complexity and external dependencies. May be appropriate for production but overkill for development/test environments.
  When valid: Consider for production deployments or complex multi-service integration tests where container orchestration benefits outweigh complexity

## Risks

- Timeout values may be insufficient for slow startup services, causing false failures in CI or on slower machines
  Mitigation: Implement configurable timeouts with sensible defaults. Add logging to track actual startup times and adjust thresholds based on metrics.
  Owner: Integration team
- Incomplete cleanup on unexpected termination (crashes, SIGKILL) may leave orphaned processes or locked resources
  Mitigation: Implement process group management and cleanup hooks. Use PID files and startup checks to detect and clean stale processes. Add CI job to verify no leaked processes.
  Owner: DevOps and integration team
- Complex event-driven patterns may introduce race conditions or deadlocks in lifecycle state transitions
  Mitigation: Implement state machine pattern for lifecycle management with clear state transitions. Add comprehensive integration tests covering startup/shutdown sequences and error paths.
  Owner: Engineering team

## Implementation Notes

- Use Node.js child_process.spawn() with event listeners for 'spawn', 'error', 'exit' events to implement lifecycle hooks
- Implement health check polling (HTTP endpoint checks, port availability, or process signal verification) with configurable retry intervals
- Use process.on('exit') and signal handlers (SIGTERM, SIGINT) to register cleanup callbacks for graceful shutdown
- Consider using libraries like 'tree-kill' for reliable process tree cleanup across platforms
- Expose TypeScript interfaces for lifecycle events to ensure type-safe integration patterns
- Document expected startup times and timeout configurations for each external service type

## Continuation Context


Verify commands:
- grep -r "spawn\|exec" packages/*/src packages/*/tests apps/*/src --include="*.ts" | grep -E "(on\('exit'|on\('error'|kill\(|SIGTERM)" || echo 'No lifecycle management found'
- grep -r "class.*Server\|class.*Process\|function.*start.*" packages/*/src --include="*.ts" -A 20 | grep -E "(Promise|async|await|EventEmitter)" || echo 'No async lifecycle patterns found'
- find . -name "*.ts" -path "*/tests/*" -o -path "*/src/*" | xargs grep -l "beforeAll\|afterAll\|beforeEach\|afterEach" | xargs grep -l "start\|stop\|cleanup" | wc -l

Accept when:
- All external process integrations implement async startup with readiness verification (health checks or event signals)
- All process managers implement cleanup handlers that are invoked on process exit or test teardown
- Integration tests demonstrate successful startup, operation, and graceful shutdown of external services without resource leaks
- Code review confirms event-driven patterns with proper error handling for all lifecycle phases

## Enforcement

- Verified by: Code review checklist requiring lifecycle management verification for any PR touching external process integration
- Verified by: CI pipeline checks for process leaks after test suite execution
- Verified by: Static analysis rules detecting spawn/exec calls without corresponding cleanup handlers
- Verified by: Integration test coverage requirements for startup and shutdown paths
- Violation handling: PR blocked if external process integration lacks documented lifecycle management
- Violation handling: CI failure if process leak detection identifies orphaned processes after test runs
- Violation handling: Architecture review required for any new external service integration to verify pattern compliance
- Violation handling: Post-incident review if production issues trace to improper process lifecycle management
- Exception process: Submit exception request to tech lead with justification and risk assessment
- Exception process: Document exception in code comments with reference to approval and timeline
- Exception process: Add technical debt ticket for future remediation if exception is temporary
- Exception process: Review exceptions quarterly to identify patterns requiring policy updates