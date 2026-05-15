# Adopt Playwright for E2E Testing of Frontend Rendering Models: Frontend Rendering Model

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The project requires comprehensive end-to-end testing of frontend rendering behavior across multiple packages (cli, web-integration)
- Testing scenarios include storage navigation (goto), cache functionality, input prompt rendering, and screenshot capabilities with CDP fallback mechanisms
- The testing.e2e facet indicates a standardized approach to validating rendering models in browser-like environments
- Multiple test files demonstrate consistent patterns for verifying UI/UX rendering behavior across different components and integration points

## Problem Statement

Frontend rendering models require reliable, automated verification to ensure consistent user experience across different browsers, rendering contexts, and edge cases. Without a standardized E2E testing framework, rendering regressions can slip through unit tests, and browser-specific issues may only surface in production.

## Decision

1. MUST: All frontend rendering model implementations MUST include Playwright-based E2E tests that verify rendering behavior in real browser contexts

## Policy Block

- MUST All frontend rendering model implementations MUST include Playwright-based E2E tests that verify rendering behavior in real browser contexts

In scope:
- All web-based frontend components requiring browser rendering
- CLI tools with web UI components or browser automation features
- Integration tests validating cross-package rendering behavior
- Screenshot and visual regression testing scenarios

Out of scope:
- Pure backend API testing without UI components
- Unit tests for non-rendering business logic
- Mobile native application testing (unless using Playwright mobile capabilities)
- Performance benchmarking (unless combined with rendering validation)

## Rationale

- Pattern detected across 4 files with 92.50% confidence indicates a deliberate architectural choice for E2E testing strategy
- Playwright provides cross-browser testing capabilities essential for validating rendering models across different browser engines
- The presence of specialized tests (CDP fallback, narrow input handling) demonstrates mature testing practices addressing real-world edge cases
- Consistent test organization across packages (cli, web-integration) suggests this is a project-wide standard rather than isolated implementation

## Consequences

Positive:
- Increased confidence in rendering behavior across different browsers and contexts through automated E2E validation
- Early detection of rendering regressions before production deployment
- Standardized testing approach across packages improves maintainability and developer onboarding
- Fallback mechanisms in tests ensure robustness even when browser APIs have limitations

Negative:
- E2E tests are slower than unit tests, potentially increasing CI/CD pipeline duration
- Playwright requires browser binaries to be installed, increasing infrastructure complexity
- Flaky tests may occur due to timing issues or browser-specific quirks requiring additional maintenance
- Learning curve for developers unfamiliar with Playwright API and E2E testing patterns

## Alternatives

- Use Cypress for E2E testing instead of Playwright (rejected)
  Rejected because: Playwright offers better cross-browser support (Chromium, Firefox, WebKit) and more robust CDP integration for advanced scenarios like screenshot fallbacks
  When valid: For projects exclusively targeting Chromium-based browsers where Cypress's developer experience advantages outweigh multi-browser support
- Rely solely on unit tests with JSDOM for rendering validation (rejected)
  Rejected because: JSDOM cannot accurately simulate real browser rendering behavior, CSS layout, or browser-specific APIs needed for comprehensive rendering model validation
  When valid: For simple component logic testing where full browser rendering is not critical
- Use Selenium WebDriver for E2E testing (rejected)
  Rejected because: Selenium has slower execution, more complex setup, and lacks modern features like auto-waiting and CDP protocol access that Playwright provides
  When valid: For legacy projects with existing Selenium infrastructure where migration costs outweigh benefits

## Risks

- E2E test flakiness due to timing issues or network dependencies could reduce developer trust in test suite
  Mitigation: Implement Playwright's auto-waiting mechanisms, use explicit wait strategies, and isolate tests from external network dependencies using mocks or fixtures
  Owner: QA and Frontend Engineering Team
- Browser binary updates may introduce breaking changes or new rendering behaviors
  Mitigation: Pin Playwright and browser versions in CI, establish regular update cycles with regression testing, and monitor Playwright release notes
  Owner: DevOps and Frontend Engineering Team
- Increased CI execution time may slow down development velocity
  Mitigation: Parallelize test execution, implement test sharding, and consider running full E2E suite only on main branch while running smoke tests on PRs
  Owner: DevOps Team

## Implementation Notes

- Organize E2E tests in dedicated directories (e.g., tests/ai/web/playwright/) to separate them from unit tests
- Create reusable Playwright fixtures for common rendering scenarios (navigation, storage, screenshots) to reduce test duplication
- Implement CDP fallback patterns for browser features that may not be available in all contexts (as demonstrated in screenshot-cdp-fallback.spec.ts)
- Use descriptive test file names that clearly indicate the rendering scenario being validated (e.g., cache-functionality.spec.ts, narrow-input-prompt.spec.ts)

## Continuation Context


Verify commands:
- find . -name '*.spec.ts' -path '*/playwright/*' | wc -l | grep -v '^0$'
- grep -r "@playwright/test" package.json packages/*/package.json
- grep -r "test.*playwright" package.json packages/*/package.json | grep -E '(test|e2e)'

Accept when:
- At least one Playwright test file exists in the project with .spec.ts extension
- Package.json includes @playwright/test as a dependency
- Test scripts reference Playwright for E2E testing execution

## Enforcement

- Verified by: CI pipeline checks for presence of Playwright E2E tests in packages with frontend components
- Verified by: Code review process validates that new rendering features include corresponding E2E tests
- Verified by: Automated linting rules ensure test files follow naming conventions (*.spec.ts in playwright directories)
- Violation handling: PR builds fail if new frontend components lack E2E test coverage
- Violation handling: Code review checklist includes verification of E2E test presence for rendering changes
- Violation handling: Quarterly audits identify packages with insufficient E2E coverage for remediation
- Exception process: Exceptions require architectural review approval with documented justification
- Exception process: Temporary exceptions must include a remediation plan with timeline
- Exception process: Exception requests must demonstrate why alternative testing approaches provide equivalent coverage