# Adopt Environment-Driven Configuration Management for Chrome Extension Packaging and Device Control: Test Helpers Automation

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all runtime environment configuration, Chrome extension packaging workflows, and device control systems that require environment-specific behavior adaptation.

## Context

- The codebase includes Chrome extension packaging scripts, device control systems, and test helpers that must operate across multiple runtime environments (development, testing, production)
- Different execution contexts require distinct configuration parameters, file paths, API endpoints, and behavioral flags that cannot be hardcoded
- Event-driven boundaries in the system (indicated by facet: boundaries.event_driven) require dynamic configuration loading to respond to runtime conditions
- Three distinct files demonstrate a consistent pattern of environment-aware configuration management with 91.07% confidence across the detection pipeline
- The pattern appears in critical infrastructure components: extension packaging (pack-extension.js), device abstraction layer (device.ts), and test automation (chrome-extension-helpers.ts)

## Problem Statement

Systems that span multiple runtime environments and event-driven boundaries need a consistent, reliable mechanism to load and apply environment-specific configuration without hardcoding values or creating brittle conditional logic. Without standardized environment management, configuration drift, deployment failures, and environment-specific bugs become common, particularly in Chrome extension packaging workflows and device control systems that must adapt to varying execution contexts.

## Decision

1. SHOULD: Test helpers and automation utilities SHOULD provide environment-specific mocking, stubbing, or real integration based on test environment configuration

## Policy Block

- SHOULD Test helpers and automation utilities SHOULD provide environment-specific mocking, stubbing, or real integration based on test environment configuration

In scope:
- Chrome extension packaging and build scripts
- Device control and hardware abstraction layers
- Test automation helpers and fixtures
- Runtime environment detection and initialization code
- Configuration loading and validation modules
- Event-driven system boundaries requiring dynamic configuration

Out of scope:
- Static type definitions and interfaces
- Pure algorithmic functions with no environmental dependencies
- Third-party library configuration managed by external tools
- Compile-time constants that are truly invariant across all environments

Exceptions:
- EXC-001: Local development requires hardcoded localhost URLs for rapid iteration without environment setup
- EXC-002: Emergency hotfix requires temporary environment-specific code path to resolve production incident

## Rationale

- Pattern detected with 91.07% confidence across 3 critical infrastructure files (pack-extension.js, device.ts, chrome-extension-helpers.ts), indicating a deliberate architectural choice
- Event-driven boundaries (facet: boundaries.event_driven) require runtime configuration flexibility to respond to dynamic execution contexts and external events
- Chrome extension packaging workflows inherently require environment-specific builds for development (unminified, with debugging) vs. production (minified, optimized)
- Device control systems must adapt to varying hardware availability, connection parameters, and feature support across development, CI, and production environments

## Consequences

Positive:
- Single codebase can deploy to multiple environments without modification, reducing deployment complexity and risk
- Environment-specific bugs are caught earlier through consistent configuration management patterns
- Secrets and credentials are properly externalized, improving security posture
- Test automation can seamlessly switch between mocked and real integrations based on environment
- Chrome extension builds are optimized per environment (debug vs. production) automatically

Negative:
- Additional complexity in configuration loading and validation logic
- Developers must understand environment variable conventions and configuration cascading rules
- Debugging can be harder when configuration issues arise from environment mismatches
- Initial setup overhead for new environments or deployment targets

## Alternatives

- Hardcode all configuration values and maintain separate branches for each environment (rejected)
  Rejected because: Creates massive maintenance burden, merge conflicts, and high risk of deploying wrong configuration to wrong environment
  When valid: Never valid for multi-environment systems
- Use build-time configuration injection with separate build artifacts per environment (rejected)
  Rejected because: While viable, creates artifact proliferation and doesn't support runtime environment detection needed for event-driven boundaries
  When valid: Acceptable for purely static deployments with no runtime environment variation
- Implement centralized configuration service with remote fetching (deferred)
  Rejected because: Adds network dependency and complexity; may be considered for future microservices architecture
  When valid: When system scales to distributed services requiring centralized configuration management

## Risks

- Missing or misconfigured environment variables cause runtime failures in production
  Mitigation: Implement configuration validation at startup with fail-fast behavior and comprehensive error messages; use schema validation libraries
  Owner: Platform Engineering Team
- Configuration drift between environments leads to environment-specific bugs that don't reproduce locally
  Mitigation: Maintain configuration templates and documentation; implement automated configuration auditing in CI/CD pipeline
  Owner: DevOps Team
- Developers accidentally commit sensitive configuration to source control
  Mitigation: Use .gitignore patterns, pre-commit hooks, and secret scanning tools; provide .env.example templates
  Owner: Security Team

## Implementation Notes

- Use dotenv or similar libraries for local development environment variable management; provide .env.example templates in repository
- Implement configuration validation schemas (e.g., Zod, Joi) that run at application startup and fail fast with clear error messages
- For Chrome extension packaging, use environment-specific manifest.json templates and merge logic in pack-extension.js
- Document all required environment variables in README.md with descriptions, valid values, and examples
- Consider implementing configuration layers: defaults.json → environment.json → process.env → runtime overrides

## Continuation Context


Verify commands:
- grep -r 'process.env' apps/chrome-extension/scripts/ packages/computer/src/ packages/computer/tests/ | grep -v node_modules
- test -f .env.example && echo 'Environment template exists' || echo 'Missing .env.example'
- grep -r 'API_KEY\|SECRET\|PASSWORD' --include='*.js' --include='*.ts' . | grep -v 'process.env' | grep -v node_modules || echo 'No hardcoded secrets found'

Accept when:
- All environment-specific values are loaded from process.env or configuration files, not hardcoded in source
- A .env.example file exists documenting all required environment variables
- No secrets or credentials are found hardcoded in source files (grep verification passes)
- Configuration validation runs at startup and provides clear error messages for missing/invalid values

## Enforcement

- Verified by: Automated CI checks scanning for hardcoded secrets and configuration values
- Verified by: Code review checklist requiring verification of environment variable usage
- Verified by: Pre-commit hooks preventing commits of .env files or hardcoded credentials
- Verified by: Runtime configuration validation tests in test suite
- Violation handling: CI pipeline fails if hardcoded secrets are detected by scanning tools
- Violation handling: Pull requests blocked until configuration is properly externalized
- Violation handling: Runtime failures with clear error messages when required environment variables are missing
- Violation handling: Security team notification for any committed secrets (with immediate rotation required)
- Exception process: Submit exception request to team lead with justification and timeline for proper implementation
- Exception process: Document exception in code with TODO comment and tracking ticket reference
- Exception process: Exceptions reviewed in sprint retrospectives and prioritized for resolution
- Exception process: Emergency hotfix exceptions require follow-up ticket created within 24 hours