---
name: semgrep
description: "Run Semgrep static analysis for fast security scanning. Use when scanning code for vulnerabilities, writing security rules, or setting up CI/CD security checks. From Trail of Bits."
---

# Semgrep Static Analysis

Fast pattern-based security scanning.

## When to Use

- Quick security scans (minutes, not hours)
- Pattern-based bug detection
- Enforcing coding standards
- Finding known vulnerability patterns
- First-pass analysis before deeper review

## Installation

```bash
# pip
pip install semgrep

# Homebrew
brew install semgrep

# Docker
docker run --rm -v "${PWD}:/src" returntocorp/semgrep semgrep --config auto /src
```

## Quick Commands

```bash
# Auto-detect rules
semgrep --config auto .

# Use specific rulesets
semgrep --config p/security-audit .
semgrep --config p/python .
semgrep --config p/javascript .
semgrep --config p/trailofbits .

# Multiple rulesets
semgrep --config p/security-audit --config p/owasp-top-ten .

# Output formats
semgrep --config auto --sarif -o results.sarif .
semgrep --config auto --json -o results.json .

# Show data flow traces
semgrep --config auto --dataflow-traces .
```

## Useful Rulesets

| Ruleset | Description |
|---------|-------------|
| `p/default` | General security and code quality |
| `p/security-audit` | Comprehensive security rules |
| `p/owasp-top-ten` | OWASP Top 10 vulnerabilities |
| `p/python` | Python-specific |
| `p/javascript` | JavaScript-specific |
| `p/trailofbits` | Trail of Bits security rules |

## For HowTheF* Codebase

### Backend (Python/FastAPI)

```bash
# Scan backend
semgrep --config p/python --config p/security-audit howthef-backend/

# Key patterns to catch:
# - SQL injection via string formatting
# - Command injection
# - Insecure deserialization
# - Hardcoded secrets
```

### Frontend (Next.js/TypeScript)

```bash
# Scan frontend
semgrep --config p/javascript --config p/typescript howthef-frontend/

# Key patterns to catch:
# - XSS via dangerouslySetInnerHTML
# - Prototype pollution
# - Insecure randomness
# - Missing input validation
```

## Writing Custom Rules

### Basic Rule Structure

```yaml
rules:
  - id: hardcoded-api-key
    languages: [python]
    message: "Hardcoded API key detected"
    severity: ERROR
    pattern: api_key = "..."
```

### Pattern Syntax

| Syntax | Description |
|--------|-------------|
| `...` | Match anything |
| `$VAR` | Capture metavariable |
| `<... ...>` | Deep expression match |

### Taint Mode (Data Flow)

Track user input through the codebase:

```yaml
rules:
  - id: sql-injection
    languages: [python]
    message: "User input flows to SQL query"
    severity: ERROR
    mode: taint
    pattern-sources:
      - pattern: request.args.get(...)
      - pattern: request.form[...]
      - pattern: request.json
    pattern-sinks:
      - pattern: cursor.execute($QUERY)
      - pattern: db.execute($QUERY)
    pattern-sanitizers:
      - pattern: int(...)
```

### HowTheF*-Specific Rules

```yaml
rules:
  # Catch direct Supabase usage (should use crud_service)
  - id: htf-direct-supabase
    languages: [python]
    message: "Use crud_service instead of direct Supabase queries"
    severity: WARNING
    pattern-either:
      - pattern: supabase.table(...).select(...)
      - pattern: supabase.table(...).insert(...)
      - pattern: supabase.table(...).update(...)
    paths:
      exclude:
        - shared/services/crud_service.py

  # Catch missing auth in endpoints
  - id: htf-unprotected-endpoint
    languages: [python]
    message: "Endpoint may be missing authentication"
    severity: WARNING
    pattern: |
      async def $FUNC(...):
          ...
    pattern-not-inside: |
      async def $FUNC(..., current_user: ... = Depends(...), ...):
          ...
```

## Suppress False Positives

```python
# In code
password = get_from_vault()  # nosemgrep: hardcoded-password

# Or ignore specific rule
dangerous_but_intentional()  # nosemgrep
```

## .semgrepignore

```
tests/fixtures/
**/testdata/
generated/
node_modules/
.venv/
__pycache__/
```

## CI Integration (GitHub Actions)

```yaml
name: Semgrep
on: [push, pull_request]

jobs:
  semgrep:
    runs-on: ubuntu-latest
    container:
      image: returntocorp/semgrep
    steps:
      - uses: actions/checkout@v4
      - run: semgrep --config p/security-audit --config p/trailofbits --error .
```

## Rationalizations to Reject

| Shortcut | Why It's Wrong |
|----------|----------------|
| "Semgrep found nothing" | Pattern-based; misses complex flows |
| "Too many findings = noisy" | High count often = real problems |
| "We have tests" | Tests don't catch security issues |

---

*Based on Trail of Bits semgrep skill*
