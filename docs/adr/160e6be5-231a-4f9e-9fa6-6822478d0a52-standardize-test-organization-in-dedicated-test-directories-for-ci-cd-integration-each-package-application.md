# Standardize Test Organization in Dedicated Test Directories for CI/CD Integration: Each Package Application

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains multiple packages and applications with test files organized in dedicated test directories (tests/unit/, tests/), indicating a deliberate organizational pattern
- Test files are consistently named with .test.ts extension and placed in structured test directories rather than co-located with source files
- This pattern appears across different service boundaries (playground package, chrome-extension app, web-bridge-mcp package), suggesting a cross-cutting organizational standard
- The pattern supports CI/CD pipeline integration by providing predictable test file locations that can be easily discovered and executed by automated test runners
- Evidence shows 3 distinct test files across different service boundaries with 90.57% confidence, indicating this is an established architectural pattern rather than coincidental organization

## Problem Statement

Without a standardized approach to test file organization and naming conventions, CI/CD pipelines struggle to reliably discover and execute tests across multiple packages and applications. Inconsistent test organization leads to fragile build configurations, missed test executions, and difficulty maintaining automated quality gates across service boundaries.

## Decision

1. MUST: Each package and application MUST maintain its own tests/ directory at the package root level

## Policy Block

- MUST Each package and application MUST maintain its own tests/ directory at the package root level

In scope:
- All TypeScript test files across all packages and applications
- Unit tests, integration tests, and end-to-end tests
- Test files for both frontend and backend services
- Test files in monorepo packages (packages/*) and applications (apps/*)
- Stress tests, performance tests, and other specialized test types

Out of scope:
- Test fixtures and test data files (may be co-located with tests or in separate fixtures/ directories)
- Mock implementations and test utilities (may follow different organization patterns)
- Documentation examples that include code snippets
- Build scripts and CI/CD configuration files
- Third-party test libraries and dependencies

## Rationale

- Pattern detected across 3 distinct service boundaries (playground package, chrome-extension app, web-bridge-mcp package) with 90.57% confidence, indicating this is an established architectural standard
- Dedicated test directories enable CI/CD pipelines to use simple glob patterns (tests/**/*.test.ts) for reliable test discovery without complex configuration
- Separating tests from source code provides clear boundaries for code coverage analysis and allows different build optimization strategies for production vs. test code
- Consistent naming conventions (.test.ts) align with industry-standard test runner expectations (Jest, Vitest, Mocha) and enable zero-configuration test execution

## Consequences

Positive:
- CI/CD pipelines can reliably discover and execute all tests using simple, consistent glob patterns across all packages
- Test organization scales naturally as the codebase grows, with clear conventions for where to place new tests
- Build tools can easily exclude test code from production bundles by ignoring tests/ directories
- Developers can quickly locate tests for any component by following predictable directory structure conventions

Negative:
- Test files are physically separated from source files, requiring developers to navigate between directories during development
- Refactoring or moving source files requires corresponding updates to test file locations to maintain naming conventions
- Additional directory structure adds slight complexity to project setup and scaffolding
- Co-located test patterns (popular in some frameworks) cannot be used, limiting flexibility in test organization approaches

## Alternatives

- Co-locate test files with source files using .test.ts suffix in the same directory as implementation files (rejected)
  Rejected because: Co-location makes it harder to exclude tests from production builds, complicates CI/CD glob patterns, and mixes concerns in source directories. The detected pattern shows deliberate separation across multiple packages.
  When valid: May be valid for small single-package projects or when using build tools with sophisticated test exclusion capabilities
- Use a single centralized tests/ directory at the monorepo root for all packages and applications (rejected)
  Rejected because: Centralized tests break package encapsulation, make it difficult to run tests for individual packages, and complicate dependency management. Evidence shows each package maintains its own tests/ directory.
  When valid: May be valid for very small monorepos with tightly coupled packages that are always tested together
- Use __tests__ directory convention (React/Jest standard) instead of tests/ directory (rejected)
  Rejected because: While __tests__ is a valid convention, the detected pattern consistently uses tests/ directories. Changing would break existing CI/CD configurations and developer expectations.
  When valid: Valid for new projects or when migrating to React-centric tooling that expects __tests__ directories

## Risks

- Developers may accidentally place test files in incorrect locations, causing tests to be skipped by CI/CD pipelines
  Mitigation: Implement linting rules or pre-commit hooks that verify test file locations match the required pattern. Add CI checks that fail if .test.ts files are found outside tests/ directories.
  Owner: Engineering team, DevOps team
- Inconsistent test directory naming (tests vs __tests__ vs test) across packages could fragment the pattern
  Mitigation: Document the standard in project README and contribution guidelines. Use scaffolding tools that automatically create correct directory structure for new packages.
  Owner: Engineering team, Platform team
- Large test suites in dedicated directories may become difficult to navigate without clear subdirectory organization
  Mitigation: Establish subdirectory conventions (unit/, integration/, e2e/) early and enforce through documentation and code review. Consider tooling to generate test directory indexes.
  Owner: Engineering team

## Implementation Notes

- Use glob patterns like 'packages/*/tests/**/*.test.ts' and 'apps/*/tests/**/*.test.ts' in CI/CD pipeline configurations to discover all tests
- Configure test runners (Jest, Vitest) to look for test files in tests/ directories with .test.ts extension by default
- Create package scaffolding templates that include tests/unit/ and tests/integration/ directories to establish the pattern for new packages
- Update tsconfig.json to exclude tests/ directories from production builds while including them in test compilation
- Consider adding ESLint rules to enforce test file naming conventions and directory placement

## Continuation Context


Verify commands:
- find . -name '*.test.ts' -not -path '*/tests/*' -not -path '*/node_modules/*' | wc -l | grep -q '^0$'
- find packages/*/tests apps/*/tests -name '*.test.ts' 2>/dev/null | wc -l
- grep -r 'testMatch\|testRegex' package.json tsconfig.json jest.config.* vitest.config.* 2>/dev/null

Accept when:
- All .test.ts files are located within tests/ directories (no test files found outside tests/ directories)
- Each package and application has its own tests/ directory at the package root level
- Test runner configurations reference the tests/ directory pattern for test discovery

## Enforcement

- Verified by: CI/CD pipeline test discovery phase verifies expected test files are found
- Verified by: Pre-commit hooks check test file locations against allowed patterns
- Verified by: Code review checklist includes verification of test file placement
- Verified by: Automated linting rules flag test files in incorrect locations
- Violation handling: CI build fails if test files are found outside tests/ directories
- Violation handling: Pre-commit hooks block commits containing misplaced test files
- Violation handling: Code review process requires correction before merge approval
- Violation handling: Automated PR comments flag violations with remediation guidance
- Exception process: Exceptions require documentation in ADR or package-specific README explaining rationale
- Exception process: Platform team approval required for package-level exceptions to test organization standards
- Exception process: Exception requests must include impact analysis on CI/CD pipeline reliability
- Exception process: Approved exceptions must be documented in .adrignore or similar configuration file