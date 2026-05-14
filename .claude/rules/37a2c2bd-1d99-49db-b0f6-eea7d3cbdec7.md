<rule_activation id="37a2c2bd-1d99-49db-b0f6-eea7d3cbdec7" title="Standardize Test Organization in Dedicated Test Directories for CI/CD Integration: Test Files Placed" applies_to="**/*.test.ts">
These rules are ALWAYS ACTIVE for all TypeScript test files across all packages and applications in the monorepo.
</rule_activation>

### Rules

- **R-TEST-001** MUST: Test files MUST be placed in dedicated test directories (tests/, tests/unit/, tests/integration/) rather than co-located with source files

### Verify

```bash
# Verify no test files exist outside tests/ directories
find . -name '*.test.ts' -not -path '*/tests/*' -not -path '*/node_modules/*' | wc -l | grep -q '^0$'

# Count test files in correct locations
find packages/*/tests apps/*/tests -name '*.test.ts' 2>/dev/null | wc -l

# Verify test runner configurations reference tests/ directory pattern
grep -r 'testMatch\|testRegex' package.json tsconfig.json jest.config.* vitest.config.* 2>/dev/null
```

**Accept when:**
- All .test.ts files are located within tests/ directories (no test files found outside tests/ directories)
- Each package and application has its own tests/ directory at the package root level
- Test runner configurations reference the tests/ directory pattern for test discovery

<enforcement>
Claude Code MUST NOT skip or defer verification. All test files must conform to the dedicated test directory structure before proceeding with code generation or modification.
</enforcement>