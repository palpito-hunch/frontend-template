# Frontend Security Standards

**Priority: P0 (Critical)** - Frontend vulnerabilities expose users to XSS, data theft, and session hijacking.

**Applies to: Frontend (Next.js/React)**

## XSS Prevention

### React Auto-Escaping

React automatically escapes values in JSX, which prevents most XSS:

```tsx
// SAFE: React escapes this automatically
const UserGreeting = ({ name }: { name: string }) => {
  return <h1>Hello, {name}!</h1>;  // Safe even if name contains <script>
};
```

### Dangerous Patterns to Avoid

```tsx
// DANGEROUS: dangerouslySetInnerHTML
// Only use when absolutely necessary AND with sanitized content
const RichContent = ({ html }: { html: string }) => {
  // WRONG: Unsanitized HTML
  return <div dangerouslySetInnerHTML={{ __html: html }} />;

  // CORRECT: Sanitized HTML
  const sanitized = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p'],
    ALLOWED_ATTR: ['href']
  });
  return <div dangerouslySetInnerHTML={{ __html: sanitized }} />;
};

// DANGEROUS: Dynamic href with javascript:
// WRONG
<a href={userProvidedUrl}>Click here</a>

// CORRECT: Validate URL protocol
const SafeLink = ({ url, children }: { url: string; children: React.ReactNode }) => {
  const isSafe = /^https?:\/\//.test(url);
  if (!isSafe) return null;
  return <a href={url} rel="noopener noreferrer">{children}</a>;
};
```

### External Links

```tsx
// Always add rel="noopener noreferrer" for external links
<a
  href="https://external-site.com"
  target="_blank"
  rel="noopener noreferrer"
>
  External Link
</a>

// Next.js Link for internal navigation (safe by default)
import Link from 'next/link';
<Link href="/dashboard">Dashboard</Link>
```

## Content Security Policy (CSP)

### Next.js Configuration

```typescript
// next.config.js
const securityHeaders = [
  {
    key: 'Content-Security-Policy',
    value: [
      "default-src 'self'",
      "script-src 'self' 'unsafe-eval' 'unsafe-inline'",  // Required for Next.js dev
      "style-src 'self' 'unsafe-inline'",
      "img-src 'self' data: https:",
      "font-src 'self'",
      "connect-src 'self' https://api.yoursite.com",
      "frame-ancestors 'none'",
      "base-uri 'self'",
      "form-action 'self'"
    ].join('; ')
  },
  {
    key: 'X-Frame-Options',
    value: 'DENY'
  },
  {
    key: 'X-Content-Type-Options',
    value: 'nosniff'
  },
  {
    key: 'Referrer-Policy',
    value: 'strict-origin-when-cross-origin'
  },
  {
    key: 'Permissions-Policy',
    value: 'camera=(), microphone=(), geolocation=()'
  }
];

module.exports = {
  async headers() {
    return [
      {
        source: '/:path*',
        headers: securityHeaders
      }
    ];
  }
};
```

## Secure Storage

### Sensitive Data

```typescript
// WRONG: Storing sensitive data in localStorage
localStorage.setItem('authToken', token);  // Vulnerable to XSS

// CORRECT: Use httpOnly cookies (set by backend)
// Tokens should be stored in httpOnly cookies, not accessible to JS

// For non-sensitive preferences, localStorage is acceptable
localStorage.setItem('theme', 'dark');
localStorage.setItem('language', 'en');
```

### Session Storage Guidelines

| Data Type | Storage | Reason |
|-----------|---------|--------|
| Auth tokens | httpOnly cookie | XSS protection |
| Refresh tokens | httpOnly cookie | XSS protection |
| User preferences | localStorage | Non-sensitive |
| Form drafts | sessionStorage | Temporary, tab-scoped |
| Sensitive PII | Never client-side | Security |

## Authentication UI

### Secure Login Forms

```tsx
const LoginForm = () => {
  const [error, setError] = useState<string | null>(null);

  const handleSubmit = async (e: FormEvent) => {
    e.preventDefault();
    const formData = new FormData(e.target as HTMLFormElement);

    try {
      await login({
        email: formData.get('email') as string,
        password: formData.get('password') as string
      });
    } catch (err) {
      // Generic error message - don't reveal if email exists
      setError('Invalid email or password');
    }
  };

  return (
    <form onSubmit={handleSubmit} autoComplete="on">
      <input
        type="email"
        name="email"
        autoComplete="email"
        required
      />
      <input
        type="password"
        name="password"
        autoComplete="current-password"
        required
        minLength={12}
      />
      {error && <p role="alert">{error}</p>}
      <button type="submit">Log In</button>
    </form>
  );
};
```

### Password Requirements UI

```tsx
const PasswordInput = () => {
  const [password, setPassword] = useState('');

  const requirements = [
    { test: (p: string) => p.length >= 12, label: 'At least 12 characters' },
    { test: (p: string) => /[A-Z]/.test(p), label: 'Uppercase letter' },
    { test: (p: string) => /[a-z]/.test(p), label: 'Lowercase letter' },
    { test: (p: string) => /[0-9]/.test(p), label: 'Number' },
    { test: (p: string) => /[^A-Za-z0-9]/.test(p), label: 'Special character' }
  ];

  return (
    <div>
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        autoComplete="new-password"
      />
      <ul>
        {requirements.map((req, i) => (
          <li key={i} style={{ color: req.test(password) ? 'green' : 'red' }}>
            {req.label}
          </li>
        ))}
      </ul>
    </div>
  );
};
```

## API Communication

### Secure Fetch Wrapper

```typescript
const API_BASE = process.env.NEXT_PUBLIC_API_URL;

async function apiFetch<T>(
  endpoint: string,
  options: RequestInit = {}
): Promise<T> {
  const response = await fetch(`${API_BASE}${endpoint}`, {
    ...options,
    credentials: 'include',  // Include cookies
    headers: {
      'Content-Type': 'application/json',
      ...options.headers
    }
  });

  if (!response.ok) {
    const error = await response.json().catch(() => ({}));
    throw new ApiError(response.status, error.message || 'Request failed');
  }

  return response.json();
}
```

### CSRF Protection

```typescript
// Get CSRF token from cookie or meta tag
function getCsrfToken(): string {
  return document.querySelector('meta[name="csrf-token"]')?.getAttribute('content') || '';
}

// Include in requests that modify data
async function apiPost<T>(endpoint: string, data: unknown): Promise<T> {
  return apiFetch(endpoint, {
    method: 'POST',
    headers: {
      'X-CSRF-Token': getCsrfToken()
    },
    body: JSON.stringify(data)
  });
}
```

## Third-Party Scripts

### Safe Script Loading

```tsx
// next/script with proper strategy
import Script from 'next/script';

// Analytics - load after page is interactive
<Script
  src="https://analytics.example.com/script.js"
  strategy="afterInteractive"
/>

// Non-critical - load when browser is idle
<Script
  src="https://widget.example.com/embed.js"
  strategy="lazyOnload"
/>
```

### Subresource Integrity (SRI)

```html
<!-- For CDN scripts, use integrity attribute -->
<script
  src="https://cdn.example.com/lib.js"
  integrity="sha384-abc123..."
  crossorigin="anonymous"
></script>
```

## Environment Variables

### Client-Side Variables

```typescript
// Only NEXT_PUBLIC_ variables are exposed to browser
// next.config.js
module.exports = {
  env: {
    // EXPOSED to browser - only non-sensitive values
    NEXT_PUBLIC_API_URL: process.env.NEXT_PUBLIC_API_URL,
    NEXT_PUBLIC_APP_NAME: 'MyApp',

    // NEVER expose secrets
    // DATABASE_URL: process.env.DATABASE_URL  // DON'T DO THIS
  }
};
```

### Variable Checklist
- [ ] Secrets never prefixed with `NEXT_PUBLIC_`
- [ ] API keys for browser use are restricted (domain, rate limited)
- [ ] Sensitive operations happen server-side only

## Checklist

### Every Component
- [ ] No `dangerouslySetInnerHTML` without sanitization
- [ ] External links have `rel="noopener noreferrer"`
- [ ] User URLs validated before use in `href`
- [ ] No sensitive data in error messages

### Forms
- [ ] Proper `autocomplete` attributes
- [ ] Generic error messages (don't reveal existence)
- [ ] CSRF token included for mutations
- [ ] Client-side validation matches server

### Storage
- [ ] Auth tokens in httpOnly cookies only
- [ ] No sensitive data in localStorage/sessionStorage
- [ ] Clear storage on logout

### Third-Party
- [ ] Scripts loaded with appropriate strategy
- [ ] SRI for CDN resources
- [ ] CSP allows only required sources
