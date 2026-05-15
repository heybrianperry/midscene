# Standardize Test File Naming with .test.ts Extension for TypeScript Tests: Test Discovery Patterns

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all TypeScript test files in the codebase. All test discovery, execution, and tooling configurations must recognize and process .test.ts files.

## Context

- The codebase contains TypeScript test files across multiple packages including CLI tools, playground environments, and browser extensions
- Test files need consistent naming conventions to enable automated test discovery by CI/CD pipelines and test runners
- The pattern shows consistent use of .test.ts extension across different project contexts (unit tests, integration tests, stress tests)
- Test file naming directly impacts build tooling configuration, IDE test runner integration, and developer workflow efficiency
- Without standardized naming, test discovery becomes fragile and requires manual configuration updates

## Problem Statement

Test files scattered across a multi-package monorepo require a consistent, predictable naming convention that enables automated test discovery by CI/CD pipelines, test runners, and development tools without manual configuration for each new test file or package.

## Decision

1. SHOULD: Test discovery patterns in CI/CD configurations SHOULD use glob patterns like **/*.test.ts to automatically discover all test files

## Policy Block

- SHOULD Test discovery patterns in CI/CD configurations SHOULD use glob patterns like **/*.test.ts to automatically discover all test files

In scope:
- All TypeScript test files in packages/cli/tests/
- All TypeScript test files in packages/playground/tests/
- All TypeScript test files in apps/chrome-extension/tests/
- Unit tests, integration tests, and stress tests written in TypeScript
- Test runner configurations (Jest, Vitest, Mocha, etc.) that process TypeScript tests

Out of scope:
- JavaScript test files (.test.js) which may follow different conventions
- React component test files that require .test.tsx extension
- End-to-end test files that may use framework-specific naming (e.g., .e2e.ts, .spec.ts for Playwright)
- Test fixture files, test utilities, or test helper modules that are not executable tests
- Documentation or example files that contain test-like code but are not part of the test suite

Exceptions:
- EXC-001: React component tests require JSX/TSX syntax
- EXC-002: Third-party testing framework mandates specific naming convention

## Rationale

- The detected pattern shows consistent adoption of .test.ts naming across 3 different packages with 92.07% confidence, indicating this is an established convention
- Standardized test file naming enables simple, maintainable glob patterns for test discovery in CI/CD pipelines without package-specific configuration
- The .test.ts extension is widely recognized by TypeScript testing tools (Jest, Vitest, Mocha) and IDE test runners, providing out-of-the-box integration
- Consistent naming reduces cognitive load for developers moving between packages and improves codebase navigability

## Consequences

Positive:
- Automated test discovery works reliably across all packages without manual configuration updates
- CI/CD pipelines can use simple, consistent glob patterns to locate and execute tests
- IDE test runners automatically detect and provide run/debug capabilities for test files
- New developers can immediately identify test files by extension without consulting documentation
- Refactoring and moving test files becomes safer as tooling automatically tracks them

Negative:
- Existing test files with non-standard naming require migration effort
- Developers familiar with .spec.ts convention from other projects may need to adjust
- Test configuration files need updates to enforce the .test.ts pattern
- Mixed naming conventions during transition period may cause confusion
- Some testing frameworks may require explicit configuration to recognize .test.ts over their defaults

## Alternatives

- Use .spec.ts extension following Angular and NestJS conventions (rejected)
  Rejected because: Pattern detection shows no usage of .spec.ts in the codebase; adopting it would conflict with established practice and require migration of existing tests
  When valid: Valid for new projects with Angular/NestJS framework requirements
- Allow mixed naming conventions (.test.ts and .spec.ts) based on developer preference (rejected)
  Rejected because: Mixed conventions complicate test discovery configuration and reduce consistency; CI/CD would need to maintain multiple glob patterns
  When valid: Never recommended; consistency is critical for automated tooling
- Use __tests__ directory with no specific file extension requirement (rejected)
  Rejected because: Directory-based organization alone doesn't provide file-level identification; .test.ts extension enables both directory and co-located test patterns
  When valid: Can be combined with .test.ts extension for additional organization

## Risks

- Existing tests with non-standard naming may be excluded from CI/CD execution during transition
  Mitigation: Audit all test files before enforcement; create migration script to rename files; run parallel test discovery patterns during transition period
  Owner: Engineering team / CI/CD maintainers
- Third-party packages or tools may have hardcoded expectations for different test file naming
  Mitigation: Document known exceptions; configure test runners explicitly when needed; maintain exception list in root documentation
  Owner: Package maintainers
- Developers may accidentally create test files with wrong extensions, causing silent test exclusion
  Mitigation: Add linting rules to detect test-like files without .test.ts extension; implement pre-commit hooks; add CI check to verify test file naming
  Owner: DevOps team

## Implementation Notes

- Update test runner configurations (jest.config.js, vitest.config.ts) to use testMatch or include patterns like '**/*.test.ts'
- Add ESLint rules or custom scripts to detect files containing test code (describe, it, test blocks) without .test.ts extension
- Create migration script to rename existing test files: find . -name '*.spec.ts' -exec rename 's/\.spec\.ts$/.test.ts/' {} \;
- Update package.json test scripts to use glob patterns that match .test.ts files
- Document the convention in CONTRIBUTING.md and test documentation with examples from each package type

## Continuation Context


Verify commands:
- find . -type f -name '*.test.ts' | wc -l
- find . -type f \( -name '*.spec.ts' -o -name '*_test.ts' \) -not -path '*/node_modules/*' | wc -l
- grep -r 'testMatch.*\.test\.ts' --include='*.config.{js,ts}' .

Accept when:
- All TypeScript test files in the repository use .test.ts extension (verify command 1 returns count > 0, command 2 returns 0)
- Test runner configurations explicitly include **/*.test.ts pattern (verify command 3 finds matches)
- No test files with alternative extensions (.spec.ts, _test.ts) exist outside documented exceptions
- CI/CD pipeline successfully discovers and executes all tests using .test.ts glob pattern

## Enforcement

- Verified by: CI/CD pipeline checks for non-compliant test file names using find commands
- Verified by: Pre-commit hooks validate test file naming conventions
- Verified by: Code review checklist includes verification of .test.ts extension for new test files
- Verified by: Automated linting rules detect test code in files without .test.ts extension
- Violation handling: CI build fails if test files with non-standard extensions are detected outside exception list
- Violation handling: Pre-commit hook blocks commits containing test files with incorrect naming
- Violation handling: Code review requires changes before merge if test files don't follow convention
- Violation handling: Weekly audit report identifies and flags non-compliant test files for remediation
- Exception process: Submit exception request to architecture review board with justification (e.g., framework requirement)
- Exception process: Document approved exception in root EXCEPTIONS.md file with rationale and scope
- Exception process: Update CI/CD exclusion patterns to allow specific exception cases
- Exception process: Review exceptions quarterly to determine if they can be eliminated