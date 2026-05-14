# Adopt Vitest as Standard Test Framework for CI/CD Pipeline: Unit Tests Placed

Status: proposed
Date: 2025-01-10
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains multiple test files across packages (playground, chrome-extension) indicating a monorepo structure requiring consistent testing infrastructure
- Test files are organized with unit tests and integration tests (bridge-start-stop, server-session-manager, server-cancel-recreate) suggesting complex async behavior requiring modern test tooling
- The presence of a centralized test setup file (packages/playground/tests/setup.ts) indicates a need for shared test configuration and initialization logic
- The facet 'paradigm.concurrency_model' with 91.17% confidence suggests the pattern relates to how tests handle asynchronous operations and concurrent execution
- Six supporting files across different packages demonstrate consistent adoption of a testing approach that integrates with the CI/CD pipeline

## Problem Statement

The project requires a modern, fast test framework that can handle complex asynchronous operations, support monorepo structures, provide consistent test execution across multiple packages, and integrate seamlessly with CI/CD pipelines while maintaining developer productivity through fast feedback loops.

## Decision

1. SHOULD: Unit tests SHOULD be placed in a tests/unit directory structure to separate them from integration tests

## Policy Block

- SHOULD Unit tests SHOULD be placed in a tests/unit directory structure to separate them from integration tests

In scope:
- All TypeScript test files in the monorepo
- Unit tests for individual components and modules
- Integration tests for cross-component interactions
- Test setup and configuration files
- CI/CD pipeline test execution stages

Out of scope:
- End-to-end tests that may use different frameworks (e.g., Playwright, Cypress)
- Performance benchmarking tests
- Manual testing procedures
- Third-party library tests

## Rationale

- Vitest provides native ESM support, TypeScript integration, and fast execution through Vite's transformation pipeline, making it ideal for modern TypeScript monorepos
- The pattern detection across 6 files with 91.17% confidence indicates this is an established, consistent practice in the codebase
- Centralized test setup enables consistent mocking, environment configuration, and test utilities across all packages
- The concurrency_model facet suggests Vitest's superior handling of async operations and concurrent test execution is a key architectural driver

## Consequences

Positive:
- Fast test execution through Vite's transformation pipeline and smart caching reduces CI/CD pipeline duration
- Native TypeScript support eliminates transpilation complexity and improves developer experience
- Consistent test framework across all packages simplifies onboarding and reduces cognitive overhead
- Built-in support for concurrent test execution improves test suite performance at scale

Negative:
- Teams must learn Vitest-specific APIs and patterns if coming from Jest or other frameworks
- Migration effort required for any existing tests using different frameworks
- Dependency on Vite ecosystem may create coupling for projects not already using Vite
- Smaller community and ecosystem compared to more established frameworks like Jest

## Alternatives

- Jest with ts-jest for TypeScript support (rejected)
  Rejected because: Jest requires additional transpilation overhead with ts-jest, resulting in slower test execution. The codebase pattern shows adoption of Vitest, indicating Jest was evaluated and found insufficient for performance requirements.
  When valid: For projects with extensive Jest infrastructure or requiring Jest-specific ecosystem plugins
- Native Node.js test runner with TypeScript (rejected)
  Rejected because: Node.js native test runner lacks the maturity, ecosystem, and developer experience features needed for complex monorepo testing scenarios
  When valid: For simple projects with minimal testing requirements and no external dependencies
- Mocha with Chai for assertion library flexibility (rejected)
  Rejected because: Mocha requires extensive configuration and lacks built-in TypeScript support, mocking utilities, and modern async handling that Vitest provides out of the box
  When valid: For projects requiring highly customized test configurations or specific assertion library requirements

## Risks

- Vitest API changes or breaking updates could require significant refactoring across all test files
  Mitigation: Pin Vitest version in package.json, establish upgrade testing procedures, and monitor Vitest release notes for breaking changes before upgrading
  Owner: Engineering team
- Performance degradation if tests are not properly structured for concurrent execution
  Mitigation: Establish test isolation guidelines, use proper setup/teardown hooks, and monitor test suite execution times in CI/CD metrics
  Owner: Engineering team
- Inconsistent test patterns across packages could emerge without governance
  Mitigation: Maintain shared test utilities and setup files, conduct code reviews focusing on test quality, and document testing patterns in developer guidelines
  Owner: Engineering team

## Implementation Notes

- Create a shared test setup file (e.g., tests/setup.ts) that configures Vitest globals, mocks, and environment variables for all packages
- Organize tests using the tests/unit and tests/integration directory structure to clearly separate test types
- Use Vitest's workspace feature for monorepo configurations to enable per-package test configuration while maintaining shared defaults
- Configure CI/CD pipelines to run 'vitest run' with appropriate flags (--reporter, --coverage) and fail on test failures or coverage thresholds

## Continuation Context


Verify commands:
- grep -r "from 'vitest'" --include='*.test.ts' || grep -r 'from "vitest"' --include='*.test.ts'
- find . -name 'vitest.config.*' -o -name 'vite.config.*' | xargs grep -l 'test'
- grep -r "describe\|test\|it\|expect" --include='*.test.ts' | head -5

Accept when:
- All test files import test utilities (describe, test, expect) from 'vitest' package
- A vitest.config.ts or vite.config.ts file exists with test configuration
- Test files follow the .test.ts naming convention and are organized in tests/ directories

## Enforcement

- Verified by: CI/CD pipeline executes 'vitest run' command and fails builds on test failures
- Verified by: Code review process verifies new tests follow Vitest patterns and naming conventions
- Verified by: Linting rules enforce test file naming conventions and import patterns
- Violation handling: CI/CD pipeline blocks merges if tests fail or are missing for new code
- Violation handling: Code review feedback requests corrections for non-compliant test patterns
- Violation handling: Automated linting failures prevent commits with incorrect test file naming
- Exception process: Exceptions for alternative test frameworks must be documented in ADR with technical justification
- Exception process: Architecture review board approval required for package-level test framework deviations
- Exception process: Temporary exceptions during migration periods must include completion timeline and tracking