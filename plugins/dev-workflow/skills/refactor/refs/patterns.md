# Refactoring Patterns

Language-agnostic refactoring patterns for reference during iterative refactoring.

## Structural Patterns

| Pattern | Before | After |
|---------|--------|-------|
| Long Method | Single method with 50+ lines | Multiple methods, each ≤ 20 lines |
| Deep Nesting | 3+ levels of nested conditionals | Early returns, flat structure |
| Duplicate Code | Same logic in 3+ places | Extracted to a shared function |
| God Class | Class with 20+ methods / multiple responsibilities | Split into focused classes by responsibility |
| Feature Envy | Method extensively using another class's data | Move method to the appropriate class |
| Long Parameter List | Function with 5+ parameters | Parameter object or builder pattern |

## Readability Patterns

| Pattern | Before | After |
|---------|--------|-------|
| Magic Numbers | `if (status == 3)` | `if (status == STATUS_ACTIVE)` |
| Unclear Naming | `d`, `tmp`, `data2` | `daysSinceLastLogin`, `pendingOrders` |
| Complex Conditional | `if (a && (b || c) && !d)` | Extract to named boolean or method |
| Nested Ternary | `a ? b ? c : d : e` | if/else or extracted variable |
| Long Expression | Chained operations on one line | Break into named intermediate variables |

## Convention Patterns

| Pattern | Before | After |
|---------|--------|-------|
| Inconsistent Style | Mixed naming conventions | Unified with project conventions |
| Missing Error Handling | Bare try/catch or ignored errors | Proper error handling per project patterns |
| Primitive Obsession | Using strings for domain concepts | Value objects, enums, or typed wrappers |
| Implicit Dependencies | Hardcoded dependencies | Dependency injection or configuration |

## Test Code Patterns

| Pattern | Before | After |
|---------|--------|-------|
| Ambiguous Assertions | `expect(x).to be >= 1` | `expect(x).to eq(expected_value)` |
| Test Duplication | Repeated setup across tests | Shared fixtures or helpers |
| Missing Edge Cases | Only happy path tested | Happy path + edge cases + error cases |
| Fragile Tests | Tests dependent on implementation details | Tests focused on behavior and contracts |
