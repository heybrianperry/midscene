# Sanitize User Input in Logging Statements to Prevent Log Injection: Raw User Input

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all logging operations that include user-controlled input or external data sources.

## Context

- The codebase processes user-provided input and external data that flows through logging systems, creating potential security vulnerabilities
- Log injection attacks can occur when untrusted input containing newlines, control characters, or format specifiers is written directly to logs without sanitization
- A consistent pattern of input validation before logging has been detected across 3 files with 91.80% confidence, indicating an established architectural practice
- The security.input_validation facet indicates this pattern specifically addresses security concerns related to validating data before it enters the logging subsystem
- Modern observability platforms parse structured logs, making them vulnerable to injection attacks that can corrupt log data, bypass security monitoring, or inject false audit trails

## Problem Statement

Without proper input sanitization, user-controlled data or external input written to application logs can contain malicious payloads including newline characters, ANSI escape sequences, or format string specifiers that corrupt log integrity, evade security monitoring, forge audit trails, or exploit log processing systems downstream.

## Decision

1. MUST_NOT: Raw user input MUST NOT be directly interpolated into log format strings without validation

## Policy Block

- MUST_NOT Raw user input MUST NOT be directly interpolated into log format strings without validation

In scope:
- All logging statements that include user-provided input (form data, query parameters, request bodies)
- Log entries containing data from external APIs, file uploads, or third-party integrations
- Error messages that echo back user input for debugging purposes
- Audit logs recording user actions or system events with user-controlled fields
- Structured logging fields populated from untrusted sources

Out of scope:
- Logging of internally-generated identifiers, timestamps, or system constants
- Debug logs in development environments with explicit security warnings (though sanitization is still recommended)
- Logs written to isolated, non-parsed storage systems with no downstream processing (rare edge case)

Exceptions:
- EXC-001: Security incident response requires preserving exact raw input for forensic analysis

## Rationale

- Pattern detected across 3 files with 91.80% confidence indicates this is an established architectural practice that has proven effective in preventing log injection vulnerabilities
- Input validation at the logging boundary provides defense-in-depth by catching malicious payloads even if earlier validation layers are bypassed
- Centralized sanitization in logging utilities ensures consistent security posture across the entire application without requiring developers to remember validation rules at every call site
- This approach aligns with OWASP logging security best practices and prevents common attack vectors including log forging, CRLF injection, and log poisoning

## Consequences

Positive:
- Prevents log injection attacks that could corrupt audit trails, evade security monitoring, or exploit log analysis tools
- Maintains log integrity and trustworthiness for security investigations and compliance audits
- Reduces attack surface by eliminating a common vulnerability class across the entire application
- Centralized sanitization reduces cognitive load on developers who no longer need to remember to validate input at every logging call

Negative:
- Adds processing overhead to every logging operation that includes external data, potentially impacting performance in high-throughput scenarios
- Sanitized logs may lose some fidelity compared to raw input, potentially complicating debugging in edge cases
- Requires ongoing maintenance to ensure sanitization logic keeps pace with new attack vectors and logging formats
- May create false sense of security if sanitization is incomplete or bypassed through alternative logging paths

## Alternatives

- Rely on downstream log processing systems to sanitize input during ingestion (rejected)
  Rejected because: Creates dependency on external systems, increases attack window, and fails defense-in-depth principle by not validating at the source
  When valid: Only acceptable when logging to isolated systems with guaranteed sanitization and no direct log file access
- Use structured logging exclusively with typed fields that automatically escape values (deferred)
  Rejected because: While beneficial, this is a larger architectural change that doesn't address legacy logging code or unstructured log messages
  When valid: Should be adopted as a complementary long-term strategy alongside input sanitization
- Implement validation at data ingestion boundaries only, before data enters the application (rejected)
  Rejected because: Fails to protect against internal data corruption, doesn't handle data transformations that occur after ingestion, and violates defense-in-depth
  When valid: Should be used as a complementary first layer, but not as a replacement for logging-level sanitization

## Risks

- Incomplete sanitization logic may miss novel attack vectors or encoding variations, providing false security
  Mitigation: Regularly review and update sanitization rules based on OWASP guidance, security advisories, and penetration testing results. Implement comprehensive test suite covering known injection techniques.
  Owner: Security team with engineering team support
- Performance degradation in high-throughput logging scenarios due to sanitization overhead
  Mitigation: Implement efficient sanitization algorithms, consider caching for repeated values, and use asynchronous logging where appropriate. Monitor logging performance metrics.
  Owner: Engineering team
- Developers may bypass sanitization utilities by using direct logging methods or creating alternative logging paths
  Mitigation: Enforce through code review, static analysis tools, and linting rules. Provide clear documentation and make sanitized logging the path of least resistance.
  Owner: Engineering team leads

## Implementation Notes

- Create centralized sanitization utilities (e.g., sanitizeForLog() function) that handle common injection vectors including newlines, carriage returns, ANSI codes, and format specifiers
- Integrate sanitization into existing logging wrappers or create new wrapper functions that automatically sanitize parameters before passing to underlying log libraries
- Document which logging methods require manual sanitization vs. which provide automatic sanitization, and establish clear naming conventions (e.g., logSafe() vs. logRaw())
- Consider implementing a whitelist approach for known-safe characters rather than blacklist approach for dangerous characters, as whitelists are more resilient to novel attacks

## Continuation Context


Verify commands:
- grep -r 'console\.log\|logger\.' packages/ | grep -v 'sanitize\|escape\|safe' | head -20
- npm run test:security -- --grep 'log injection'
- eslint packages/ --rule 'no-unsanitized-logging: error' --format json

Accept when:
- All logging statements that include user input or external data pass through sanitization functions
- Security test suite includes log injection test cases that verify newlines, control characters, and ANSI codes are properly escaped
- Static analysis or linting rules flag unsanitized logging calls during CI/CD pipeline

## Enforcement

- Verified by: Automated static analysis and linting rules in CI/CD pipeline that detect unsanitized logging patterns
- Verified by: Security-focused code review checklist that explicitly verifies input sanitization in logging statements
- Verified by: Regular penetration testing and security audits that include log injection attack scenarios
- Violation handling: CI/CD pipeline fails if static analysis detects unsanitized user input in logging statements
- Violation handling: Code review process blocks merge requests that introduce unsanitized logging of external data
- Violation handling: Security team creates tickets for remediation of violations discovered during audits, prioritized by risk level
- Exception process: Developer submits exception request to security team with justification and risk assessment
- Exception process: Security team reviews request and may approve with compensating controls (e.g., restricted log access, additional monitoring)
- Exception process: Approved exceptions are documented in code comments with ticket reference and expiration date for review