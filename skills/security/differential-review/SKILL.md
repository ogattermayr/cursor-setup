---
name: differential-review
description: "Security-focused differential review of code changes (PRs, commits, diffs). Use when reviewing PRs, analyzing git diffs, or checking for security regressions. From Trail of Bits."
---

# Differential Security Review

Security-focused code review for PRs, commits, and diffs.

## Core Principles

1. **Risk-First**: Focus on auth, crypto, external calls, validation
2. **Evidence-Based**: Every finding backed by line numbers, attack scenarios
3. **Honest**: Explicitly state coverage limits and confidence level

## Rationalizations to Reject

| Rationalization | Why It's Wrong |
|-----------------|----------------|
| "Small PR, quick review" | Heartbleed was 2 lines |
| "Just a refactor" | Refactors break invariants |
| "Git history takes too long" | History reveals regressions |

## Quick Reference

### Risk Level Triggers

| Risk Level | Triggers |
|------------|----------|
| HIGH | Auth, crypto, external calls, validation removal |
| MEDIUM | Business logic, state changes, new public APIs |
| LOW | Comments, tests, UI, logging |

## Red Flags (Stop and Investigate)

- Removed code from "security", "CVE", or "fix" commits
- Access control removed (onlyOwner, auth checks)
- Validation removed without replacement
- External calls added without checks

## Common Vulnerability Patterns

### Security Regressions

```bash
# Check if removed code was security-related
git log -S "removed_function" --all --grep="security\|fix\|CVE"
```

**Red flags:**
- Commit message contains "security", "fix", "CVE"
- Code removed <6 months ago
- No explanation for re-addition

### Missing Validation

```bash
# Find removed validation
git diff HEAD~1 | grep "^-.*require\|assert\|raise\|throw"
```

Questions:
- Was validation moved elsewhere?
- Does removal expose vulnerability?

### Access Control Bypass

```bash
# Find removed auth checks
git diff HEAD~1 | grep "^-.*auth\|permission\|role\|admin"
```

Example:
```diff
- if not current_user.is_admin:
-     raise HTTPException(403)
  return sensitive_data
```

### Unchecked Return Values

```python
# DANGEROUS
result = external_api.call()
# proceed without checking result

# SAFE
result = external_api.call()
if not result.success:
    raise APIError(result.error)
```

## Detection Commands

```bash
# Find removed security checks
git diff HEAD~1 | grep "^-" | grep -E "auth|permission|validate|check"

# Find new external calls
git diff HEAD~1 | grep "^+" | grep -E "fetch|request|http|api"

# Find changed error handling
git diff HEAD~1 | grep -E "except|catch|error|throw"
```

## Review Workflow

### For Small Changes (<20 files)

1. Identify risk level per file
2. Git blame on any removed code
3. Check test coverage for changes
4. Document findings

### For Medium Changes (20-200 files)

1. Prioritize HIGH risk files
2. Full analysis on auth/crypto changes
3. Surface scan on MEDIUM risk
4. Calculate blast radius (who calls changed code?)

### For Large Changes (200+ files)

1. Focus on critical paths only
2. Auth system changes get deep review
3. State explicit coverage limitations

## Quality Checklist

Before completing review:

- [ ] All changed files analyzed
- [ ] Git blame on removed security code
- [ ] Attack scenarios are concrete (not generic)
- [ ] Findings reference specific lines
- [ ] Coverage limitations stated

## Integration with HowTheF*

### Backend (FastAPI/Python)

Check for:
- `crud_service` filter strings with user input
- Missing auth dependencies on endpoints
- Removed Pydantic validation
- Exception handling that swallows errors

### Frontend (Next.js/React)

Check for:
- Removed auth checks in API routes
- `dangerouslySetInnerHTML` with user content
- Missing input validation
- Direct Supabase calls from client (should use API routes)

---

*Based on Trail of Bits differential-review skill*
