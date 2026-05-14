# Standardize Environment Variable Configuration for Runtime Secrets Management: Sensitive Configuration Data

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all runtime components that require configuration or secrets management.

## Context

- The system requires secure management of sensitive configuration data including API keys, authentication tokens, and service credentials across multiple runtime environments (CLI, browser extensions, Puppeteer agents, and device controllers)
- Environment variables provide a standard, platform-agnostic mechanism for injecting configuration at runtime without hardcoding secrets in source code or version control
- The pattern appears consistently across 4 distinct components (CLI tests, device management, web integration, and Chrome extension helpers), indicating an established architectural convention
- Modern deployment platforms (Docker, Kubernetes, serverless) and CI/CD pipelines natively support environment variable injection, making this approach operationally practical
- The facet classification as 'security.secrets_management' indicates this pattern specifically addresses secure handling of sensitive data rather than general configuration

## Problem Statement

How should the system securely manage and inject runtime configuration and secrets across heterogeneous execution environments (CLI tools, browser automation, device controllers, and test suites) without exposing sensitive data in source code or requiring complex configuration file management?

## Decision

1. MUST: All sensitive configuration data (API keys, tokens, credentials, service URLs with embedded secrets) MUST be sourced from environment variables rather than hardcoded in source files

## Policy Block

- MUST All sensitive configuration data (API keys, tokens, credentials, service URLs with embedded secrets) MUST be sourced from environment variables rather than hardcoded in source files

In scope:
- All runtime components requiring API keys or authentication tokens
- CLI tools and command-line interfaces
- Browser automation agents and Puppeteer launchers
- Device controllers and hardware interfaces
- Test suites requiring external service credentials
- Chrome extensions and browser integrations

Out of scope:
- Build-time configuration that does not contain secrets
- Public configuration values that can be safely committed to version control
- Static assets and resource files
- Type definitions and interface declarations

Exceptions:
- EXC-001: Test environments may use mock or placeholder values for environment variables when testing configuration loading logic itself
- EXC-002: Development environments may provide default non-production credentials for optional services to improve developer experience

## Rationale

- The pattern appears with 90.03% confidence across 4 distinct files spanning CLI, device management, web integration, and testing contexts, demonstrating consistent architectural adoption
- Environment variables are the industry-standard approach recommended by the Twelve-Factor App methodology for configuration management in modern cloud-native applications
- This approach enables the same codebase to run in multiple environments (development, staging, production) without code changes, improving deployment safety and operational flexibility
- Separating secrets from code reduces the risk of accidental exposure through version control, code sharing, or log aggregation systems

## Consequences

Positive:
- Secrets are never committed to version control, reducing the risk of credential leakage through repository access or git history
- The same application binary can be deployed across multiple environments with different configurations, simplifying CI/CD pipelines
- Integration with secret management systems (AWS Secrets Manager, HashiCorp Vault, Kubernetes Secrets) becomes straightforward as they all support environment variable injection
- Developer onboarding is simplified as environment setup is documented through required environment variables rather than complex configuration file formats

Negative:
- Environment variables are visible to all processes running under the same user, creating potential information disclosure risks in shared hosting environments
- Managing large numbers of environment variables across multiple services can become operationally complex without proper tooling
- Debugging configuration issues may be more difficult as values are not visible in the codebase and may vary between environments
- Some deployment platforms have limits on the number or size of environment variables, potentially constraining configuration complexity

## Alternatives

- Store configuration in encrypted JSON or YAML files committed to the repository (rejected)
  Rejected because: Requires key management for decryption, adds complexity, and still requires secure distribution of decryption keys. Environment variables are simpler and more universally supported.
  When valid: May be appropriate for complex hierarchical configuration that exceeds environment variable size limits
- Use a centralized configuration service (e.g., Consul, etcd) with runtime fetching (rejected)
  Rejected because: Adds infrastructure dependencies and network calls during startup. Increases system complexity and introduces additional failure modes. Environment variables provide sufficient functionality for current needs.
  When valid: Appropriate for large-scale microservices deployments requiring dynamic configuration updates without restarts
- Hardcode configuration with build-time substitution (rejected)
  Rejected because: Requires separate builds for each environment, violates immutable deployment principles, and makes emergency credential rotation difficult. Does not align with modern DevOps practices.
  When valid: Only acceptable for truly static, non-sensitive configuration that never changes between environments

## Risks

- Environment variables may be exposed through process listings, core dumps, or error reporting systems that capture process environment
  Mitigation: Use secret management systems that inject variables at runtime, implement process isolation, and configure error reporting to exclude environment data. Consider using file-based secrets for highly sensitive data.
  Owner: Security team and platform engineering
- Developers may accidentally log or print environment variable values during debugging, exposing secrets in log aggregation systems
  Mitigation: Implement automated log scanning for common secret patterns, provide secure debugging tools, and conduct security training on safe logging practices. Use linting rules to detect direct environment variable logging.
  Owner: Engineering team and security team
- Missing or misconfigured environment variables may cause runtime failures that are difficult to diagnose in production
  Mitigation: Implement comprehensive startup validation that checks all required variables, provide clear error messages, and maintain documentation of required variables per component. Use infrastructure-as-code to validate environment configuration.
  Owner: Platform engineering and SRE team

## Implementation Notes

- Create a centralized configuration module in each package that reads and validates environment variables at startup, providing a single source of truth for configuration access
- Document all required environment variables in a README or .env.example file at the package root, including descriptions, example values (non-sensitive), and whether they are required or optional
- Use TypeScript type definitions to enforce type safety when accessing configuration values, preventing runtime type errors from string environment variables
- Implement a startup validation phase that checks for required environment variables and fails fast with clear error messages before attempting to use the configuration
- For local development, support .env files using libraries like dotenv, but ensure .env files are in .gitignore and provide .env.example as a template

## Continuation Context


Verify commands:
- grep -r 'process\.env\.' --include='*.ts' --include='*.js' | grep -v 'test' | wc -l
- grep -r "['\"]\(API_KEY\|TOKEN\|SECRET\|PASSWORD\)['\"]\s*:\s*['\"]" --include='*.ts' --include='*.js' | grep -v 'process.env' | wc -l
- find . -name '.env' -not -path '*/node_modules/*' | xargs grep -l 'API_KEY\|TOKEN\|SECRET' 2>/dev/null | while read f; do grep -q "^$(basename $f)$" .gitignore || echo "$f not in gitignore"; done

Accept when:
- All sensitive configuration values (API keys, tokens, passwords) are accessed via process.env rather than hardcoded strings
- No .env files containing secrets are committed to version control (verified by .gitignore entries)
- Each package has documentation listing required environment variables with descriptions and example values
- Configuration modules validate required environment variables at startup and provide clear error messages for missing values

## Enforcement

- Verified by: Automated CI pipeline checks using grep patterns to detect hardcoded secrets
- Verified by: Code review checklist requiring verification that new configuration uses environment variables
- Verified by: Static analysis tools (e.g., ESLint rules) to detect direct string literals for sensitive configuration keys
- Verified by: Security scanning tools that detect potential credential exposure in source code
- Violation handling: CI pipeline fails if hardcoded secrets are detected in source files
- Violation handling: Pull requests containing potential violations are blocked until remediated
- Violation handling: Security team is notified of violations detected in production code
- Violation handling: Incidents of credential exposure trigger immediate credential rotation and security review
- Exception process: Developer submits exception request to tech lead with justification and risk assessment
- Exception process: Security team reviews exception for security implications
- Exception process: Approved exceptions are documented in code comments with expiration dates
- Exception process: Exceptions are reviewed quarterly and must be re-approved or remediated