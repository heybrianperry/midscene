# Standardize Environment Variable Configuration Management Pattern: Direct Process Env

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all configuration and environment management code across the codebase.

## Context

- The codebase exhibits a consistent pattern across multiple packages (playground, cli, visualizer) for managing runtime configuration through environment variables
- Configuration management requires a standardized approach to ensure consistent behavior across different execution contexts (development, testing, production)
- Pattern detected with 91.13% confidence across 3 files suggests an established architectural convention that should be formalized
- Environment-specific configuration needs to be accessible at runtime while maintaining type safety and validation capabilities

## Problem Statement

Without a standardized approach to environment variable configuration management, different packages may implement inconsistent patterns for accessing and validating runtime configuration, leading to maintenance overhead, potential runtime errors, and difficulty in understanding configuration dependencies across the system.

## Decision

1. MUST_NOT: Direct process.env access MUST NOT be scattered throughout application code; all access must go through configuration modules

## Policy Block

- MUST_NOT Direct process.env access MUST NOT be scattered throughout application code; all access must go through configuration modules

In scope:
- All packages within the monorepo (playground, cli, visualizer, etc.)
- Runtime configuration accessed via environment variables
- Application initialization and bootstrap code
- Test setup and teardown procedures requiring environment configuration

Out of scope:
- Build-time configuration (webpack, vite, etc.) that does not affect runtime behavior
- Third-party library configuration that requires direct environment variable access
- Temporary debugging or diagnostic code marked for removal

Exceptions:
- EXC-001: Third-party libraries require direct process.env access and do not support configuration injection

## Rationale

- Pattern detected across 3 files with 91.13% confidence indicates this is an established architectural convention worth formalizing
- Centralized configuration management improves maintainability by providing a single source of truth for environment-dependent behavior
- Type-safe configuration access reduces runtime errors and improves developer experience through IDE autocomplete and compile-time validation
- Consistent pattern across packages (playground, cli, visualizer) demonstrates cross-cutting architectural concern that benefits from standardization

## Consequences

Positive:
- Improved code maintainability through centralized configuration management
- Enhanced type safety and reduced runtime errors from invalid environment variable access
- Better testability through ability to mock or override configuration in test environments
- Clearer documentation of configuration dependencies and requirements for each package

Negative:
- Additional boilerplate required to create and maintain configuration modules for each package
- Potential performance overhead from validation logic at initialization time
- Learning curve for developers unfamiliar with the centralized configuration pattern
- May require refactoring existing code that directly accesses process.env

## Alternatives

- Allow direct process.env access throughout the codebase without centralization (rejected)
  Rejected because: Leads to scattered configuration logic, makes it difficult to track dependencies, and increases risk of runtime errors from missing or invalid environment variables
  When valid: Only acceptable for throwaway prototypes or single-file scripts
- Use a third-party configuration management library (e.g., dotenv-safe, convict, config) (deferred)
  Rejected because: Not rejected, but deferred pending evaluation of library overhead vs. custom implementation benefits
  When valid: Should be reconsidered if configuration complexity grows significantly or if validation requirements become more sophisticated
- Implement a monorepo-wide shared configuration package that all other packages import (rejected)
  Rejected because: Creates tight coupling between packages and reduces flexibility for package-specific configuration needs
  When valid: Only valid for truly global configuration that applies to all packages without variation

## Risks

- Developers may bypass centralized configuration and directly access process.env, undermining the pattern
  Mitigation: Implement linting rules to detect direct process.env access and enforce code review checks
  Owner: Engineering team
- Configuration modules may become bloated with unrelated logic beyond environment variable management
  Mitigation: Establish clear boundaries for configuration module responsibilities and separate concerns (validation, transformation, access)
  Owner: Architecture team
- Initialization-time validation may cause application startup failures in production if not properly tested
  Mitigation: Implement comprehensive test coverage for configuration validation and include configuration checks in CI/CD pipeline
  Owner: DevOps and engineering team

## Implementation Notes

- Create a configuration module (common.ts, config.ts, or env.ts) in each package's src directory
- Export typed constants or objects representing environment variables rather than functions when possible for better tree-shaking
- Use TypeScript's strict mode to ensure all configuration exports are properly typed
- Document required vs. optional environment variables in package README files
- Consider using Zod or similar validation libraries for complex configuration schemas

## Continuation Context


Verify commands:
- grep -r 'process\.env' --include='*.ts' --include='*.js' --exclude-dir=node_modules | grep -v 'common.ts\|config.ts\|env.ts' | wc -l
- find . -name 'common.ts' -o -name 'config.ts' -o -name 'env.ts' | xargs grep -l 'process\.env' | wc -l
- npm run lint -- --rule 'no-process-env: error' 2>&1 | grep -c 'process.env'

Accept when:
- All direct process.env accesses are contained within designated configuration modules (common.ts, config.ts, env.ts)
- Each package with environment dependencies has a dedicated configuration module with typed exports
- Linting rules successfully detect and flag any direct process.env access outside configuration modules

## Enforcement

- Verified by: ESLint rules configured to detect direct process.env access outside configuration modules
- Verified by: Code review checklist includes verification of configuration management patterns
- Verified by: Automated CI checks run grep commands to detect pattern violations
- Violation handling: CI pipeline fails if direct process.env access is detected outside configuration modules
- Violation handling: Pull requests with violations are blocked until refactored to use centralized configuration
- Violation handling: Existing violations are tracked as technical debt items and prioritized for refactoring
- Exception process: Developer documents the exception reason in code comments and PR description
- Exception process: Tech lead reviews and approves the exception based on technical merit
- Exception process: Exception is logged in the configuration module with a TODO comment for future resolution
- Exception process: Exceptions are reviewed quarterly to determine if they can be eliminated