<rule_activation id="92e80def-50cc-4cdd-a7b5-e1f052abf3c0" title="Standardize Core Library Module Imports and Dependencies: Modules Not Create" applies_to="**/*">
These rules are ALWAYS ACTIVE for all TypeScript, JavaScript, C++, and related files across packages/visualizer, packages/recorder, packages/cli, apps/*, and packages/computer/native/.
</rule_activation>

### Rules

- **R-20-001** MUST_NOT: Modules MUST NOT create circular dependencies between core libraries and application code.

### Verify

```bash
# Check ESLint import ordering for TypeScript/JavaScript files
eslint --ext .ts,.tsx,.js,.mjs --rule 'import/order: error' packages/ apps/

# Inspect import statements across key packages
grep -r "^import.*from" packages/visualizer/src packages/recorder/src apps/ | head -20

# Verify core library dependencies are declared in package.json files
find packages/ apps/ -name 'package.json' -exec jq '.dependencies, .devDependencies' {} \;

# Detect circular dependencies using madge
madge --circular --extensions ts,tsx,js packages/ apps/
```

**Accept when:**
- ESLint import ordering rules pass for all TypeScript/JavaScript files with no violations
- No circular dependencies detected by madge or similar dependency analysis tools
- All package.json files declare core library dependencies explicitly with no implicit peer dependencies
- Code review checklist includes verification of import ordering and dependency declarations

<enforcement>
Claude Code MUST NOT skip or defer verification. Circular dependency detection and import ordering validation are mandatory before accepting changes to this codebase.
</enforcement>