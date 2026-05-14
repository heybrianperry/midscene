<rule_activation id="3d7efc82-3edf-43ec-a8fb-4f41f2cbb6ea" title="Adopt Environment-Driven Configuration Management for Chrome Extension Packaging and Device Control: Chrome Extension Packaging" applies_to="**/*">
These rules are ALWAYS ACTIVE for all runtime environment configuration, Chrome extension packaging workflows, and device control systems that require environment-specific behavior adaptation.
</rule_activation>

### Rules

- **R-ENV-001** MUST: Chrome extension packaging scripts MUST support environment-specific manifest modifications, asset bundling, and output paths based on detected or specified environment

### Verify

```bash
# Check for environment variable usage in critical infrastructure files
grep -r 'process.env' apps/chrome-extension/scripts/ packages/computer/src/ packages/computer/tests/ | grep -v node_modules

# Verify environment template exists
test -f .env.example && echo 'Environment template exists' || echo 'Missing .env.example'

# Scan for hardcoded secrets or credentials
grep -r 'API_KEY\|SECRET\|PASSWORD' --include='*.js' --include='*.ts' . | grep -v 'process.env' | grep -v node_modules || echo 'No hardcoded secrets found'
```

**Accept when:**
- All environment-specific values are loaded from process.env or configuration files, not hardcoded in source
- A .env.example file exists documenting all required environment variables
- No secrets or credentials are found hardcoded in source files (grep verification passes)
- Configuration validation runs at startup and provides clear error messages for missing/invalid values

<enforcement>
Claude Code MUST NOT skip or defer verification. Configuration validation and environment variable checks are mandatory before accepting any changes to Chrome extension packaging, device control, or test automation systems.
</enforcement>