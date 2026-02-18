# 7.6 Security Best Practices

## Overview

This sub-lesson covers security best practices for insurance systems, including OWASP Top 10 mitigations and secure configuration guidelines.

---

## OWASP Top 10 Mitigations

The OWASP Top 10 represents the most critical web application security risks. Here are the mitigations for each:

| Vulnerability | Mitigation |
|--------------|------------|
| **Injection** | Parameterized queries, input validation |
| **Broken Authentication** | MFA, session management, password policies |
| **Sensitive Data Exposure** | Encryption, masking, access controls |
| **XXE** | Disable external entities, use JSON |
| **Broken Access Control** | RBAC, principle of least privilege |
| **Security Misconfiguration** | Hardening, security headers |
| **XSS** | Output encoding, CSP |
| **Insecure Deserialization** | Input validation, integrity checks |
| **Vulnerable Components** | Dependency scanning, updates |
| **Insufficient Logging** | Comprehensive audit logging |

---

## Injection Prevention

### SQL Injection

```python
# WRONG - Vulnerable to SQL injection
def get_policy_unsafe(policy_number: str):
    query = f"SELECT * FROM policies WHERE policy_number = '{policy_number}'"
    return db.execute(query)

# CORRECT - Parameterized query
def get_policy_safe(policy_number: str):
    query = "SELECT * FROM policies WHERE policy_number = %s"
    return db.execute(query, (policy_number,))

# CORRECT - Using ORM
def get_policy_orm(policy_number: str):
    return Policy.query.filter_by(policy_number=policy_number).first()
```

### Command Injection

```python
# WRONG - Vulnerable to command injection
def generate_report_unsafe(filename: str):
    os.system(f"generate_report.sh {filename}")

# CORRECT - Use subprocess with list arguments
def generate_report_safe(filename: str):
    if not re.match(r'^[\w\-\.]+$', filename):
        raise ValueError("Invalid filename")
    subprocess.run(["generate_report.sh", filename], check=True)
```

---

## Security Headers

Configure these HTTP security headers on all responses:

```
# Recommended HTTP Security Headers
Strict-Transport-Security: max-age=31536000; includeSubDomains
Content-Security-Policy: default-src 'self'; script-src 'self'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: geolocation=(), microphone=(), camera=()
```

### Header Explanations

| Header | Purpose |
|--------|---------|
| **Strict-Transport-Security** | Forces HTTPS connections |
| **Content-Security-Policy** | Prevents XSS by restricting resource sources |
| **X-Content-Type-Options** | Prevents MIME type sniffing |
| **X-Frame-Options** | Prevents clickjacking |
| **X-XSS-Protection** | Enables browser XSS filters |
| **Referrer-Policy** | Controls referrer information |
| **Permissions-Policy** | Restricts browser features |

### Implementation Example

```python
# FastAPI middleware
from starlette.middleware.base import BaseHTTPMiddleware

class SecurityHeadersMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        response = await call_next(request)

        response.headers["Strict-Transport-Security"] = "max-age=31536000; includeSubDomains"
        response.headers["Content-Security-Policy"] = "default-src 'self'"
        response.headers["X-Content-Type-Options"] = "nosniff"
        response.headers["X-Frame-Options"] = "DENY"
        response.headers["X-XSS-Protection"] = "1; mode=block"
        response.headers["Referrer-Policy"] = "strict-origin-when-cross-origin"
        response.headers["Permissions-Policy"] = "geolocation=(), microphone=(), camera=()"

        return response

app.add_middleware(SecurityHeadersMiddleware)
```

---

## Input Validation

### Validation Rules

```python
from pydantic import BaseModel, validator, Field
import re

class PolicyCreateRequest(BaseModel):
    """Request model with input validation"""

    policy_number: str = Field(..., regex=r'^POL-\d{10}$')
    customer_id: str = Field(..., regex=r'^CUS-[A-Z0-9]{8}$')
    premium_amount: float = Field(..., gt=0, le=10000000)
    coverage_limit: int = Field(..., gt=0, le=100000000)
    email: str = Field(..., regex=r'^[\w\.-]+@[\w\.-]+\.\w+$')
    phone: str = Field(..., regex=r'^\+?[\d\-\s]{10,15}$')

    @validator('email')
    def validate_email(cls, v):
        if not re.match(r'^[\w\.-]+@[\w\.-]+\.\w+$', v):
            raise ValueError('Invalid email format')
        return v.lower()

    @validator('policy_number')
    def validate_policy_number(cls, v):
        if not v.startswith('POL-'):
            raise ValueError('Policy number must start with POL-')
        return v.upper()
```

### Sanitization

```python
import bleach
import html

def sanitize_html(input_text: str) -> str:
    """Remove all HTML tags from input"""
    return bleach.clean(input_text, tags=[], strip=True)

def sanitize_for_display(input_text: str) -> str:
    """Escape HTML for safe display"""
    return html.escape(input_text)

def sanitize_filename(filename: str) -> str:
    """Sanitize filename for safe storage"""
    # Remove path separators and null bytes
    sanitized = re.sub(r'[/\\:\*\?"<>|\x00]', '_', filename)
    # Limit length
    return sanitized[:255]
```

---

## Session Management

### Secure Session Configuration

```yaml
session:
  # Session lifetime
  max_age: 3600  # 1 hour
  idle_timeout: 900  # 15 minutes

  # Cookie settings
  cookie_name: "__session"
  cookie_secure: true
  cookie_httponly: true
  cookie_samesite: "strict"

  # Session token
  token_length: 32
  token_algorithm: "sha256"

  # Concurrent sessions
  max_concurrent_sessions: 3
  revoke_on_password_change: true
```

### Session Security Practices

| Practice | Implementation |
|----------|---------------|
| **Regenerate session ID** | After login and privilege escalation |
| **Secure cookies** | HttpOnly, Secure, SameSite flags |
| **Session timeout** | Absolute and idle timeout |
| **Concurrent limit** | Limit active sessions per user |
| **Session revocation** | Invalidate on logout/password change |

---

## Password Policy

### Password Requirements

```yaml
password_policy:
  min_length: 12
  max_length: 128
  require_uppercase: true
  require_lowercase: true
  require_numbers: true
  require_special: true
  special_characters: "!@#$%^&*()_+-=[]{}|;:,.<>?"

  # History and expiry
  password_history: 12  # Cannot reuse last 12 passwords
  max_age_days: 90
  min_age_days: 1

  # Lockout
  max_failed_attempts: 5
  lockout_duration_minutes: 30

  # Breach checking
  check_pwned_passwords: true
  check_common_passwords: true
```

### Password Validation

```python
import re
from pwned_passwords import check_pwned

def validate_password(password: str) -> list[str]:
    """Validate password against policy"""
    errors = []

    if len(password) < 12:
        errors.append("Password must be at least 12 characters")

    if not re.search(r'[A-Z]', password):
        errors.append("Password must contain uppercase letter")

    if not re.search(r'[a-z]', password):
        errors.append("Password must contain lowercase letter")

    if not re.search(r'\d', password):
        errors.append("Password must contain number")

    if not re.search(r'[!@#$%^&*()_+\-=\[\]{}|;:,.<>?]', password):
        errors.append("Password must contain special character")

    if check_pwned(password):
        errors.append("Password found in data breach - choose another")

    return errors
```

---

## Security Checklist

### Development

- [ ] Input validation on all user inputs
- [ ] Parameterized queries for database access
- [ ] Output encoding for displayed data
- [ ] CSRF tokens on state-changing operations
- [ ] Secure session management
- [ ] Strong password hashing (bcrypt, Argon2)

### Deployment

- [ ] TLS 1.3 for all connections
- [ ] Security headers configured
- [ ] Debug mode disabled
- [ ] Default credentials changed
- [ ] Unnecessary services disabled
- [ ] Firewall rules configured

### Monitoring

- [ ] Audit logging enabled
- [ ] Failed login monitoring
- [ ] Anomaly detection
- [ ] Security alerting configured
- [ ] Log retention policy implemented

---

## Navigation

[Back to Lesson 07](../README.md) | [Previous: Audit Trail](../05-audit-trail/README.md)
