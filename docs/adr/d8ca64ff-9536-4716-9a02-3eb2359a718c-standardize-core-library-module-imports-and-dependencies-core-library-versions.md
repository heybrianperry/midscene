# Standardize Core Library Module Imports and Dependencies: Core Library Versions

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains 11 files across multiple packages (visualizer, recorder, computer, cli, chrome-extension) that exhibit consistent library import and module dependency patterns
- Files span different technology stacks including TypeScript/React components (.tsx), native C++ modules (.cpp, .hpp), and JavaScript test files (.mjs), indicating a polyglot architecture
- The pattern signature 352185f1ce9ffb39c86c964ce8959ec4 appears consistently across frontend components, browser extensions, native bindings, and CLI tooling with 88.18% confidence
- Core library dependencies are being used for fundamental functionality including UI rendering (React components), native protocol handling (RDP), and test infrastructure
- The detection of this pattern across diverse file types suggests an established architectural convention for how core libraries and modules should be imported and organized

## Problem Statement

Without standardized guidelines for core library imports and module dependencies, the codebase risks inconsistent dependency management, version conflicts, circular dependencies, and unclear module boundaries across the polyglot architecture spanning TypeScript, C++, and JavaScript environments.

## Decision

1. SHOULD: Core library versions SHOULD be managed centrally through workspace-level dependency management (e.g., pnpm workspace, yarn workspaces)

## Policy Block

- SHOULD Core library versions SHOULD be managed centrally through workspace-level dependency management (e.g., pnpm workspace, yarn workspaces)

In scope:
- All TypeScript and JavaScript files in packages/visualizer, packages/recorder, packages/cli, and apps/*
- All C++ native module files in packages/computer/native/*
- React component files (.tsx) across all packages
- Test files (.mjs, .test.ts, .spec.ts) that import core testing libraries
- Package manifest files (package.json, CMakeLists.txt) declaring core dependencies

Out of scope:
- Third-party library code in node_modules
- Generated code or build artifacts
- Vendor-specific SDK files that have their own import conventions
- Legacy code marked for refactoring with explicit technical debt tickets

Exceptions:
- EX-20-001: Dynamic imports are required for code splitting or lazy loading in frontend bundles
- EX-20-002: Native C++ modules require platform-specific conditional imports

## Rationale

- The pattern was detected across 11 files with 88.18% confidence, indicating a strong existing convention that should be formalized
- Consistent library import patterns reduce cognitive load for developers working across different packages and technology stacks
- Explicit dependency management prevents version conflicts and makes dependency graphs easier to analyze and optimize
- Standardizing import ordering and module organization improves code readability and makes automated tooling (linters, bundlers) more effective

## Consequences

Positive:
- Improved code consistency across the polyglot codebase spanning TypeScript, JavaScript, and C++
- Easier onboarding for new developers who can rely on predictable import patterns
- Better support for automated dependency analysis and security vulnerability scanning
- Reduced risk of circular dependencies and module resolution issues
- Clearer separation between core library dependencies and application-specific code

Negative:
- Existing code may require refactoring to comply with standardized import patterns
- Additional CI checks and linting rules will be needed to enforce the standards
- Developers must learn and follow the import ordering conventions
- May require updates to build tooling and bundler configurations to align with the standards

## Alternatives

- Allow each package to define its own library import conventions independently (rejected)
  Rejected because: Would lead to inconsistent patterns across the monorepo and increase cognitive load when switching between packages
  When valid: Only valid for isolated packages that will never share code or developers with the rest of the monorepo
- Use automatic import sorting tools without documented standards (rejected)
  Rejected because: Tools alone don't capture architectural intent or handle edge cases like native module imports; explicit standards provide context
  When valid: Could be used as a complementary enforcement mechanism after standards are documented
- Adopt a strict dependency injection pattern to minimize direct library imports (deferred)
  Rejected because: Would require significant architectural changes; current pattern is working well with formalization
  When valid: Could be reconsidered if the codebase grows significantly or if testing becomes difficult due to tight coupling

## Risks

- Large-scale refactoring required to bring existing code into compliance may introduce bugs
  Mitigation: Implement changes incrementally with comprehensive test coverage; use automated tools (ESLint, clang-format) to minimize manual changes
  Owner: Engineering team leads
- Native C++ module import patterns may conflict with platform-specific requirements
  Mitigation: Document platform-specific exceptions clearly; involve platform team in standard definition; use preprocessor directives appropriately
  Owner: Platform/native team
- Overly strict import rules may hinder rapid prototyping and experimentation
  Mitigation: Provide clear exception process for experimental code; allow relaxed rules in designated prototype directories
  Owner: Engineering team

## Implementation Notes

- Configure ESLint with import ordering rules (eslint-plugin-import) for TypeScript/JavaScript files to enforce R-20-001 and R-20-006
- Set up clang-format or similar tooling for C++ files with consistent include ordering
- Create a monorepo-wide dependency management strategy using pnpm workspaces or similar to centralize core library versions
- Add pre-commit hooks that validate import ordering and flag circular dependencies
- Document the standard import order in CONTRIBUTING.md with examples for each file type (tsx, ts, cpp, hpp, mjs)
- Create barrel files (index.ts) in shared packages to simplify common import patterns while maintaining explicit dependencies

## Continuation Context


Verify commands:
- eslint --ext .ts,.tsx,.js,.mjs --rule 'import/order: error' packages/ apps/
- grep -r "^import.*from" packages/visualizer/src packages/recorder/src apps/ | head -20
- find packages/ apps/ -name 'package.json' -exec jq '.dependencies, .devDependencies' {} \;
- madge --circular --extensions ts,tsx,js packages/ apps/

Accept when:
- ESLint import ordering rules pass for all TypeScript/JavaScript files with no violations
- No circular dependencies detected by madge or similar dependency analysis tools
- All package.json files declare core library dependencies explicitly with no implicit peer dependencies
- Code review checklist includes verification of import ordering and dependency declarations

## Enforcement

- Verified by: Automated ESLint checks in CI pipeline for TypeScript/JavaScript import ordering
- Verified by: Madge or similar tool in CI to detect circular dependencies
- Verified by: Code review checklist requiring verification of import patterns and dependency declarations
- Verified by: Pre-commit hooks that run import ordering validation
- Violation handling: CI build fails if ESLint import ordering rules are violated
- Violation handling: Pull requests blocked if circular dependencies are detected
- Violation handling: Automated comments on PRs highlighting import order violations with suggested fixes
- Violation handling: Monthly dependency audit reports identifying packages with undeclared or inconsistent core library versions
- Exception process: Developer creates exception request documenting the specific requirement and justification
- Exception process: Tech lead or platform team reviews exception based on policy_exceptions criteria
- Exception process: Approved exceptions documented in code comments with ticket reference
- Exception process: Exceptions reviewed quarterly to determine if they can be eliminated or should become permanent