# Standardize Public API Export Patterns for External Integration: Breaking Changes Public

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains multiple packages (visualizer, cli, playground, computer) that expose public APIs for external consumption and integration
- Pattern detected across 13 files with 89.48% confidence indicates a consistent approach to structuring external-facing API surfaces
- Components like blackboard, player, nav-actions, and universal-playground require stable, well-defined interfaces for third-party integration
- The architecture separates internal implementation details from public API contracts to maintain backward compatibility and clear boundaries
- Cross-package dependencies and external tooling (RDP helper, rsbuild config) necessitate explicit API design patterns

## Problem Statement

Without standardized patterns for public API design and export, external integrators face inconsistent interfaces, unclear stability guarantees, and difficulty understanding which APIs are intended for public consumption versus internal use. This leads to brittle integrations, breaking changes, and increased maintenance burden.

## Decision

1. SHOULD: Breaking changes to public APIs SHOULD be deprecated for at least one major version before removal, with clear migration paths documented

## Policy Block

- SHOULD Breaking changes to public APIs SHOULD be deprecated for at least one major version before removal, with clear migration paths documented

In scope:
- All packages intended for external consumption (visualizer, cli, playground, computer)
- Public-facing functions, classes, types, and interfaces exported from package entry points
- APIs documented in public documentation or README files
- Cross-package interfaces used by external integrators or third-party tools

Out of scope:
- Internal utility functions not exported from package root
- Private implementation details within package subdirectories
- Test utilities and fixtures
- Development-only tooling and build configuration internals

Exceptions:
- EXC-001: Experimental or alpha-stage APIs that require rapid iteration without stability guarantees
- EXC-002: Internal APIs that must be shared across packages within the monorepo but not intended for external use

## Rationale

- Pattern detected across 13 files with 89.48% confidence indicates this is an established architectural practice worth codifying
- Consistent public API patterns reduce integration friction and enable external developers to build reliable extensions and integrations
- Explicit separation between public and internal APIs allows internal refactoring without breaking external consumers
- TypeScript-based type definitions provide compile-time safety and excellent IDE support for API consumers

## Consequences

Positive:
- External integrators gain clear, stable interfaces with predictable evolution patterns
- Internal teams can refactor implementation details without fear of breaking external consumers
- Improved developer experience through consistent API patterns across all packages
- Reduced support burden from breaking changes and unclear API boundaries

Negative:
- Additional overhead in maintaining separate public API surfaces and internal implementations
- Potential duplication when internal and public APIs serve similar purposes
- Requires discipline to avoid accidentally exposing internal details through public interfaces
- May slow down rapid prototyping when experimental features need public exposure

## Alternatives

- Export all functions and classes directly without explicit public API boundaries (rejected)
  Rejected because: Creates tight coupling between internal implementation and external consumers, making refactoring extremely difficult and leading to frequent breaking changes
  When valid: Only appropriate for internal-only packages with no external consumers
- Use a facade pattern with a single unified API gateway for all packages (rejected)
  Rejected because: Creates a bottleneck and reduces package independence; doesn't align with the monorepo structure where packages have distinct purposes
  When valid: Could be considered for a future unified SDK layer that wraps multiple packages
- Rely solely on documentation to indicate public vs internal APIs without structural enforcement (rejected)
  Rejected because: Documentation-only approaches are easily violated and provide no compile-time or runtime enforcement of boundaries
  When valid: May be acceptable for very small projects with single maintainers

## Risks

- Developers may accidentally expose internal APIs through public exports, creating unintended dependencies
  Mitigation: Implement automated linting rules and code review checklists to verify public API exports; use TypeScript's @internal JSDoc tag and API extractor tools
  Owner: Engineering team and package maintainers
- Maintaining backward compatibility may constrain internal refactoring and technical debt reduction
  Mitigation: Establish clear deprecation policies with migration guides; use adapter patterns to bridge old and new implementations during transition periods
  Owner: Architecture team
- Inconsistent application of public API patterns across packages may confuse external integrators
  Mitigation: Create package templates and scaffolding tools that enforce consistent structure; conduct regular architecture reviews
  Owner: Developer experience team

## Implementation Notes

- Use TypeScript's 'export' statements in index.ts files at package roots to create explicit public API surfaces
- Leverage API Extractor or similar tools to generate API reports and detect unintended breaking changes in CI/CD pipelines
- Consider using the 'exports' field in package.json to enforce module boundaries and prevent deep imports into internal directories
- Document public APIs using JSDoc with @public, @internal, @experimental, and @deprecated tags to clearly communicate stability levels

## Continuation Context


Verify commands:
- grep -r "export.*from" packages/*/src/index.ts | wc -l
- find packages -name 'index.ts' -type f | xargs grep -L 'export' && echo 'Missing exports found' || echo 'All index files have exports'
- npx api-extractor run --local --verbose 2>&1 | grep -i 'error\|warning' || echo 'API surface validated'

Accept when:
- All packages intended for external use have an index.ts or equivalent public API entry point with explicit exports
- API extractor or equivalent tooling runs successfully in CI without detecting unintended breaking changes
- Public API exports include TypeScript type definitions and JSDoc documentation for all exported members

## Enforcement

- Verified by: Automated CI checks using API Extractor or similar tools to detect API surface changes
- Verified by: Code review checklist requiring verification of public API boundaries for all package changes
- Verified by: TypeScript compiler strict mode to enforce type safety on exported APIs
- Violation handling: CI pipeline fails if API surface changes are detected without corresponding version bump
- Violation handling: Pull requests exposing internal APIs without @internal tags are flagged for revision
- Violation handling: Breaking changes without deprecation period require architecture team approval and major version bump
- Exception process: Submit exception request to architecture team with justification for deviation from standard pattern
- Exception process: Document exception in package README and ADR updates section
- Exception process: Set timeline for bringing exceptional cases into compliance or establishing new pattern if broadly applicable