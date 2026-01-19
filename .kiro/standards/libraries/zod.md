# Zod Usage Standards

## Purpose

Zod schema validation patterns and best practices. These standards ensure consistent input validation, type inference, and error handling.

---

## Basic Schema Definition

### Primitive Types

```typescript
import { z } from 'zod';

// String validations
const emailSchema = z.string().email();
const urlSchema = z.string().url();
const uuidSchema = z.string().uuid();
const nonEmptySchema = z.string().min(1, 'Required');

// Number validations
const positiveSchema = z.number().positive();
const integerSchema = z.number().int();
const rangeSchema = z.number().min(0).max(100);

// Boolean
const boolSchema = z.boolean();

// Date
const dateSchema = z.date();
const dateStringSchema = z.string().datetime();
```

### Object Schemas

```typescript
// ✅ Define schema with all validations
const UserSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  name: z.string().min(1).max(100),
  age: z.number().int().positive().optional(),
  role: z.enum(['admin', 'user', 'guest']),
  createdAt: z.date(),
});

// ✅ Infer TypeScript type from schema
type User = z.infer<typeof UserSchema>;
```

### Array Schemas

```typescript
const TagsSchema = z.array(z.string()).min(1).max(10);
const UsersSchema = z.array(UserSchema);

// With unique constraint
const UniqueIdsSchema = z.array(z.string().uuid()).refine(
  (ids) => new Set(ids).size === ids.length,
  { message: 'IDs must be unique' }
);
```

---

## Input Validation Patterns

### API Request Validation

```typescript
// schemas/user.schema.ts
export const CreateUserSchema = z.object({
  email: z.string().email('Invalid email format'),
  name: z.string().min(1, 'Name is required').max(100),
  password: z.string().min(8, 'Password must be at least 8 characters'),
});

export const UpdateUserSchema = CreateUserSchema.partial();

export type CreateUserInput = z.infer<typeof CreateUserSchema>;
export type UpdateUserInput = z.infer<typeof UpdateUserSchema>;
```

### Usage in API Route

```typescript
// app/api/users/route.ts
import { CreateUserSchema } from '@/schemas/user.schema';

export async function POST(request: NextRequest): Promise<NextResponse> {
  const body = await request.json();

  const result = CreateUserSchema.safeParse(body);
  if (!result.success) {
    return NextResponse.json(
      {
        error: 'VALIDATION_ERROR',
        details: result.error.flatten().fieldErrors,
      },
      { status: 400 }
    );
  }

  const user = await userService.createUser(result.data);
  return NextResponse.json(user, { status: 201 });
}
```

---

## Error Handling

### Safe Parsing (Recommended)

```typescript
// ✅ Use safeParse for graceful error handling
const result = UserSchema.safeParse(data);

if (!result.success) {
  // Access structured errors
  const errors = result.error.flatten();
  console.log(errors.fieldErrors);
  // { email: ['Invalid email'], name: ['Required'] }
  return;
}

// result.data is typed as User
const user = result.data;
```

### Throwing Validation (When Appropriate)

```typescript
// ⚠️ Use parse only when you want to throw on invalid input
try {
  const user = UserSchema.parse(data);
} catch (error) {
  if (error instanceof z.ZodError) {
    throw new ValidationError('Invalid input', {
      fields: error.flatten().fieldErrors,
    });
  }
  throw error;
}
```

### Custom Error Messages

```typescript
const UserSchema = z.object({
  email: z.string({
    required_error: 'Email is required',
    invalid_type_error: 'Email must be a string',
  }).email('Please enter a valid email address'),

  age: z.number({
    required_error: 'Age is required',
  }).int('Age must be a whole number')
    .positive('Age must be positive'),
});
```

---

## Schema Composition

### Extending Schemas

```typescript
const BaseUserSchema = z.object({
  email: z.string().email(),
  name: z.string(),
});

// Extend with additional fields
const AdminUserSchema = BaseUserSchema.extend({
  permissions: z.array(z.string()),
  department: z.string(),
});
```

### Picking and Omitting

```typescript
// Pick specific fields
const UserCredentialsSchema = UserSchema.pick({
  email: true,
  password: true,
});

// Omit fields
const PublicUserSchema = UserSchema.omit({
  password: true,
  internalId: true,
});
```

### Merging Schemas

```typescript
const AddressSchema = z.object({
  street: z.string(),
  city: z.string(),
  country: z.string(),
});

const UserWithAddressSchema = UserSchema.merge(AddressSchema);
```

### Partial and Required

```typescript
// Make all fields optional
const PartialUserSchema = UserSchema.partial();

// Make specific fields optional
const UpdateUserSchema = UserSchema.partial({
  name: true,
  age: true,
});

// Make all fields required
const RequiredUserSchema = PartialUserSchema.required();
```

---

## Advanced Patterns

### Discriminated Unions

```typescript
const NotificationSchema = z.discriminatedUnion('type', [
  z.object({
    type: z.literal('email'),
    email: z.string().email(),
    subject: z.string(),
  }),
  z.object({
    type: z.literal('sms'),
    phoneNumber: z.string(),
    message: z.string().max(160),
  }),
  z.object({
    type: z.literal('push'),
    deviceToken: z.string(),
    title: z.string(),
  }),
]);

type Notification = z.infer<typeof NotificationSchema>;
```

### Custom Refinements

```typescript
const PasswordSchema = z.string()
  .min(8)
  .refine(
    (password) => /[A-Z]/.test(password),
    { message: 'Password must contain at least one uppercase letter' }
  )
  .refine(
    (password) => /[0-9]/.test(password),
    { message: 'Password must contain at least one number' }
  );

const DateRangeSchema = z.object({
  startDate: z.date(),
  endDate: z.date(),
}).refine(
  (data) => data.endDate > data.startDate,
  { message: 'End date must be after start date', path: ['endDate'] }
);
```

### Transformations

```typescript
// Transform string to number
const StringToNumberSchema = z.string().transform((val) => parseInt(val, 10));

// Transform and validate
const TrimmedStringSchema = z.string()
  .transform((val) => val.trim())
  .pipe(z.string().min(1, 'Cannot be empty after trimming'));

// Parse JSON string
const JsonSchema = z.string().transform((val, ctx) => {
  try {
    return JSON.parse(val);
  } catch {
    ctx.addIssue({
      code: z.ZodIssueCode.custom,
      message: 'Invalid JSON',
    });
    return z.NEVER;
  }
});
```

### Coercion

```typescript
// Coerce types from strings (useful for form data, query params)
const QueryParamsSchema = z.object({
  page: z.coerce.number().int().positive().default(1),
  limit: z.coerce.number().int().min(1).max(100).default(20),
  active: z.coerce.boolean().default(true),
});

// Usage with query string
const params = QueryParamsSchema.parse({
  page: '2',      // Coerced to number 2
  limit: '50',    // Coerced to number 50
  active: 'true', // Coerced to boolean true
});
```

---

## Integration with Prisma

```typescript
import { z } from 'zod';
import { Prisma } from '@prisma/client';

// Schema matching Prisma input type
const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1),
  profile: z.object({
    bio: z.string().optional(),
    avatar: z.string().url().optional(),
  }).optional(),
}) satisfies z.ZodType<Prisma.UserCreateInput>;
```

---

## Environment Variables

```typescript
// env.schema.ts
const EnvSchema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']),
  DATABASE_URL: z.string().url(),
  API_KEY: z.string().min(1),
  PORT: z.coerce.number().default(3000),
});

// Validate at startup
export const env = EnvSchema.parse(process.env);
```

---

## Summary

| Pattern | When to Use |
|---------|-------------|
| `safeParse` | API input validation, form handling |
| `parse` | Internal validation where errors should throw |
| `z.infer` | Type inference from schemas |
| `extend/merge` | Building on existing schemas |
| `pick/omit` | Creating subset schemas |
| `partial` | Update/patch operations |
| `discriminatedUnion` | Polymorphic data structures |
| `refine` | Custom validation logic |
| `transform` | Data transformation |
| `coerce` | Query params, form data |
