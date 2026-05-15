<rule_activation id="2ba59aeb-9bdc-4d3a-9aae-889e733a3fef" title="Adopt Playwright for Browser-Based Testing and Rendering Verification: Tests Use Chrome" applies_to="**/*">
These rules are ALWAYS ACTIVE for all browser-based testing, UI rendering verification, and web integration test scenarios across the codebase.
</rule_activation>

### Rules

- **R-PLAYWRIGHT-001** MAY: Tests MAY use Chrome DevTools Protocol (CDP) directly for advanced browser automation scenarios not covered by standard Playwright APIs.

### Scope

**In scope:**
- All web-based UI component testing
- Browser rendering verification tests
- Web integration test suites
- Screenshot and visual regression testing
- Storage state and navigation testing
- Cache functionality verification
- Cross-browser compatibility tests

**Out of scope:**
- Unit tests for non-UI business logic
- Backend API integration tests without browser rendering
- Mobile native application testing
- Server-side rendering tests that don't require browser automation
- Performance benchmarking tests (unless specifically testing rendering performance)

### Verify

```bash
# Verify Playwright imports in test files
grep -r "@playwright/test" packages/*/tests --include="*.ts" --include="*.js"

# Find Playwright test files by naming convention
find packages -name "*.spec.ts" -path "*/playwright/*" -type f

# Verify Playwright API usage patterns
grep -r "page\.goto\|page\.screenshot\|browserContext" packages/*/tests --include="*.spec.ts" | head -20
```

**Accept when:**
- All web integration test files import from '@playwright/test' and use Playwright APIs
- Test files follow naming convention (*.spec.ts) and are organized in appropriate test directories
- Screenshot tests implement CDP fallback mechanisms and storage/navigation tests verify state persistence
- CI/CD pipeline successfully executes Playwright tests across multiple browser contexts

<enforcement>
Claude Code MUST verify Playwright adoption through automated CI/CD pipeline checks, code review processes, and static analysis tools. Violations are flagged during code review and escalated to the Architecture Review Board for repeated non-compliance.
</enforcement>