# OWASP Top 10 Security Standards

**Priority: P0 (Critical)** - Security vulnerabilities can lead to data breaches, financial loss, and legal liability.

## Overview

This document maps OWASP Top 10 (2021) vulnerabilities to practical prevention patterns for our TypeScript/Node.js stack.

## A01: Broken Access Control

### Red Flags
- Missing authorization checks on API endpoints
- Direct object references without ownership validation
- Relying solely on client-side access control

### Required Patterns

```typescript
// WRONG: No ownership check
app.get('/api/orders/:id', async (req, res) => {
  const order = await prisma.order.findUnique({ where: { id: req.params.id } });
  return res.json(order);
});

// CORRECT: Validate ownership
app.get('/api/orders/:id', requireAuth, async (req, res) => {
  const order = await prisma.order.findUnique({
    where: {
      id: req.params.id,
      userId: req.user.id  // Ownership check
    }
  });

  if (!order) {
    throw new NotFoundError('ORDER_NOT_FOUND', 'Order not found');
  }

  return res.json(order);
});
```

### Checklist
- [ ] Every endpoint has explicit authorization middleware
- [ ] Resource access validates ownership/permissions
- [ ] Admin functions are protected by role checks
- [ ] API endpoints deny by default

## A02: Cryptographic Failures

### Red Flags
- Storing passwords in plain text
- Using weak hashing algorithms (MD5, SHA1)
- Hardcoded secrets in source code
- Transmitting sensitive data without TLS

### Required Patterns

```typescript
// Password hashing - use bcrypt with sufficient rounds
import bcrypt from 'bcrypt';

const SALT_ROUNDS = 12;

async function hashPassword(password: string): Promise<string> {
  return bcrypt.hash(password, SALT_ROUNDS);
}

async function verifyPassword(password: string, hash: string): Promise<boolean> {
  return bcrypt.compare(password, hash);
}
```

### Checklist
- [ ] Passwords hashed with bcrypt (cost factor >= 12)
- [ ] Secrets stored in environment variables, never in code
- [ ] Sensitive data encrypted at rest
- [ ] All traffic uses HTTPS

## A03: Injection

### Red Flags
- String concatenation in queries
- Unparameterized database queries
- Executing user input as code
- Unescaped output in templates

### Required Patterns

```typescript
// WRONG: SQL injection vulnerable
const query = `SELECT * FROM users WHERE email = '${email}'`;

// CORRECT: Use Prisma (parameterized by default)
const user = await prisma.user.findUnique({
  where: { email }  // Automatically parameterized
});

// WRONG: Command injection
exec(`convert ${userFilename} output.png`);

// CORRECT: Use arrays for arguments
execFile('convert', [userFilename, 'output.png']);
```

### Checklist
- [ ] Always use Prisma or parameterized queries
- [ ] Never concatenate user input into queries
- [ ] Validate and sanitize all user input
- [ ] Use allowlists for dynamic values when possible

## A04: Insecure Design

### Required Patterns
- Implement rate limiting on all public endpoints
- Use CAPTCHA for sensitive operations
- Implement account lockout after failed attempts
- Design with least privilege principle

See: `api-security.md` for rate limiting patterns.

## A05: Security Misconfiguration

### Checklist
- [ ] Remove default credentials
- [ ] Disable unnecessary features and endpoints
- [ ] Keep dependencies updated (`npm audit`)
- [ ] Use security headers (see `api-security.md`)
- [ ] Disable detailed error messages in production

## A06: Vulnerable Components

### Required Practices

```bash
# Regular dependency audits
npm audit

# Update dependencies
npm update

# Check for outdated packages
npm outdated
```

### Checklist
- [ ] Run `npm audit` in CI pipeline
- [ ] Address high/critical vulnerabilities immediately
- [ ] Pin dependency versions in production
- [ ] Review changelogs before major updates

## A07: Authentication Failures

### Red Flags
- Weak password requirements
- Missing brute force protection
- Session tokens in URLs
- Improper session invalidation

### Required Patterns

```typescript
// Password requirements
const passwordSchema = z.string()
  .min(12, 'Password must be at least 12 characters')
  .regex(/[A-Z]/, 'Password must contain uppercase letter')
  .regex(/[a-z]/, 'Password must contain lowercase letter')
  .regex(/[0-9]/, 'Password must contain number')
  .regex(/[^A-Za-z0-9]/, 'Password must contain special character');

// Session configuration
const sessionConfig = {
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    secure: process.env.NODE_ENV === 'production',
    httpOnly: true,
    sameSite: 'strict',
    maxAge: 24 * 60 * 60 * 1000  // 24 hours
  }
};
```

See: `api-security.md` for full authentication patterns.

## A08: Software and Data Integrity Failures

### Checklist
- [ ] Verify package integrity with lockfiles
- [ ] Use Subresource Integrity (SRI) for CDN resources
- [ ] Sign commits and releases
- [ ] Validate data integrity on critical operations

## A09: Security Logging and Monitoring

### Required Patterns

```typescript
// Log security events
function logSecurityEvent(event: {
  type: 'AUTH_FAILURE' | 'ACCESS_DENIED' | 'SUSPICIOUS_ACTIVITY';
  userId?: string;
  ip: string;
  details: string;
}): void {
  logger.warn({
    category: 'security',
    ...event,
    timestamp: new Date().toISOString()
  });
}

// Usage
logSecurityEvent({
  type: 'AUTH_FAILURE',
  ip: req.ip,
  details: `Failed login attempt for email: ${email}`
});
```

### Checklist
- [ ] Log all authentication events
- [ ] Log authorization failures
- [ ] Include IP, user ID, timestamp in logs
- [ ] Never log sensitive data (passwords, tokens)
- [ ] Set up alerts for suspicious patterns

## A10: Server-Side Request Forgery (SSRF)

### Red Flags
- Fetching URLs provided by users
- Following redirects without validation
- Internal service URLs exposed

### Required Patterns

```typescript
// WRONG: SSRF vulnerable
const response = await fetch(userProvidedUrl);

// CORRECT: Validate URL against allowlist
const ALLOWED_HOSTS = ['api.trusted-service.com'];

function isAllowedUrl(url: string): boolean {
  try {
    const parsed = new URL(url);
    return ALLOWED_HOSTS.includes(parsed.hostname);
  } catch {
    return false;
  }
}

if (!isAllowedUrl(userProvidedUrl)) {
  throw new ValidationError('INVALID_URL', 'URL not allowed');
}
```

## Quick Reference

| Vulnerability | Primary Defense |
|--------------|-----------------|
| Broken Access Control | Authorization middleware + ownership checks |
| Cryptographic Failures | bcrypt + env secrets + TLS |
| Injection | Prisma/parameterized queries |
| Insecure Design | Rate limiting + least privilege |
| Security Misconfiguration | Security headers + dependency updates |
| Vulnerable Components | npm audit + dependency updates |
| Auth Failures | Strong passwords + session security |
| Integrity Failures | Lockfiles + SRI |
| Logging Failures | Security event logging |
| SSRF | URL allowlists |
