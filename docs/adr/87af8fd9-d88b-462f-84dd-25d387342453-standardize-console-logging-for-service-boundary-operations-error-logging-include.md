# Standardize Console Logging for Service Boundary Operations: Error Logging Include

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The codebase spans multiple service boundaries including chrome extensions, recorders, visualizers, and playground servers, each requiring operational visibility
- Service definitions and boundaries require consistent logging patterns to enable debugging across distributed components and asynchronous operations
- Console-based logging has been adopted as a lightweight observability mechanism across 5 files with 90.30% pattern consistency
- The pattern emerged organically in service boundary code where traditional debugging tools are limited (browser extensions, web workers, server processes)
- Development and production environments require different levels of logging verbosity while maintaining consistent log structure

## Problem Statement

Without standardized logging practices at service boundaries, developers face inconsistent observability across components, making it difficult to trace operations through chrome extensions, recording services, visualization components, and server processes. The lack of structured logging conventions leads to debugging challenges and operational blind spots in distributed system interactions.

## Decision

1. SHOULD: Error logging SHOULD include stack traces and error context to facilitate root cause analysis

## Policy Block

- SHOULD Error logging SHOULD include stack traces and error context to facilitate root cause analysis

In scope:
- Chrome extension background workers and service workers
- Recorder timeline components and state management
- IndexedDB operations and data persistence layers
- Visualizer playground components with user interactions
- Server-side playground and development servers
- Cross-boundary communication and message passing
- Asynchronous operations and promise chains at service boundaries

Out of scope:
- Internal utility functions without service boundary implications
- Pure computational logic without side effects
- Third-party library internals
- Performance-critical hot paths where logging overhead is prohibitive
- Unit test code and test fixtures

Exceptions:
- EXC-001: Performance profiling indicates console logging causes measurable performance degradation in production
- EXC-002: Service operates in security-sensitive context where console output could leak information

## Rationale

- Pattern detected across 5 files with 90.30% confidence indicates strong organic adoption of console logging at service boundaries
- Console logging provides immediate visibility in browser developer tools, Node.js processes, and chrome extension debugging contexts without additional infrastructure
- Service boundary operations (workers, servers, database operations) are inherently asynchronous and distributed, requiring explicit logging for observability
- The lightweight nature of console logging aligns with development workflow while providing production debugging capabilities when needed

## Consequences

Positive:
- Improved debugging experience across service boundaries with consistent log patterns
- Faster incident resolution through traceable operations in chrome extensions, recorders, and servers
- Lower barrier to observability without requiring complex logging infrastructure
- Enhanced developer productivity with immediate feedback in browser and Node.js consoles
- Consistent pattern across codebase reduces cognitive load when switching between components

Negative:
- Console logging may introduce performance overhead in high-frequency operations if not managed carefully
- Production console logs can accumulate and impact browser memory if not properly controlled
- Lack of structured logging format may make automated log analysis more difficult
- Console output is ephemeral and not persisted for long-term analysis without additional tooling
- Risk of accidentally logging sensitive information if developers are not vigilant

## Alternatives

- Implement structured logging library (Winston, Pino, or similar) across all service boundaries (rejected)
  Rejected because: Adds dependency overhead and complexity for browser-based components (chrome extensions, visualizers) where console logging is native and sufficient. Structured logging libraries are better suited for pure server-side environments.
  When valid: Consider for server-side services if log aggregation and analysis requirements grow beyond console capabilities
- Use distributed tracing system (OpenTelemetry, Jaeger) for service boundary observability (rejected)
  Rejected because: Significant infrastructure and instrumentation overhead for a development-focused codebase. Tracing systems require backend services and add complexity disproportionate to current needs.
  When valid: Adopt when system scales to production with multiple deployed services requiring end-to-end request tracing
- No standardized logging, rely on debugger breakpoints and ad-hoc logging (rejected)
  Rejected because: Inconsistent observability makes debugging across service boundaries difficult. Debuggers are not always available in production-like environments (chrome extensions, web workers). Ad-hoc logging leads to pattern fragmentation.
  When valid: Never valid for service boundary code; may be acceptable for isolated pure functions

## Risks

- Developers may accidentally log sensitive user data or authentication tokens in console output
  Mitigation: Implement code review checklist for logging statements. Add linting rules to detect common sensitive data patterns. Provide developer training on secure logging practices.
  Owner: Engineering team with security team oversight
- Excessive console logging in production may degrade performance or consume browser memory
  Mitigation: Implement environment-based log level controls. Add performance monitoring to detect logging overhead. Document guidelines for conditional logging in hot paths.
  Owner: Engineering team
- Console-only logging provides no persistence for post-mortem analysis of production issues
  Mitigation: Document that console logging is for development and immediate debugging. Plan migration path to persistent logging for production services. Consider browser extension APIs for error reporting.
  Owner: Engineering team and DevOps

## Implementation Notes

- Start by auditing existing console.log statements in the 5 identified files to establish baseline patterns and conventions
- Create a lightweight logging utility wrapper that enforces consistent format (timestamp, component name, log level, message) while maintaining console simplicity
- Add environment variable support (DEBUG, LOG_LEVEL) to control logging verbosity in development vs production builds
- Document logging conventions in developer guide with examples for each service boundary type (worker, server, database, UI component)
- Implement ESLint rules to warn on console.log in production code without appropriate guards or context
- For chrome extension workers, leverage chrome.runtime.lastError patterns alongside console logging for error handling

## Continuation Context


Verify commands:
- grep -r 'console\.(log|error|warn|info)' apps/chrome-extension/src/scripts/ packages/recorder/src/ packages/visualizer/src/ packages/playground/src/ | wc -l
- grep -r 'console\.log.*password\|console\.log.*token\|console\.log.*secret' apps/ packages/ || echo 'No sensitive data patterns found'
- find apps/chrome-extension packages/recorder packages/visualizer packages/playground -name '*.ts' -o -name '*.tsx' | xargs grep -l 'console\.' | wc -l

Accept when:
- Console logging statements are present in service boundary files (worker.ts, RecordTimeline.tsx, indexedDB.ts, universal-playground/index.tsx, server.ts) with appropriate context
- No console.log statements contain obvious sensitive data patterns (password, token, secret, apiKey)
- At least 80% of service boundary operations include logging for initialization, errors, or state transitions
- Code review checklist includes verification of logging practices and sensitive data exclusion

## Enforcement

- Verified by: Code review process checks for appropriate logging at service boundaries
- Verified by: ESLint rules flag console statements without context or in sensitive areas
- Verified by: Automated grep-based checks in CI pipeline verify no sensitive data patterns in logs
- Verified by: Manual audit of new service boundary code during architecture review
- Violation handling: Code review blocks merge if logging exposes sensitive data
- Violation handling: ESLint warnings for console statements without context require justification or fix
- Violation handling: Post-merge violations trigger immediate remediation ticket
- Violation handling: Repeated violations result in developer training on secure logging practices
- Exception process: Developer documents performance impact or technical constraint preventing standard logging
- Exception process: Team lead reviews exception request and approves alternative observability approach
- Exception process: Exception documented in code comments with ADR reference and justification
- Exception process: Exceptions reviewed quarterly to assess if constraints still apply