# Standardize console.log for Development Logging and Debugging: Use Console Debug

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase spans multiple runtime environments including Chrome extensions, CLI tools, visualizer components, and native build scripts, each requiring different logging approaches
- Development and debugging workflows require immediate visibility into application state, errors, and execution flow without complex infrastructure setup
- The pattern appears consistently across 10 files with 91.26% confidence, indicating an established practice rather than isolated usage
- Browser-based components (Chrome extension, visualizer) and Node.js components (CLI, build scripts) both rely on console-based logging as the primary observability mechanism
- The detected pattern shows console.log usage in critical paths including state management, event recording, bridge communication, and build processes

## Problem Statement

Teams need a lightweight, universally available logging mechanism that works across browser and Node.js environments without requiring external dependencies or infrastructure, while maintaining consistency in how diagnostic information is captured during development and debugging activities.

## Decision

1. MAY: Use console.debug for verbose diagnostic information that is typically filtered out in production builds

## Policy Block

- MAY Use console.debug for verbose diagnostic information that is typically filtered out in production builds

In scope:
- Chrome extension components (store, worker scripts, event recorders, bridge connectors)
- CLI tools and configuration factories
- Visualizer UI components and service controls
- Recorder timeline components
- Build scripts and native compilation processes
- Development and debugging workflows

Out of scope:
- Production logging infrastructure requiring persistence, aggregation, or analysis
- Structured logging for monitoring and alerting systems
- Audit logging for compliance or security requirements
- Performance metrics and telemetry collection
- Third-party library logging configurations

Exceptions:
- EXC-001: Production diagnostic logging is required for critical error paths that cannot be captured through other means
- EXC-002: Build scripts and tooling require persistent console output for CI/CD pipeline visibility

## Rationale

- Console logging is universally available across all JavaScript runtime environments without requiring additional dependencies or configuration
- The pattern detection shows 91.26% confidence across 10 diverse files, indicating this is an established and effective practice within the codebase
- Browser DevTools and Node.js console provide rich inspection capabilities for logged objects, making console.log highly effective for development debugging
- The lightweight nature of console logging enables rapid development iteration without the overhead of configuring structured logging frameworks

## Consequences

Positive:
- Zero-dependency logging solution that works immediately in any JavaScript environment
- Developers can quickly add diagnostic output without learning framework-specific APIs or configuration
- Browser DevTools and Node.js provide powerful filtering, searching, and inspection capabilities for console output
- Consistent logging approach across browser extensions, CLI tools, and UI components reduces cognitive overhead

Negative:
- Console logs are ephemeral and not persisted, making post-mortem debugging of production issues difficult
- No structured logging format makes automated parsing and analysis challenging
- Performance impact if excessive logging is left enabled in production builds
- Lack of log levels and filtering at the application layer requires manual filtering in console tools

## Alternatives

- Adopt a structured logging framework like Winston or Pino for all components (rejected)
  Rejected because: Adds significant dependency overhead and configuration complexity for browser-based components; overkill for development debugging needs
  When valid: Consider for backend services or when persistent, structured logs are required for production monitoring
- Implement a custom logging abstraction layer that wraps console methods (deferred)
  Rejected because: Would provide better control over log levels and formatting but requires upfront development effort
  When valid: Revisit if the codebase grows to require centralized log management, filtering, or production log collection
- Use browser extension-specific logging APIs or Node.js debug module (rejected)
  Rejected because: Creates environment-specific logging implementations that reduce code portability and increase maintenance burden
  When valid: Use for environment-specific features that require specialized logging capabilities not available through console

## Risks

- Sensitive information may be accidentally logged to console and exposed in production environments or shared screenshots
  Mitigation: Implement code review guidelines to check for sensitive data logging; use linting rules to detect common patterns of credential logging
  Owner: Engineering team
- Excessive console logging in production builds may impact performance, especially in tight loops or high-frequency operations
  Mitigation: Use build-time dead code elimination to remove debug logs; establish guidelines for conditional logging based on environment flags
  Owner: Engineering team
- Lack of structured logging makes it difficult to diagnose production issues or perform log analysis at scale
  Mitigation: Document that console.log is for development only; plan migration to structured logging for production-critical paths as the system matures
  Owner: Engineering lead

## Implementation Notes

- Establish naming conventions for log message prefixes (e.g., '[StoreName]', '[Worker]', '[Bridge]') to enable easy filtering in console output
- Configure build tools (webpack, rollup, etc.) to strip console.log statements in production builds using plugins like terser or babel-plugin-transform-remove-console
- Use environment variables or feature flags to conditionally enable verbose logging in development: if (process.env.DEBUG) console.log(...)
- Document common debugging patterns in the project README, including how to enable verbose logging and filter console output effectively

## Continuation Context


Verify commands:
- grep -r 'console\.log' apps/chrome-extension/src packages/cli/src packages/visualizer/src packages/recorder/src --include='*.ts' --include='*.tsx' | wc -l
- grep -r 'console\.log.*password\|console\.log.*token\|console\.log.*secret' . --include='*.ts' --include='*.tsx' || echo 'No sensitive data logging detected'
- npm run build 2>&1 | grep -i 'console' || echo 'Build completed without console warnings'

Accept when:
- Console.log statements are present in the detected files (chrome-extension, CLI, visualizer, recorder, build scripts) for development debugging
- No console.log statements contain obvious sensitive information patterns (passwords, tokens, secrets, API keys)
- Production builds either strip console statements or include only intentional operational logging with documented justification

## Enforcement

- Verified by: Code review process checks for sensitive information in console.log statements
- Verified by: Build configuration verification ensures production builds strip or minimize console output
- Verified by: Periodic grep audits of console.log usage to identify patterns that should be removed or refactored
- Violation handling: Console.log statements containing sensitive data must be removed immediately upon discovery
- Violation handling: Excessive console logging impacting production performance triggers refactoring to conditional or structured logging
- Violation handling: Violations identified in code review require revision before merge approval
- Exception process: Request exception through pull request comments with explicit justification for production console logging
- Exception process: Engineering lead reviews and approves exceptions based on operational necessity
- Exception process: Approved exceptions must include inline code comments documenting the rationale and expected output