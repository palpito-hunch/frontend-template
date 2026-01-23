# Input Validation & Sanitization Standards

**Priority: P0 (Critical)** - Unvalidated input is the root cause of most security vulnerabilities.

## Core Principle

**Never trust user input.** All data from external sources must be validated and sanitized before use.

## Validation vs Sanitization

| Concept | Purpose | Example |
|---------|---------|---------|
| **Validation** | Reject invalid input | Reject email without @ |
| **Sanitization** | Transform input to safe form | Escape HTML entities |

**Always validate first, then sanitize if needed.**

## Required Validation with Zod

### String Validation

```typescript
import { z } from 'zod';

// Email
const emailSchema = z.string()
  .email('Invalid email format')
  .max(254, 'Email too long')
  .toLowerCase()
  .trim();

// Username (alphanumeric only)
const usernameSchema = z.string()
  .min(3, 'Username must be at least 3 characters')
  .max(30, 'Username must be at most 30 characters')
  .regex(/^[a-zA-Z0-9_]+$/, 'Username can only contain letters, numbers, and underscores');

// URL
const urlSchema = z.string()
  .url('Invalid URL')
  .refine(url => {
    const parsed = new URL(url);
    return ['http:', 'https:'].includes(parsed.protocol);
  }, 'Only HTTP(S) URLs allowed');

// Free text with length limits
const descriptionSchema = z.string()
  .min(1, 'Description required')
  .max(1000, 'Description too long')
  .trim();
```

### Number Validation

```typescript
// Positive integer
const idSchema = z.number()
  .int('Must be an integer')
  .positive('Must be positive');

// Price (decimal with precision)
const priceSchema = z.number()
  .positive('Price must be positive')
  .multipleOf(0.01, 'Price can have at most 2 decimal places')
  .max(1000000, 'Price exceeds maximum');

// Pagination
const pageSchema = z.number()
  .int()
  .min(1)
  .default(1);

const limitSchema = z.number()
  .int()
  .min(1)
  .max(100)
  .default(20);
```

### Array Validation

```typescript
// Array with size limits
const tagsSchema = z.array(z.string().min(1).max(50))
  .min(1, 'At least one tag required')
  .max(10, 'Maximum 10 tags allowed');

// Unique values
const idsSchema = z.array(z.string().uuid())
  .refine(ids => new Set(ids).size === ids.length, 'Duplicate IDs not allowed');
```

## HTML Sanitization

### When to Sanitize

Sanitize HTML only when you **must** accept rich text content. Prefer plain text when possible.

```typescript
import DOMPurify from 'isomorphic-dompurify';

// Configure allowed tags
const sanitizeHtml = (dirty: string): string => {
  return DOMPurify.sanitize(dirty, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p', 'br', 'ul', 'ol', 'li'],
    ALLOWED_ATTR: ['href', 'target', 'rel'],
    ALLOW_DATA_ATTR: false
  });
};

// Always sanitize before storing AND before rendering
const userContent = sanitizeHtml(req.body.content);
```

### XSS Prevention

```typescript
// WRONG: Direct interpolation
const html = `<div>${userInput}</div>`;

// CORRECT: Use framework escaping (React auto-escapes)
return <div>{userInput}</div>;

// CORRECT: Manual escaping when needed
import { escape } from 'lodash';
const safeText = escape(userInput);
```

## File Upload Validation

```typescript
import { z } from 'zod';
import path from 'path';

const ALLOWED_EXTENSIONS = ['.jpg', '.jpeg', '.png', '.gif', '.webp'];
const MAX_FILE_SIZE = 5 * 1024 * 1024;  // 5MB

const fileSchema = z.object({
  originalname: z.string().refine(name => {
    const ext = path.extname(name).toLowerCase();
    return ALLOWED_EXTENSIONS.includes(ext);
  }, 'File type not allowed'),
  size: z.number().max(MAX_FILE_SIZE, 'File too large'),
  mimetype: z.string().refine(mime =>
    mime.startsWith('image/'), 'Only images allowed'
  )
});

// Additional server-side checks
async function validateFile(file: Express.Multer.File): Promise<void> {
  // Check magic bytes (file signature)
  const fileType = await import('file-type');
  const type = await fileType.fromBuffer(file.buffer);

  if (!type || !ALLOWED_EXTENSIONS.includes(`.${type.ext}`)) {
    throw new ValidationError('INVALID_FILE_TYPE', 'File type not allowed');
  }
}
```

## SQL/NoSQL Injection Prevention

### Always Use Parameterized Queries

```typescript
// WRONG: String concatenation
const query = `SELECT * FROM users WHERE id = '${userId}'`;

// CORRECT: Prisma (always parameterized)
const user = await prisma.user.findUnique({
  where: { id: userId }
});

// CORRECT: Raw query with parameters
const users = await prisma.$queryRaw`
  SELECT * FROM users WHERE email = ${email}
`;
```

### Dynamic Column/Table Names

```typescript
// WRONG: Direct interpolation
const column = req.query.sortBy;
const query = `SELECT * FROM users ORDER BY ${column}`;

// CORRECT: Allowlist validation
const ALLOWED_SORT_COLUMNS = ['name', 'email', 'createdAt'] as const;
type SortColumn = typeof ALLOWED_SORT_COLUMNS[number];

const sortBy = z.enum(ALLOWED_SORT_COLUMNS).parse(req.query.sortBy);
```

## Command Injection Prevention

```typescript
import { execFile } from 'child_process';

// WRONG: Shell command with user input
exec(`convert ${filename} output.png`);

// CORRECT: Use execFile with arguments array
execFile('convert', [filename, 'output.png'], (error, stdout) => {
  // Handle result
});

// CORRECT: Validate filename
const filenameSchema = z.string()
  .regex(/^[a-zA-Z0-9_-]+\.[a-zA-Z]+$/, 'Invalid filename');
```

## Path Traversal Prevention

```typescript
import path from 'path';

const UPLOADS_DIR = '/app/uploads';

function getSafePath(userPath: string): string {
  // Resolve to absolute path
  const resolved = path.resolve(UPLOADS_DIR, userPath);

  // Ensure it's within allowed directory
  if (!resolved.startsWith(UPLOADS_DIR)) {
    throw new ValidationError('INVALID_PATH', 'Path traversal not allowed');
  }

  return resolved;
}

// Usage
const filePath = getSafePath(req.params.filename);
```

## JSON Validation

```typescript
// Limit JSON body size
app.use(express.json({ limit: '100kb' }));

// Validate JSON structure
const requestSchema = z.object({
  action: z.enum(['create', 'update', 'delete']),
  data: z.record(z.unknown()).refine(
    obj => Object.keys(obj).length <= 50,
    'Too many fields'
  )
});
```

## Checklist

### Every User Input
- [ ] Validated with Zod schema
- [ ] Length/size limits enforced
- [ ] Type checked (string, number, etc.)
- [ ] Format validated (email, URL, etc.)

### Rich Text/HTML
- [ ] Sanitized with DOMPurify
- [ ] Allowlist of tags/attributes
- [ ] Sanitize before storage AND display

### File Uploads
- [ ] Extension validated against allowlist
- [ ] MIME type validated
- [ ] File size limited
- [ ] Magic bytes verified

### Database Queries
- [ ] Using Prisma or parameterized queries
- [ ] Dynamic values validated against allowlist
- [ ] No string concatenation
