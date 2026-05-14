# Enforce Environment Variable Validation with Zod Schema for Runtime Configuration: Configuration Validation Occur

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all runtime configuration and environment variable handling in the codebase. All modules that consume environment variables or external configuration MUST apply the validation rules defined herein.

## Context

- The codebase processes environment variables and runtime configuration from external sources that may contain invalid, malformed, or malicious data
- Without validation, invalid configuration can cause runtime errors, security vulnerabilities, or undefined behavior that is difficult to debug
- The pattern was detected across 3 files with 91.53% confidence, indicating a consistent architectural approach to input validation using Zod schemas
- The security.input_validation facet indicates this pattern addresses security concerns related to untrusted external input
- Early validation at configuration load time provides fail-fast behavior and clear error messages rather than cryptic runtime failures

## Problem Statement

Environment variables and external configuration sources are inherently untyped and untrusted. Without systematic validation, applications may operate with invalid configuration leading to runtime errors, security vulnerabilities, type coercion issues, and difficult-to-diagnose failures. A standardized approach to validating and parsing environment variables is needed to ensure type safety and fail-fast behavior.

## Decision

1. MUST: Configuration validation MUST occur at application startup or module initialization, not lazily during runtime

## Policy Block

- MUST Configuration validation MUST occur at application startup or module initialization, not lazily during runtime

In scope:
- Environment variables consumed from process.env
- Configuration files loaded from external sources (JSON, YAML, TOML)
- Runtime parameters passed via command-line arguments
- Configuration injected via container orchestration or cloud platforms
- Any external input that controls application behavior or runtime settings

Out of scope:
- Hardcoded constants defined in source code
- Type-safe configuration objects already validated upstream
- Internal state management that doesn't originate from external sources
- Test fixtures and mock data in test suites

Exceptions:
- EXC-001: Development and debugging scenarios where direct environment variable access is needed for troubleshooting
- EXC-002: Legacy code modules scheduled for deprecation within the current quarter

## Rationale

- Zod provides runtime type validation with TypeScript type inference, ensuring configuration is both runtime-safe and compile-time type-safe
- The pattern was detected with 91.53% confidence across 3 files, indicating this is an established architectural practice in the codebase
- Early validation at startup provides fail-fast behavior, preventing applications from running with invalid configuration and causing issues later
- The security.input_validation facet indicates this pattern addresses critical security concerns by treating external configuration as untrusted input that must be validated

## Consequences

Positive:
- Type safety: Configuration objects have accurate TypeScript types inferred from Zod schemas, preventing type-related bugs
- Fail-fast behavior: Invalid configuration is detected immediately at startup with clear error messages, not during runtime
- Security: Untrusted external input is validated before use, preventing injection attacks and unexpected behavior
- Developer experience: Clear schema definitions serve as documentation and provide autocomplete in IDEs

Negative:
- Additional dependency: Requires Zod library as a runtime dependency, increasing bundle size
- Boilerplate: Each configuration module requires schema definition code, adding initial development overhead
- Learning curve: Developers must learn Zod schema syntax and validation patterns
- Startup time: Validation adds minimal overhead to application startup time

## Alternatives

- Use TypeScript interfaces without runtime validation (rejected)
  Rejected because: TypeScript types are erased at runtime and provide no protection against invalid environment variables or configuration at runtime
  When valid: Only valid for internal type-safe configuration that never originates from external sources
- Manual validation with if-statements and type guards (rejected)
  Rejected because: Manual validation is error-prone, verbose, difficult to maintain, and doesn't provide automatic TypeScript type inference
  When valid: May be acceptable for very simple single-value validations in isolated utility functions
- Use alternative validation libraries (Yup, Joi, io-ts) (rejected)
  Rejected because: Zod provides superior TypeScript integration with type inference and is already established in the codebase with 91.53% confidence
  When valid: Could be reconsidered if Zod proves inadequate for specific validation requirements

## Risks

- Incomplete migration: Existing code may still access process.env directly, bypassing validation
  Mitigation: Implement linting rules to detect direct process.env access and require validation. Conduct codebase audit to identify and migrate legacy patterns.
  Owner: Engineering team
- Schema drift: Zod schemas may become out of sync with actual environment variable usage over time
  Mitigation: Include schema validation in CI/CD pipeline. Require schema updates as part of code review for any configuration changes.
  Owner: Engineering team
- Overly strict validation may prevent legitimate use cases or emergency configuration changes
  Mitigation: Design schemas with appropriate flexibility. Document exception process for urgent production issues. Use SHOULD rules for recommendations rather than MUST where appropriate.
  Owner: Engineering team and SRE

## Implementation Notes

- Create a centralized configuration module that exports validated config objects and their schemas for reuse across the application
- Use z.coerce for type conversion (e.g., z.coerce.number() for numeric environment variables stored as strings)
- Provide sensible defaults using .default() for optional configuration to reduce deployment complexity
- Structure error messages to clearly indicate which environment variable failed validation and what the expected format is
- Consider using .strict() on object schemas to catch typos in configuration keys
- Export both the schema type and the validated config object: export const configSchema = z.object({...}); export const config = configSchema.parse(process.env);

## Continuation Context


Verify commands:
- grep -r 'process\.env\.' --include='*.ts' --include='*.js' --exclude-dir=node_modules | grep -v 'z\.object\|zod\|schema' | wc -l
- grep -r 'import.*zod' --include='*.ts' packages/ | wc -l
- npm test -- --grep 'configuration.*validation'

Accept when:
- All configuration modules use Zod schemas for validation with no direct process.env access in application logic
- CI pipeline includes tests that verify configuration validation behavior with invalid inputs
- Linting rules detect and flag direct environment variable access without validation

## Enforcement

- Verified by: ESLint custom rule to detect direct process.env access outside of configuration modules
- Verified by: Code review checklist requiring Zod schema validation for any new configuration parameters
- Verified by: CI/CD pipeline tests that validate configuration schemas with both valid and invalid inputs
- Verified by: Static analysis tools scanning for unvalidated external input usage
- Violation handling: CI build fails if linting rules detect direct process.env access in application code
- Violation handling: Code review blocks merge if new configuration parameters lack Zod validation
- Violation handling: Security scanning tools flag unvalidated external input as high-priority issues
- Violation handling: Quarterly architecture reviews audit configuration validation coverage
- Exception process: Submit exception request to tech lead with justification and risk assessment
- Exception process: Document exception in code comments with ticket reference and expiration date
- Exception process: Architecture review board evaluates exceptions for legacy code or special circumstances
- Exception process: All exceptions must include a remediation plan with timeline