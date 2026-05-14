# Standardize Unit Test Structure with Vitest for TypeScript Packages: Test Files Mirror

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all TypeScript package development within the monorepo and applies to all unit test implementations in CI/CD pipelines.

## Context

- The codebase contains multiple TypeScript packages (playground, cli) that require consistent testing approaches for maintainability and reliability
- Unit tests are critical components of CI/CD pipelines and must execute reliably across different package contexts
- A standardized testing pattern was detected across 3 files with 91.40% confidence, indicating an established practice
- The monorepo structure requires consistent test organization to enable efficient test execution and debugging
- Test files follow a naming convention (*.test.ts) and are organized in tests/unit or tests/unit-test directories

## Problem Statement

Without a standardized unit testing structure and framework across TypeScript packages, teams face inconsistent test execution, difficult maintenance, unreliable CI/CD pipelines, and reduced confidence in test coverage. The lack of uniformity creates friction when developers move between packages and makes it harder to establish quality gates.

## Decision

1. SHOULD: Test files SHOULD mirror the structure and naming of the source files they test (e.g., playground.ts → playground.test.ts)

## Policy Block

- SHOULD Test files SHOULD mirror the structure and naming of the source files they test (e.g., playground.ts → playground.test.ts)

In scope:
- All TypeScript packages within the monorepo
- Unit tests for business logic, utilities, and service classes
- CI/CD pipeline test execution stages
- Local development test runs
- Pre-commit and pre-push test hooks

Out of scope:
- Integration tests that require external services
- End-to-end tests that test full application flows
- Performance and load testing
- Manual testing procedures
- Non-TypeScript packages or legacy code not yet migrated

Exceptions:
- EXC-001: Legacy packages undergoing gradual migration may temporarily use alternative testing frameworks
- EXC-002: Specialized testing requirements (e.g., browser-specific tests) require framework-specific tooling

## Rationale

- Pattern detected across 3 files with 91.40% confidence indicates this is an established and proven practice in the codebase
- Vitest provides excellent TypeScript support, fast execution, and compatibility with modern JavaScript tooling
- Consistent test structure reduces cognitive load when developers work across multiple packages
- Standardized naming conventions enable automated test discovery and execution in CI/CD pipelines
- The tests/unit directory structure clearly separates unit tests from other test types, improving organization and execution targeting

## Consequences

Positive:
- Developers can quickly understand and write tests following established patterns across all packages
- CI/CD pipelines can reliably discover and execute tests with consistent commands
- Test execution time is optimized through Vitest's fast, parallel test runner
- Improved code quality through consistent testing practices and easier test maintenance
- Better developer experience with hot module reloading and watch mode during development

Negative:
- Teams already using alternative frameworks (Jest, Mocha) will need to migrate existing tests
- Learning curve for developers unfamiliar with Vitest, though it's similar to Jest
- Additional configuration and setup required for new packages to conform to the standard
- Potential limitations if Vitest doesn't support specific edge cases or testing scenarios

## Alternatives

- Use Jest as the standard testing framework (rejected)
  Rejected because: Jest has slower startup times and less optimal TypeScript support compared to Vitest. The detected pattern shows Vitest is already established in the codebase.
  When valid: For packages with extensive existing Jest test suites where migration cost outweighs benefits
- Allow each package to choose its own testing framework (rejected)
  Rejected because: This approach leads to inconsistency, increased maintenance burden, and difficulty in establishing organization-wide testing standards
  When valid: Never recommended for a monorepo structure where consistency is critical
- Use native Node.js test runner (rejected)
  Rejected because: Native test runner lacks maturity, ecosystem support, and advanced features like mocking and coverage that Vitest provides
  When valid: For simple packages with minimal testing requirements and no external dependencies

## Risks

- Migration effort for existing test suites using different frameworks may be underestimated
  Mitigation: Create migration guides, provide examples, and allocate dedicated time for test migration in sprint planning
  Owner: Engineering Team Leads
- Vitest may have breaking changes or lack support for specific testing scenarios
  Mitigation: Pin Vitest versions in package.json, monitor release notes, and maintain fallback documentation for edge cases
  Owner: Platform Engineering Team
- Inconsistent adoption across teams may lead to partial implementation
  Mitigation: Enforce through CI/CD checks, code review guidelines, and automated linting rules that verify test structure
  Owner: DevOps and Engineering Managers

## Implementation Notes

- Install Vitest as a dev dependency in each package: npm install -D vitest
- Create or update vitest.config.ts in package root with appropriate TypeScript and coverage settings
- Configure package.json test script to run 'vitest run' for CI and 'vitest' for local development with watch mode
- Use the detected pattern files (playground.test.ts, batch-runner.test.ts, server-session-manager.test.ts) as reference implementations
- Set up coverage thresholds in vitest.config.ts to maintain minimum coverage requirements (e.g., 80% line coverage)
- Integrate test execution into CI/CD pipeline with appropriate failure conditions and reporting

## Continuation Context


Verify commands:
- find packages -name '*.test.ts' -type f | grep -E 'tests/(unit|unit-test)/' | wc -l
- grep -r "from 'vitest'" packages/*/tests/unit*/*.test.ts | wc -l
- find packages -name 'vitest.config.ts' -o -name 'vitest.config.js' | wc -l

Accept when:
- All TypeScript packages contain unit tests in tests/unit or tests/unit-test directories with .test.ts extensions
- Vitest imports are present in test files and vitest.config files exist in package roots
- CI/CD pipeline successfully executes test commands and reports results for all packages
- Test coverage reports are generated and meet minimum threshold requirements

## Enforcement

- Verified by: Automated CI/CD pipeline checks that verify test execution and coverage
- Verified by: Code review process that validates test structure and naming conventions
- Verified by: ESLint or custom linting rules that enforce test file naming and location
- Verified by: Pre-commit hooks that run unit tests and verify they pass
- Violation handling: CI/CD pipeline fails if tests are not in the correct directory structure
- Violation handling: Pull requests are blocked if test coverage falls below threshold
- Violation handling: Automated comments on PRs highlight non-compliant test files
- Violation handling: Build fails if Vitest is not configured or test scripts are missing
- Exception process: Developer submits exception request to Tech Lead with justification and impact analysis
- Exception process: Tech Lead reviews and either approves with conditions or requests alternative approach
- Exception process: Approved exceptions are documented in package README with expiration date or migration plan
- Exception process: Exceptions are reviewed quarterly to ensure they remain valid and necessary