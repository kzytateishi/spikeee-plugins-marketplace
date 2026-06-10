# Refactoring Report

## Target
- **Branch**: {branch_name}
- **Files**: {file_count} files
- **Rounds completed**: {round_count}

## Changes by Round

### Round 1: Structural Improvements
| File | Change | Rationale |
|------|--------|-----------|
| {path} | {change} | {reason} |

### Round 2: Readability
| File | Change | Rationale |
|------|--------|-----------|
| {path} | {change} | {reason} |

### Round 3: Convention Compliance
| File | Change | Rationale |
|------|--------|-----------|
| {path} | {change} | {reason} |

## Before / After Examples

### {file_path}
```
# Before
{original_code}

# After
{refactored_code}
```

## Needs Review (not refactored)
Correctness/edge smells noticed while refactoring — hand these to `/code-review` (behavior-preserving refactor does not fix them):
| File:Line | Smell | Category |
|-----------|-------|----------|
| {path}:{line} | {smell} | nullable / race / time-boundary / index / escaping / ... |

## Quality Verification
- Tests: PASS / FAIL
- Lint: PASS / FAIL
- Type Check: PASS / FAIL

## Summary
- Duplications removed: {count}
- Methods split: {count}
- Names improved: {count}
- Convention violations fixed: {count}
