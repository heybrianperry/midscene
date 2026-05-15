# Standardize Browser and Platform API Abstraction Layers for External Integration: Application Logic Not

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The codebase integrates with multiple external platform APIs including Chrome Extension APIs, browser DOM APIs, and native system interfaces that have varying contracts and stability guarantees
- Components across packages (chrome-extension, visualizer, playground, cli) require consistent patterns for accessing platform-specific functionality while maintaining testability and portability
- Direct coupling to platform APIs creates maintenance burden when APIs change, makes testing difficult, and reduces code reusability across different runtime environments
- The pattern appears in 18 files with 89.07% confidence, indicating a systematic approach to abstracting external API dependencies through wrapper utilities and interface layers
- Cross-platform support requirements (browser extensions, web applications, CLI tools, native components) necessitate a unified strategy for managing external API contracts

## Problem Statement

How should the codebase consistently abstract and manage dependencies on external platform APIs (browser APIs, Chrome Extension APIs, native system APIs) to ensure maintainability, testability, and portability across different runtime environments while preserving type safety and developer ergonomics?

## Decision

1. MUST_NOT: Application logic MUST NOT directly reference global platform objects (window.chrome, process, etc.) outside of designated abstraction modules

## Policy Block

- MUST_NOT Application logic MUST NOT directly reference global platform objects (window.chrome, process, etc.) outside of designated abstraction modules

In scope:
- All Chrome Extension API usage (chrome.*, browser.*)
- Browser DOM APIs accessed from application code (window, document, navigator)
- Node.js platform APIs used in CLI and build tools (process, fs, path)
- Native system APIs exposed through FFI or bindings (RDP, system calls)
- Third-party platform SDKs that provide external integration points

Out of scope:
- Standard JavaScript/TypeScript language features (Promise, Array, Object)
- Internal application APIs and service interfaces
- NPM package imports that provide cross-platform abstractions
- Test framework APIs (Jest, Playwright) used exclusively in test code
- Build tool configurations and scripts

Exceptions:
- EXC-001: Platform detection or feature detection code that must check for API existence
- EXC-002: Type declarations or ambient type definitions for platform APIs

## Rationale

- The pattern detected across 18 files with 89.07% confidence demonstrates a consistent architectural approach to managing external API dependencies, indicating this is an established practice worth codifying
- Abstraction layers enable testing by allowing mock implementations, reduce coupling to specific platform versions, and provide a single point of control for handling API changes or deprecations
- Cross-platform requirements across chrome-extension, visualizer, playground, and CLI packages necessitate a unified strategy to avoid duplicating platform-specific logic throughout the codebase
- Type-safe wrappers improve developer experience by providing better IDE support, compile-time error checking, and documentation compared to raw platform API usage

## Consequences

Positive:
- Improved testability through dependency injection and mock implementations of platform APIs
- Reduced maintenance burden when platform APIs change, as updates are localized to abstraction modules
- Enhanced portability enabling code reuse across different runtime environments (browser, Node.js, native)
- Better error handling and debugging through centralized API access points with consistent error reporting
- Stronger type safety and IDE support through well-defined TypeScript interfaces

Negative:
- Additional indirection layer increases initial development complexity and requires maintaining wrapper code
- Potential performance overhead from abstraction layers, though typically negligible for I/O-bound operations
- Risk of abstraction leakage where platform-specific details still propagate through the wrapper interfaces
- Learning curve for developers who must understand both the platform APIs and the abstraction layer conventions

## Alternatives

- Direct platform API usage throughout the codebase without abstraction layers (rejected)
  Rejected because: Creates tight coupling to platform APIs, makes testing extremely difficult, and spreads platform-specific logic throughout the codebase making it hard to maintain or port
  When valid: Only appropriate for small, single-platform prototypes with no testing requirements
- Use dependency injection framework to inject platform API instances at runtime (rejected)
  Rejected because: Adds significant complexity and framework dependency for a problem that can be solved with simpler module-level abstractions; overkill for most use cases
  When valid: May be appropriate for large enterprise applications with complex runtime configuration needs
- Adopt existing cross-platform abstraction libraries (e.g., webextension-polyfill for browser APIs) (accepted)
  When valid: Should be used when mature, well-maintained libraries exist for the specific platform API being abstracted

## Risks

- Abstraction layers may not keep pace with platform API evolution, causing feature lag or requiring frequent updates
  Mitigation: Establish regular review cycles for platform API changes; design abstractions to be extensible; document version compatibility requirements
  Owner: Platform Integration Team
- Over-abstraction could hide important platform-specific behaviors or constraints that developers need to understand
  Mitigation: Document platform-specific behaviors in wrapper interfaces; provide escape hatches for advanced use cases; maintain clear mapping to underlying platform APIs
  Owner: Engineering Team
- Inconsistent abstraction patterns across different packages could lead to fragmentation and confusion
  Mitigation: Establish shared abstraction patterns in a common package; conduct code reviews focusing on API design consistency; provide templates and examples
  Owner: Architecture Team

## Implementation Notes

- Create dedicated utility modules (e.g., utils/chrome.ts, utils/platform.ts) in each package that needs platform API access, following consistent naming conventions
- Use TypeScript interfaces to define abstraction contracts, enabling easy mocking in tests and clear documentation of expected behaviors
- Implement feature detection at module initialization time and cache results to avoid repeated checks; provide clear error messages when required APIs are unavailable
- For Chrome Extension APIs, consider using webextension-polyfill or similar libraries to normalize differences between Chrome and Firefox implementations
- Document the mapping between abstraction methods and underlying platform APIs to help developers understand the relationship and debug issues

## Continuation Context


Verify commands:
- grep -r 'window\.chrome\|global\.chrome' --include='*.ts' --include='*.tsx' --exclude-dir=node_modules --exclude='**/utils/chrome.ts' . | grep -v 'typeof.*chrome' | wc -l | awk '{if ($1 > 0) exit 1}'
- grep -r 'process\.' --include='*.ts' --exclude-dir=node_modules --exclude='**/utils/*.ts' --exclude='**/*.config.*' . | wc -l | awk '{if ($1 > 10) exit 1}'
- find . -name 'chrome.ts' -o -name 'platform.ts' -o -name '*-api.ts' | grep utils | wc -l | awk '{if ($1 < 1) exit 1}'

Accept when:
- No direct references to platform globals (window.chrome, process, etc.) exist outside of designated abstraction utility modules
- At least one platform API abstraction module exists in packages that integrate with external APIs
- All platform API abstractions provide TypeScript type definitions and handle availability checks

## Enforcement

- Verified by: Automated linting rules checking for direct platform API usage outside abstraction modules
- Verified by: Code review checklist items requiring abstraction layer usage for new platform integrations
- Verified by: CI pipeline verification commands checking for policy violations
- Violation handling: CI build fails if direct platform API usage is detected outside approved abstraction modules
- Violation handling: Pull requests blocked until platform API access is properly abstracted
- Violation handling: Existing violations tracked as technical debt items with remediation timeline
- Exception process: Developer submits exception request with justification to tech lead
- Exception process: Tech lead reviews for legitimate cases (feature detection, type definitions)
- Exception process: Approved exceptions documented in code with comments explaining rationale
- Exception process: Exception list reviewed quarterly to identify patterns requiring policy updates