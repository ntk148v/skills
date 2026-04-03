---
name: security-expert
description: Use when analyzing security vulnerabilities, conducting security reviews, checking for common vulnerabilities, or when the user mentions "security", "vulnerability", "CVE", "OWASP", or "penetration test".
---

# Security Expert

## Overview

Security Expert provides a systematic approach to identifying and preventing security vulnerabilities in code. It covers OWASP Top 10, common vulnerability patterns, secure coding practices, and remediation strategies for building secure applications.

## When to Use

Use this skill when:

- Conducting security reviews
- Analyzing code for vulnerabilities
- The user mentions "security", "vulnerability", or "secure coding"
- Checking for OWASP Top 10 issues
- Reviewing authentication and authorization
- Validating input handling
- Investigating potential CVEs

## Security Review Process

### Phase 1: Threat Modeling

Before writing code, understand potential threats:

1. **Identify Assets**
   - User data (PII, credentials)
   - Financial information
   - Intellectual property
   - System access

2. **Map Attack Surface**
   - User inputs (forms, APIs, URLs)
   - External services
   - File uploads
   - Configuration files

3. **Identify Attack Vectors**
   - SQL injection
   - Cross-site scripting (XSS)
   - Cross-site request forgery (CSRF)
   - Authentication bypass
   - Authorization flaws

### Phase 2: Input Validation

All user input is evil until proven otherwise.

#### SQL Injection

**The Problem:**

```sql
-- User input directly concatenated
SELECT * FROM users WHERE id = ${userId}

-- Attacker input: 1; DROP TABLE users;--
```

**The Solution - Parameterized Queries:**

```javascript
// ❌ BAD: String concatenation
const query = `SELECT * FROM users WHERE id = ${userId}`;

// ✅ GOOD: Parameterized query
const query = "SELECT * FROM users WHERE id = ?";
db.query(query, [userId]);
```

```python
# ❌ BAD: String formatting
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")

# ✅ GOOD: Parameterized query
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
```

#### Command Injection

```javascript
// ❌ BAD: User input in shell command
const cmd = `ls ${userDirectory}`;
exec(cmd);

// ✅ GOOD: Avoid shell when possible, or use safe alternatives
const files = fs.readdirSync(userDirectory);
```

```python
# ❌ BAD: shell=True with user input
subprocess.run(f"ls {directory}", shell=True)

# ✅ GOOD: list form without shell
subprocess.run(["ls", directory])
```

#### Path Traversal

```javascript
// ❌ BAD: User input in file path
const file = path.join uploadsDir, req.body.filename;
fs.readFile(file);

// ✅ GOOD: Validate and sanitize
const filename = req.body.filename;
if (!isValidFilename(filename)) {
  throw new Error('Invalid filename');
}
const file = path.join(uploadsDir, filename);
if (!file.startsWith(uploadsDir)) {
  throw new Error('Path traversal detected');
}
```

### Phase 3: Authentication & Authorization

#### Password Security

```javascript
// ❌ BAD: Plain text or weak hashing
const hash = crypto.createHash("md5").update(password).digest("hex");

// ✅ GOOD: Strong hashing with bcrypt/argon2
const hash = await bcrypt.hash(password, 12);
const match = await bcrypt.compare(password, hash);
```

```python
# ❌ BAD: md5 or sha1
import hashlib
hash = hashlib.md5(password.encode()).hexdigest()

# ✅ GOOD: bcrypt or argon2
from passlib.context import CryptContext
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
hash = pwd_context.hash(password)
```

#### Session Management

- Use secure, random session IDs
- Implement proper session timeout
- Regenerate session ID after authentication
- Secure session cookies (HttpOnly, Secure, SameSite)

```javascript
// ❌ BAD: Weak session ID
const sessionId = user.id; // Using user ID as session!

// ✅ GOOD: Cryptographically random session
const sessionId = crypto.randomBytes(32).toString("hex");
```

#### Multi-Factor Authentication

- Support TOTP (Time-based One-Time Password)
- Support hardware keys (WebAuthn/FIDO2)
- Implement backup codes securely
- Rate limit authentication attempts

### Phase 4: Authorization

#### Access Control

```javascript
// ❌ BAD: Client-side only authorization
if (user.role === "admin") {
  showAdminPanel();
}

// ✅ GOOD: Server-side authorization check
async function adminHandler(req, res) {
  const user = await getUser(req.session.userId);
  if (user.role !== "admin") {
    return res.status(403).json({ error: "Forbidden" });
  }
  // Proceed with admin operation
}
```

#### IDOR (Insecure Direct Object Reference)

```javascript
// ❌ BAD: No authorization check
app.get("/orders/:orderId", async (req, res) => {
  const order = await db.orders.findById(req.params.orderId);
  res.json(order);
});

// ✅ GOOD: Verify ownership
app.get("/orders/:orderId", async (req, res) => {
  const order = await db.orders.findById(req.params.orderId);
  if (order.userId !== req.session.userId) {
    return res.status(403).json({ error: "Forbidden" });
  }
  res.json(order);
});
```

### Phase 5: Data Protection

#### Sensitive Data Exposure

```javascript
// ❌ BAD: Logging sensitive data
logger.info(`User login: ${email} password: ${password}`);

// ✅ GOOD: Never log sensitive data
logger.info(`User login attempt for: ${email}`);
```

```python
# ❌ BAD: Returning all fields including sensitive
def get_user(user_id):
    return db.users.find_one(id=user_id)

# ✅ GOOD: Select only needed fields
def get_user(user_id):
    return db.users.find_one(
        id=user_id,
        projection={'password_hash': 0, 'ssn': 0}
    )
```

#### Encryption

```javascript
// ❌ BAD: No encryption for sensitive data
const userData = { ssn: "123-45-6789" };
localStorage.setItem("user", JSON.stringify(userData));

// ✅ GOOD: Encrypt sensitive data
const encrypted = crypto
  .createCipher("aes-256-gcm")
  .update(JSON.stringify(userData), "utf8", "hex");
localStorage.setItem("user", encrypted);
```

#### Secrets Management

```javascript
// ❌ BAD: Hardcoded secrets
const API_KEY = "sk_live_abc123";
const DB_PASSWORD = "secret123";

// ✅ GOOD: Environment variables or secret manager
const API_KEY = process.env.API_KEY;
const DB_PASSWORD = await secretManager.get("db_password");
```

### Phase 6: Output Encoding

#### XSS (Cross-Site Scripting)

```html
<!-- ❌ BAD: Unescaped user input -->
<div>{{ userBio }}</div>
<!-- If userBio = "<script>stealCookies()</script>" -->

<!-- ✅ GOOD: Context-aware escaping -->
<div>{{ userBio | escape }}</div>
```

```javascript
// ❌ BAD: InnerHTML with user input
element.innerHTML = userComment;

// ✅ GOOD: Text content or sanitization
element.textContent = userComment;
// Or sanitize with DOMPurify
element.innerHTML = DOMPurify.sanitize(userComment);
```

#### Content Security Policy

```html
<!-- ✅ GOOD: Strict CSP header -->
<meta
  http-equiv="Content-Security-Policy"
  content="default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'"
/>
```

### Phase 7: API Security

#### Rate Limiting

```javascript
// ❌ BAD: No rate limiting
app.post("/login", async (req, res) => {
  const user = await authenticate(req.body);
  res.json(user);
});

// ✅ GOOD: Rate limiting
const rateLimit = require("express-rate-limit");
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 attempts per window
  message: "Too many attempts, please try again later",
});
app.post("/login", limiter, async (req, res) => {
  const user = await authenticate(req.body);
  res.json(user);
});
```

#### CORS Configuration

```javascript
// ❌ BAD: Permissive CORS
app.use(cors({ origin: "*" }));

// ✅ GOOD: Restrictive CORS
app.use(
  cors({
    origin: ["https://trusted-domain.com"],
    credentials: true,
  }),
);
```

#### API Authentication

```javascript
// ❌ BAD: Basic auth in URL
// GET /api/data?api_key=sk_live_abc123

// ✅ GOOD: Bearer token in header
// GET /api/data
// Authorization: Bearer sk_live_abc123
```

### Phase 8: Dependency Security

#### Vulnerability Scanning

```bash
# npm audit
npm audit

# yarn audit
yarn audit

# Python safety
pip audit

# Docker scan
docker scan image:latest
```

#### Keeping Dependencies Updated

```bash
# Check outdated packages
npm outdated

# Update specific package
npm update lodash@latest

# Update all packages
npm update
```

## OWASP Top 10 (2021)

### A01: Broken Access Control

- Implement proper authorization at all layers
- Deny by default
- Log access control failures
- Rate limit API access

### A02: Cryptographic Failures

- Classify data processed by application
- Encrypt data in transit and at rest
- Use strong cryptographic algorithms
- Don't cache sensitive data

### A03: Injection

- Use parameterized queries
- Validate and sanitize input
- Escape special characters
- Use ORM when possible

### A04: Insecure Design

- Threat model during development
- Use secure design patterns
- Segregate tenant resources
- Limit resource consumption

### A05: Security Misconfiguration

- Harden cloud services
- Remove unnecessary features
- Review configurations regularly
- Automated security checks

### A06: Vulnerable Components

- Remove unused dependencies
- Monitor for CVE announcements
- Use component analysis tools
- Only obtain from official sources

### A07: Authentication Failures

- Implement multi-factor authentication
- Don't deploy with default credentials
- Use secure session management
- Limit failed login attempts

### A08: Software and Data Integrity

- Verify integrity of components
- Don't trust unsigned code
- Review CI/CD pipeline
- Verify dependencies

### A09: Security Logging Failures

- Log all authentication events
- Log access control failures
- Use centralized logging
- Ensure log integrity

### A10: Server-Side Request Forgery (SSRF)

- Validate all user-supplied URLs
- Use allowlists for URLs
- Disable HTTP redirections
- Sanitize forwarded URLs

## Security Checklist

### Authentication

- [ ] Strong password requirements (min 8 chars, mixed case, numbers, symbols)
- [ ] Secure password hashing (bcrypt/argon2 with cost factor)
- [ ] Multi-factor authentication available
- [ ] Account lockout after failed attempts
- [ ] Session timeout and regeneration
- [ ] Secure session tokens (HttpOnly, Secure, SameSite)

### Authorization

- [ ] Server-side authorization checks
- [ ] Principle of least privilege
- [ ] Resource ownership verification
- [ ] Role-based access control (RBAC)
- [ ] Audit logging of permission changes

### Input Validation

- [ ] Validate all user input
- [ ] Use parameterized queries
- [ ] Sanitize file uploads
- [ ] Validate file types and sizes
- [ ] Sanitize output for display

### Data Protection

- [ ] Encrypt sensitive data at rest
- [ ] Use TLS for data in transit
- [ ] No secrets in code or version control
- [ ] Secure secrets management
- [ ] Data minimization in logging

### API Security

- [ ] Rate limiting
- [ ] Proper CORS configuration
- [ ] Authentication for all endpoints
- [ ] Input validation on API
- [ ] Output encoding

### Dependencies

- [ ] Regular vulnerability scanning
- [ ] Keep dependencies updated
- [ ] Remove unused dependencies
- [ ] Use official sources only
- [ ] Lock file integrity

## Common Vulnerabilities

| Vulnerability     | Attack                   | Impact               | Fix                   |
| ----------------- | ------------------------ | -------------------- | --------------------- |
| SQL Injection     | Manipulate queries       | Data breach          | Parameterized queries |
| XSS               | Inject scripts           | Session hijacking    | Escape output         |
| CSRF              | Forge requests           | Unauthorized actions | CSRF tokens           |
| IDOR              | Access others' resources | Data exposure        | Authorization checks  |
| SSRF              | Abuse server resources   | Internal access      | URL validation        |
| Path Traversal    | Access system files      | System compromise    | Path sanitization     |
| Command Injection | Execute commands         | System compromise    | Avoid shell, sanitize |

## Secure Development Lifecycle

1. **Design**: Threat model, secure architecture
2. **Develop**: Secure coding practices, code review
3. **Test**: Security testing, vulnerability scanning
4. **Deploy**: Secure configuration, hardening
5. **Monitor**: Logging, alerting, incident response

## Quick Reference

| Category     | Key Actions                                |
| ------------ | ------------------------------------------ |
| Input        | Validate, sanitize, parameterized queries  |
| Auth         | Strong hashing, MFA, session security      |
| Access       | Server-side checks, least privilege        |
| Data         | Encrypt, minimize exposure, secure storage |
| APIs         | Rate limit, CORS, authentication           |
| Dependencies | Scan, update, remove unused                |

## The Bottom Line

Security must be built in from the start, not bolted on at the end. Always validate input, encode output, use proper authentication and authorization, protect sensitive data, and keep dependencies updated. When in doubt, err on the side of caution.
