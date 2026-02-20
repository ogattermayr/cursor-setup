# Security Audit

Run a comprehensive security audit on the codebase or specific files.

## Scope

Analyze the provided files or the entire codebase for security vulnerabilities.

## Audit Steps

1. **Secrets Detection** - Check for hardcoded API keys, passwords, tokens
2. **Authentication Review** - Verify all endpoints have proper auth checks
3. **Authorization Check** - Ensure users can only access their own data
4. **Input Validation** - Confirm Pydantic models validate all inputs
5. **SQL Injection** - Look for unsafe string interpolation in queries
6. **XSS Prevention** - Check for `dangerouslySetInnerHTML` and dynamic URLs
7. **Sensitive Data** - Ensure no PII/secrets in logs or responses
8. **Dependencies** - Note any known vulnerable packages

## Report Format

For each finding, report:

### [SEVERITY] - Issue Title

**Location:** `file/path.py:line`

**Issue:** Description of the vulnerability

**Risk:** What could happen if exploited

**Fix:** How to remediate

---

## Severity Levels

- **CRITICAL** - Exploitable now, data breach risk → Fix immediately
- **HIGH** - Significant risk → Fix before deploy
- **MEDIUM** - Should be fixed → Fix within sprint  
- **LOW** - Best practice → Fix when convenient

## Example Output

### [HIGH] - Missing Auth Check on User Endpoint

**Location:** `features/users/api/users_api.py:45`

**Issue:** The `get_user_details_endpoint` does not verify the requesting user has permission to view the target user's data.

**Risk:** Any authenticated user could view any other user's profile data.

**Fix:** Add ownership check:
```python
if current_user.id != user_id and not current_user.is_admin:
    raise HTTPException(403, "Not authorized")
```
