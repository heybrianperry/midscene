# Adopt React Component Export Pattern with Named Index Exports: Consumers Not Import

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all component library development and applies to all React-based UI packages within the monorepo.

## Context

- The codebase contains multiple React component packages (visualizer, recorder) that require consistent module organization and export patterns for maintainability and developer experience
- Component libraries need clear public API contracts that separate implementation details from exported interfaces, enabling better encapsulation and refactoring flexibility
- The pattern was detected across 7 files with 87.14% confidence, indicating strong consistency in how components are structured and exported throughout the visualizer and recorder packages
- Modern React development practices favor index.tsx barrel exports that provide clean import paths and hide internal component structure from consumers
- The monorepo architecture requires standardized module boundaries to support tree-shaking, code splitting, and clear dependency management across packages

## Problem Statement

Without a standardized component export pattern, React component libraries suffer from inconsistent import paths, unclear public API boundaries, and difficulty maintaining backward compatibility. Developers face confusion about which components are intended for external use versus internal implementation details, leading to tight coupling and brittle refactoring.

## Decision

1. MUST_NOT: Consumers MUST NOT import directly from implementation files, bypassing the index.tsx export layer

## Policy Block

- MUST_NOT Consumers MUST NOT import directly from implementation files, bypassing the index.tsx export layer

In scope:
- All React component packages within the monorepo (packages/visualizer, packages/recorder)
- TypeScript/TSX component files that define reusable UI components
- Component directories that contain index.tsx or index.ts files serving as module entry points
- Public API surfaces for component libraries intended for cross-package consumption

Out of scope:
- Internal utility functions or hooks that are not component-specific
- Test files and test utilities
- Build configuration and tooling scripts
- Native C++ modules (e.g., rdp package) which follow different export conventions
- Single-file components that don't require directory-based organization

Exceptions:
- EXC-001: A component is explicitly marked as internal-only and documented as not part of the public API
- EXC-002: Legacy components being gradually migrated to the new pattern

## Rationale

- The pattern appears consistently across 7 files in the visualizer and recorder packages, demonstrating established practice and team consensus on this approach
- Index-based exports provide a stable public API surface that allows internal refactoring without breaking consumer imports
- Named exports improve developer experience through better IDE support, explicit imports, and easier tree-shaking for bundlers
- This pattern aligns with React ecosystem best practices and is widely adopted in popular component libraries like Material-UI and Ant Design

## Consequences

Positive:
- Clear separation between public API and implementation details enables safer refactoring and evolution of component internals
- Consistent import patterns across the codebase improve developer onboarding and reduce cognitive load
- Better tree-shaking and code splitting opportunities as bundlers can analyze explicit named exports
- Enhanced IDE autocomplete and refactoring capabilities when using named exports over default exports

Negative:
- Additional boilerplate required for each component directory (index.tsx file)
- Potential for confusion when developers must remember to update index.tsx when adding new exports
- Slightly deeper file nesting may impact navigation in some development environments
- Migration effort required for existing components not following this pattern

## Alternatives

- Use default exports for all components without index files (rejected)
  Rejected because: Default exports provide poor IDE support, make refactoring difficult, and don't scale well for components that need to export multiple related items (component + types + utilities)
  When valid: May be acceptable for single-file utility components that will never need additional exports
- Flat file structure with all components in a single directory (rejected)
  Rejected because: Doesn't scale for large component libraries, makes it difficult to co-locate related files (styles, tests, sub-components), and provides no encapsulation of internal implementation details
  When valid: Suitable only for very small libraries with fewer than 10 simple components
- Barrel exports at package level only, without per-component index files (rejected)
  Rejected because: Loses the benefit of component-level encapsulation and makes it harder to understand what constitutes a component's public API versus its internal structure
  When valid: Could work for packages with very few components where all files are considered public API

## Risks

- Developers may forget to update index.tsx when adding new exports, leading to components that exist but aren't accessible
  Mitigation: Implement linting rules to detect exported components not re-exported through index files, and include this check in CI pipeline
  Owner: Engineering team
- Circular dependency issues may arise if index files are not carefully managed
  Mitigation: Use dependency-cruiser or similar tools to detect circular dependencies, establish clear component hierarchy guidelines
  Owner: Engineering team
- Inconsistent adoption across the codebase during migration period may cause confusion
  Mitigation: Create migration guide, prioritize high-traffic components first, use deprecation warnings for old import paths
  Owner: Technical lead

## Implementation Notes

- Create a component template or generator script that scaffolds the correct directory structure with index.tsx automatically
- For existing components, gradually migrate by creating index.tsx files and adding deprecation notices to direct imports
- Document the pattern in the project's contributing guidelines with clear examples from the visualizer package
- Consider using TypeScript path aliases to make imports cleaner (e.g., @visualizer/components/Logo instead of ../../../component/logo)

## Continuation Context


Verify commands:
- find packages/*/src/component* -name 'index.tsx' -o -name 'index.ts' | wc -l
- grep -r "export.*from.*component.*index" packages/visualizer packages/recorder
- eslint --rule 'no-restricted-imports: [error, {patterns: ["**/component/*/*.tsx", "!**/component/*/index.tsx"]}]' packages/

Accept when:
- All component directories in packages/visualizer/src/component and packages/recorder/src/components contain an index.tsx or index.ts file
- Grep search confirms that component imports use the index pattern rather than direct file imports
- ESLint rules pass confirming no restricted direct imports to component implementation files

## Enforcement

- Verified by: Automated ESLint rules checking for direct imports bypassing index files
- Verified by: Code review checklist requiring index.tsx for new components
- Verified by: CI pipeline verification commands checking component export patterns
- Violation handling: ESLint violations block PR merge in CI pipeline
- Violation handling: Code review feedback requests changes to follow the pattern
- Violation handling: Automated bot comments on PRs suggesting the correct import pattern when violations detected
- Exception process: Developer documents exception rationale in PR description with reference to specific exception ID (EXC-001 or EXC-002)
- Exception process: Package maintainer or technical lead reviews and approves exception in PR review
- Exception process: Exception is documented in code with JSDoc @internal tag or tracking issue link