---
name: security-audit
description: Security vulnerability detection for web applications. Use when reviewing code for security issues, checking for hardcoded secrets, analyzing authentication flows, or auditing API endpoints.
---

# Security Audit Skill

Use this skill to perform security audits on the HowTheF* codebase.

## When to Use

- Before deploying new features
- When implementing authentication/authorization
- When handling sensitive data (PII, payments, API keys)
- When reviewing API endpoints
- When adding new dependencies

## Audit Categories

### 1. Secrets Detection (CRITICAL)

Check for hardcoded credentials:

```python
# PATTERNS TO FLAG:
api_key = "sk-..."           # Hardcoded API key
password = "admin123"        # Hardcoded password
secret = "..."               # Any variable named secret with literal
AWS_ACCESS_KEY = "AKIA..."   # AWS credentials
SUPABASE_KEY = "eyJ..."      # Supabase keys

# SAFE PATTERNS:
api_key = os.getenv("API_KEY")
password = settings.PASSWORD
```

**Files to check:**
- `*.py` - Python source files
- `*.env.example` - Should not contain real values
- `*.json` - Config files
- `.env` files should be in `.gitignore`

### 2. SQL Injection (HIGH)

Even with `crud_service`, check for unsafe patterns:

```python
# DANGEROUS - String interpolation in filter
filter_string = f"name=eq.{user_input}"  # User input directly in filter

# SAFER - Validate/sanitize input first
if not is_valid_identifier(user_input):
    raise ValueError("Invalid input")
filter_string = f"name=eq.{sanitized_input}"
```

**Check:**
- Any use of f-strings with user input in database queries
- Raw SQL in any form
- Dynamic table names from user input

### 3. Authentication & Authorization (HIGH)

```python
# DANGEROUS - No auth check
@router.get("/users/{user_id}")
async def get_user(user_id: str):
    return await get_user_by_id(user_id)  # Anyone can access any user

# SAFE - Auth check
@router.get("/users/{user_id}")
async def get_user(user_id: str, current_user: User = Depends(get_current_user)):
    if current_user.id != user_id and not current_user.is_admin:
        raise HTTPException(403, "Not authorized")
    return await get_user_by_id(user_id)
```

**Check:**
- All endpoints have auth dependencies
- User can only access their own data (unless admin)
- Org-level data checks `org_id` matches user's org

### 4. XSS Prevention (MEDIUM) - Frontend

```tsx
// DANGEROUS - Renders raw HTML
<div dangerouslySetInnerHTML={{ __html: userContent }} />

// DANGEROUS - Unescaped in href
<a href={userInput}>Link</a>  // Could be javascript:alert('xss')

// SAFE - Use proper escaping or sanitization
import DOMPurify from 'dompurify'
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userContent) }} />
```

**Check:**
- Any use of `dangerouslySetInnerHTML`
- Dynamic `href` or `src` attributes
- User content rendered without escaping

### 5. Input Validation (MEDIUM)

```python
# DANGEROUS - No validation
async def create_user(data: dict):
    await crud_service.create_record("users", data)

# SAFE - Pydantic validation
class CreateUserRequest(BaseModel):
    email: EmailStr
    name: str = Field(min_length=1, max_length=100)
    
async def create_user(data: CreateUserRequest):
    await crud_service.create_record("users", data.model_dump())
```

**Check:**
- All API inputs use Pydantic models
- File uploads validate type and size
- Numeric inputs have reasonable bounds

### 6. Sensitive Data Exposure (MEDIUM)

```python
# DANGEROUS - Logs sensitive data
logger.info(f"User login: {user.email}, password: {password}")
logger.info(f"API response: {response}")  # May contain PII

# SAFE - Redact sensitive fields
logger.info(f"User login: {user.email}")
logger.info(f"API response: {redact_pii(response)}")
```

**Check:**
- No passwords/tokens in logs
- No PII (email, phone, address) in logs unless necessary
- Error responses don't leak stack traces to client

### 7. CORS & Headers (LOW)

```python
# DANGEROUS - Allow all origins
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Too permissive
)

# SAFE - Specific origins
app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://app.howthef.ai"],
)
```

### 8. Dependency Vulnerabilities (LOW)

Run security checks on dependencies:

```bash
# Python
pip-audit
safety check

# JavaScript
npm audit
```

---

## Audit Checklist

### Backend (Python/FastAPI)

- [ ] No hardcoded secrets (use env vars)
- [ ] All endpoints have auth dependencies
- [ ] User data access checks ownership
- [ ] Pydantic validation on all inputs
- [ ] No sensitive data in logs
- [ ] CORS configured for specific origins
- [ ] Rate limiting on public endpoints
- [ ] File uploads validated

### Frontend (Next.js/React)

- [ ] No API keys in client code
- [ ] No `dangerouslySetInnerHTML` with user content
- [ ] No dynamic `javascript:` URLs
- [ ] Auth state managed securely
- [ ] Sensitive routes protected
- [ ] No PII in localStorage (use httpOnly cookies)

### Database

- [ ] RLS policies enabled on Supabase tables
- [ ] No direct client queries (use API routes)
- [ ] Sensitive columns encrypted if needed
- [ ] Audit logs for sensitive operations

---

## Severity Levels

| Level | Description | Action |
|-------|-------------|--------|
| **CRITICAL** | Exploitable now, data breach risk | Fix immediately, block deploy |
| **HIGH** | Significant risk, needs attention | Fix before next deploy |
| **MEDIUM** | Should be fixed, not urgent | Fix within sprint |
| **LOW** | Best practice improvement | Fix when convenient |

---

## External Resources

For deeper security analysis, consider:

- **Trail of Bits Skills** - Advanced security analysis (available as Cursor Remote Rule from `trailofbits/skills`)
- **Semgrep** - Static analysis for security patterns
- **OWASP Top 10** - Common web vulnerabilities reference
