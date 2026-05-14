<rule_activation id="bb658460-dcbf-4fd9-a959-d0c77cebaf4b" title="Standardize console.log for Development Logging and Debugging: Remove Conditionally Disable" applies_to="**/*">
These rules are ALWAYS ACTIVE for all files in the codebase, particularly Chrome extension components, CLI tools, visualizer UI components, recorder timeline components, and build scripts.
</rule_activation>

### Rules

- **R-CONSOLE-001** SHOULD: Remove or conditionally disable verbose console.log statements before production releases unless they provide operational value.
- **R-CONSOLE-002** SHOULD: Establish naming conventions for log message prefixes (e.g., '[StoreName]', '[Worker]', '[Bridge]') to enable easy filtering in console output.
- **R-CONSOLE-003** SHOULD: Configure build tools (webpack, rollup, etc.) to strip console.log statements in production builds using plugins like terser or babel-plugin-transform-remove-console.
- **R-CONSOLE-004** SHOULD: Use environment variables or feature flags to conditionally enable verbose logging in development (e.g., `if (process.env.DEBUG) console.log(...)`).
- **R-CONSOLE-005** MUST: Never log sensitive information (passwords, tokens, secrets, API keys) to console.
- **R-CONSOLE-006** MUST: Document inline code comments for any approved exceptions to production console logging, including rationale and expected output.

### Verify

```bash
# Count console.log statements across target files
grep -r 'console\.log' apps/chrome-extension/src packages/cli/src packages/visualizer/src packages/recorder/src --include='*.ts' --include='*.tsx' | wc -l

# Audit for sensitive data patterns in console.log
grep -r 'console\.log.*password\|console\.log.*token\|console\.log.*secret' . --include='*.ts' --include='*.tsx' || echo 'No sensitive data logging detected'

# Verify production build strips or minimizes console output
npm run build 2>&1 | grep -i 'console' || echo 'Build completed without console warnings'
```

**Accept when:**
- Console.log statements are present in detected files (chrome-extension, CLI, visualizer, recorder, build scripts) for development debugging purposes
- No console.log statements contain obvious sensitive information patterns (passwords, tokens, secrets, API keys)
- Production builds either strip console statements or include only intentional operational logging with documented justification
- Code review process verifies no sensitive data is logged to console
- Build configuration is verified to strip or minimize console output in production

<enforcement>
Claude Code MUST verify all rules in this activation block. Violations of R-CONSOLE-005 (sensitive data logging) require immediate remediation. Exceptions to R-CONSOLE-001 require explicit justification in pull request comments and engineering lead approval.
</enforcement>