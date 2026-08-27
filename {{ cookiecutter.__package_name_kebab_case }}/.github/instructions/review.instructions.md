---
applyTo: '**/*.py'
---

## Code Review Guidelines

### Overview
Triggered when user asks for code review. Can be scoped to:
- Specific directory/module
- Current git diff (staged + working changes)
- Entire codebase
- Individual files or functions

**Clarify ambiguous scope** before proceeding.

### Review Categories

Use the following severity categories for issues:

| Category | Definition | Action Required |
|----------|-----------|-----------------|
| 🔴 **Critical** | Breaks functionality, security issue, data loss risk, crashes in prod | Must fix before merge |
| 🟠 **Important** | Violates guidelines, poor performance, maintainability concern, missing validation | Should fix before merge |
| 🟡 **Improvement** | Code quality enhancement, optimization opportunity, better naming, documentation gaps | Nice to have, consider for next iteration |
| 🟢 **Good** | Follows best practices, well-structured, good testing, clear documentation | Highlight as positive |


### Review Methodology

1. **Gather Context** - Read relevant files, check `coding_guidelines.instructions.md` and any other relevant instruction files.
2. **Evaluate** - Type hints, naming, docstrings, error handling, security, data validation, testing
3. **Identify Issues** - Runtime errors, performance, security, duplication, missing validation, API design, breaking changes
4. **Run Validation** - Execute `poe check`, `poe check_licenses`, `poe test`

### Review Output Format

```markdown
## Code Review: [Scope]

### Summary
- **Files Reviewed**: X files, Y lines
- **Issues**: Z critical, A important, B improvements
- **Assessment**: [Brief summary]
- **Test Coverage**: X% (was Y%) [↑/↓]
- **Validation**: ✅ `poe check`, `poe check_licenses`, `poe test` all passed

### Findings Table
| Priority | Category | Location | Issue | Fix |
|----------|----------|----------|-------|-----|
| 🔴 | Security | `src/auth.py:45` | Hardcoded API key | Use `os.getenv("API_KEY")` |
| 🟠 | Type Hint | `src/process.py:12` | Missing return type | Add `-> dict[str, int]` |

### Detailed Analysis
Group by priority: 🔴 Critical, 🟠 Important, 🟡 Improvements, 🟢 Strengths

### Version Bump Recommendation
**Type (X.Y.Z)**: Rationale based on changes

### Action Items
- [ ] Address critical and important issues with actionable suggestions
- [ ] Review version bump recommendation
```

### Key Review Checks

| Aspect | Checks |
|--------|--------|
| **Type Hints** | All parameters & returns annotated, Python 3.12+ syntax |
| **Naming** | snake_case functions/variables, PascalCase classes, UPPERCASE constants |
| **Docstrings** | Google style with Args, Returns, Raises sections |
| **Error Handling** | Meaningful exceptions, descriptive messages, no silent failures |
| **Security** | No hardcoded secrets, validated inputs, parameterized queries |
| **Data** | Pandera schemas, schema attribute access (not hardcoded strings) |
| **Testing** | Tests for critical paths, fixtures for reuse, 70%+ coverage |
| **Performance** | No O(n²) on large data, vectorized operations, efficient DataFrame usage |
| **Duplication** | DRY principle followed, extraction opportunities noted |

### Git Diff Reviews

When reviewing `git diff` (current changes):
- Identify changed files and understand intent
- Check for side effects on other modules
- Verify breaking changes don't require test updates
- Assess if change is well-scoped or could be split further

### When Unclear
Ask: "Is this intentional?", "What's the performance target?", "Should this be tested?"

### Automated Validation

*Must* run: `poe check`, `poe check_licenses`, `poe test`

**Report**: Coverage %, license violations (🔴), formatting issues (🟠), test failures (🔴)

### Edge Cases
- **Defer**: Stylistic issues, large refactors (unless critical)
- **Never defer**: Security, data integrity, type safety
