<rule_activation id="f1cd5bdd-16de-44bc-8f90-d3ee9a2f89cc" title="Standardize Browser and Platform API Abstraction Layers for External Integration: Application Logic Not" applies_to="**/*">
These rules are ALWAYS ACTIVE for all files matching `**/*`. Application logic MUST NOT directly reference global platform objects outside of designated abstraction modules to ensure maintainability, testability, and portability across different runtime environments.
</rule_activation>

### Rules

- **R-API-001** MUST_NOT: Application logic MUST NOT directly reference global platform objects (window.chrome, process, etc.) outside of designated abstraction modules.

### Scope

**In scope:**
- All Chrome Extension API usage (chrome.*, browser.*)
- Browser DOM APIs accessed from application code (window, document, navigator)
- Node.js platform APIs used in CLI and build tools (process, fs, path)
- Native system APIs exposed through FFI or bindings (RDP, system calls)
- Third-party platform SDKs that provide external integration points

**Out of scope:**
- Standard JavaScript/TypeScript language features (Promise, Array, Object)
- Internal application APIs and service interfaces
- NPM package imports that provide cross-platform abstractions
- Test framework APIs (Jest, Playwright) used exclusively in test code
- Build tool configurations and scripts

**Exceptions:**
- EXC-001: Platform detection or feature detection code that must check for API existence
- EXC-002: Type declarations or ambient type definitions for platform APIs

### Verify

```bash
# Check for direct window.chrome or global.chrome references outside abstraction modules
grep -r 'window\.chrome\|global\.chrome' --include='*.ts' --include='*.tsx' --exclude-dir=node_modules --exclude='**/utils/chrome.ts' . | grep -v 'typeof.*chrome' | wc -l | awk '{if ($1 > 0) exit 1}'

# Check for direct process references outside abstraction modules
grep -r 'process\.' --include='*.ts' --exclude-dir=node_modules --exclude='**/utils/*.ts' --exclude='**/*.config.*' . | wc -l | awk '{if ($1 > 10) exit 1}'

# Verify at least one platform API abstraction module exists
find . -name 'chrome.ts' -o -name 'platform.ts' -o -name '*-api.ts' | grep utils | wc -l | awk '{if ($1 < 1) exit 1}'
```

**Accept when:**
- No direct references to platform globals (window.chrome, process, etc.) exist outside of designated abstraction utility modules
- At least one platform API abstraction module exists in packages that integrate with external APIs
- All platform API abstractions provide TypeScript type definitions and handle availability checks

### Implementation Guidance

- Create dedicated utility modules (e.g., utils/chrome.ts, utils/platform.ts) in each package that needs platform API access, following consistent naming conventions
- Use TypeScript interfaces to define abstraction contracts, enabling easy mocking in tests and clear documentation of expected behaviors
- Implement feature detection at module initialization time and cache results to avoid repeated checks; provide clear error messages when required APIs are unavailable
- For Chrome Extension APIs, consider using webextension-polyfill or similar libraries to normalize differences between Chrome and Firefox implementations
- Document the mapping between abstraction methods and underlying platform APIs to help developers understand the relationship and debug issues

<enforcement>
Claude Code MUST verify compliance with R-API-001 before accepting code changes. Violations of direct platform API usage outside abstraction modules MUST be flagged. CI pipeline verification commands MUST pass. Pull requests MUST NOT be approved until platform API access is properly abstracted.
</enforcement>