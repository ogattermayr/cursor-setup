---
name: sharp-edges
description: "Identifies error-prone APIs, dangerous configurations, and footgun designs. Use when reviewing API designs, configuration schemas, or evaluating code for 'secure by default' principles. From Trail of Bits."
---

# Sharp Edges Analysis

Evaluates whether APIs, configurations, and interfaces are resistant to developer misuse. Identifies designs where the "easy path" leads to insecurity.

## When to Use

- Reviewing API or library design decisions
- Auditing configuration schemas for dangerous options
- Assessing authentication/authorization interfaces
- Reviewing any code that exposes security-relevant choices

## Core Principle

**The pit of success**: Secure usage should be the path of least resistance. If developers must read documentation carefully or remember special rules to avoid vulnerabilities, the API has failed.

## Rationalizations to Reject

| Rationalization | Why It's Wrong |
|-----------------|----------------|
| "It's documented" | Developers don't read docs under deadline pressure |
| "Advanced users need flexibility" | Flexibility creates footguns; most usage is copy-paste |
| "It's the developer's responsibility" | You designed the footgun |
| "Nobody would actually do that" | Developers do everything imaginable under pressure |

## Sharp Edge Categories

### 1. Dangerous Defaults

Defaults that are insecure, or zero/empty values that disable security.

```python
# What happens when lifetime=0?
def verify_otp(code, lifetime=300):
    if lifetime == 0:
        return True  # OOPS: 0 means "accept all"?
```

**Questions to ask:**
- What happens with `timeout=0`? `max_attempts=0`? `key=""`?
- Is the default the most secure option?
- Can any default value disable security entirely?

### 2. Silent Failures

Errors that don't surface, or success that masks failure.

```python
# Silent bypass
def verify_signature(sig, data, key):
    if not key:
        return True  # No key = skip verification?!
```

### 3. Configuration Cliffs

One wrong setting creates catastrophic failure.

```yaml
# One typo = disaster
verify_ssl: fasle  # Typo silently accepted?

# Dangerous combinations accepted silently
auth_required: true
bypass_auth_for_health_checks: true
health_check_path: "/"  # Oops
```

## Python Sharp Edges

### Mutable Default Arguments
```python
# DANGEROUS
def append_to(item, target=[]):
    target.append(item)
    return target

# FIX
def append_to(item, target=None):
    if target is None:
        target = []
```

### Code Execution
```python
# DANGEROUS
eval(user_input)
exec(user_input)
pickle.loads(user_data)
yaml.load(user_data)  # Use yaml.safe_load()
subprocess.run(f"cmd {user_input}", shell=True)  # Use list form
```

### Exception Swallowing
```python
# DANGEROUS
try:
    important_security_check()
except:
    pass  # Security check failure ignored!
```

### Detection Patterns

| Pattern | Risk |
|---------|------|
| `def f(x=[])` or `def f(x={})` | Mutable default argument |
| `eval(`, `exec(`, `compile(` | Code execution |
| `pickle.loads(`, `yaml.load(` | Deserialization RCE |
| `except:` or `except Exception:` | Over-broad exception |
| `subprocess.*(..., shell=True)` | Command injection |

## JavaScript/TypeScript Sharp Edges

### Prototype Pollution
```javascript
// DANGEROUS: Merging untrusted objects
function merge(target, source) {
    for (let key in source) {
        target[key] = source[key];  // Includes __proto__!
    }
}

// Attacker sends: {"__proto__": {"isAdmin": true}}
merge({}, JSON.parse(userInput));
// Now ALL objects have isAdmin = true
```

### Loose Equality
```javascript
// DANGEROUS
if (userRole == "admin") {  // Use === instead
    grantAdmin();
}
0 == ""  // true!
```

### ReDoS (Regular Expression DoS)
```javascript
// DANGEROUS: Catastrophic backtracking
const regex = /^(a+)+$/;
regex.test("aaaaaaaaaaaaaaaaaaaaaaaaaaaa!");
// Exponential time - freezes event loop
```

### TypeScript Assertions
```typescript
// DANGEROUS: No runtime check!
const user = userData as Admin;
user.adminMethod();  // Runtime error if not Admin
```

### Detection Patterns

| Pattern | Risk |
|---------|------|
| `==` instead of `===` | Type coercion bugs |
| `obj[userInput]` | Prototype pollution |
| `(a+)+`, `(.*)+` in regex | ReDoS |
| `eval(`, `Function(`, `setTimeout(string` | Code execution |
| `as Type` assertions | Runtime type mismatch |
| Missing `await` before async | Race condition |

## Analysis Workflow

### Phase 1: Surface Identification
1. Map security-relevant APIs: auth, crypto, session, validation
2. Identify developer choice points: algorithms, timeouts, modes
3. Find configuration schemas: env vars, config files, constructors

### Phase 2: Edge Case Probing
For each choice point, ask:
- **Zero/empty/null**: What happens with `0`, `""`, `null`?
- **Type confusion**: Can different security concepts be swapped?
- **Error paths**: What happens on invalid input? Silent acceptance?

### Phase 3: Threat Modeling
Consider three adversaries:
1. **The Scoundrel**: Malicious developer controlling config
2. **The Lazy Developer**: Copy-pastes examples, skips docs
3. **The Confused Developer**: Misunderstands the API

## Severity Classification

| Severity | Criteria |
|----------|----------|
| Critical | Default or obvious usage is insecure |
| High | Easy misconfiguration breaks security |
| Medium | Unusual but possible misconfiguration |
| Low | Requires deliberate misuse |

---

*Based on Trail of Bits sharp-edges skill*
