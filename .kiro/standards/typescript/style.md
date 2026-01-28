# TypeScript Style Guide

## Purpose

TypeScript-specific style rules that complement ESLint configuration. These rules guide code generation to produce lint-compliant, idiomatic TypeScript.

---

## Enforced Rules Reference

These rules are enforced by ESLint and TypeScript compiler. Violations will fail CI.

### ESLint Rules (Error Level)

| Rule | Description | See Section |
|------|-------------|-------------|
| `explicit-function-return-type` | Functions must have return types | [Type Annotations](#type-annotations) |
| `explicit-module-boundary-types` | Exported functions must have explicit types | [Type Annotations](#type-annotations) |
| `no-floating-promises` | Promises must be awaited or returned | [Async/Await](#asyncawait) |
| `only-throw-error` | Only throw Error subclasses | [Error Classes](#error-classes) |
| `no-unused-vars` | No unused variables (prefix with `_` to ignore) | [Variables](#variables) |
| `prefer-const` | Use `const` when variable is never reassigned | [Variables](#variables) |
| `no-var` | Never use `var`, use `let` or `const` | [Variables](#variables) |

### ESLint Rules (Warning Level)

| Rule | Description | See Section |
|------|-------------|-------------|
| `no-explicit-any` | Avoid `any` type | [Any Type](#avoiding-any) |
| `no-unsafe-assignment` | Avoid assigning `any` to typed variables | [Any Type](#avoiding-any) |
| `no-unsafe-member-access` | Avoid accessing members of `any` | [Any Type](#avoiding-any) |
| `no-unsafe-return` | Avoid returning `any` from typed functions | [Any Type](#avoiding-any) |
| `no-console` | No `console.log` (allow `console.warn`, `console.error`) | [Logging](#logging) |

### TypeScript Compiler Options

| Option | Effect | See Section |
|--------|--------|-------------|
| `strict: true` | Enables all strict type-checking | All sections |
| `noUncheckedIndexedAccess` | Array/object access returns `T \| undefined` | [Array Access](#array-access-with-nouncheckedindexedaccess) |
| `exactOptionalPropertyTypes` | Optional props can't be explicitly `undefined` | [Optional Properties](#optional-properties) |

### Test File Exceptions

These rules are relaxed in test files (`**/*.test.ts`, `**/__tests__/**`):
- `no-explicit-any`: off
- `no-unsafe-*`: off
- `explicit-function-return-type`: off

---

## Type Annotations

### explicit-function-return-type (ESLint: error)

All functions must have explicit return type annotations.

```typescript
// ❌ ESLint error: Missing return type on function
function calculateTotal(items: Item[]) {
  return items.reduce((sum, item) => sum + item.price, 0);
}

// ✅ Add explicit return type
function calculateTotal(items: Item[]): number {
  return items.reduce((sum, item) => sum + item.price, 0);
}

// ❌ ESLint error: Missing return type on function
async function fetchUser(id: string) {
  return await repository.findById(id);
}

// ✅ Add explicit Promise return type
async function fetchUser(id: string): Promise<User | null> {
  return await repository.findById(id);
}
```

### explicit-module-boundary-types (ESLint: error)

Exported functions and class methods must have explicit types for all parameters and return values.

```typescript
// ❌ ESLint error: Missing return type on exported function
export function createUser(data) {
  return repository.create(data);
}

// ✅ Explicit types for exported functions
export function createUser(data: CreateUserInput): Promise<User> {
  return repository.create(data);
}

// ❌ ESLint error: Missing return type on public method
export class UserService {
  async findById(id) {
    return this.repository.findById(id);
  }
}

// ✅ Explicit types for class methods
export class UserService {
  async findById(id: string): Promise<User | null> {
    return this.repository.findById(id);
  }
}
```

### When Types Are Inferred (allowExpressions)

The ESLint rule allows type inference in certain cases:

```typescript
// ✅ Arrow functions in callbacks (allowTypedFunctionExpressions)
items.map(item => item.name);  // Return type inferred from arrow function
items.filter(item => item.active);

// ✅ Higher-order functions (allowHigherOrderFunctions)
const withLogging = (fn: () => void) => () => {
  console.log('Calling function');
  fn();
};

// ✅ Inline function expressions (allowExpressions)
const handler = function(event: Event) {
  // Return type inferred
};

// ⚠️ But named functions still need explicit types
function handler(event: Event): void {  // Return type required
  // ...
}
```

### Class Properties

```typescript
// ✅ Class properties with types
class UserService {
  private readonly repository: UserRepository;
  private cache: Map<string, User> = new Map();
  private isInitialized = false;  // Type inferred from initializer
}

// ✅ Complex object destructuring in parameters
function processConfig({ timeout, retries }: { timeout: number; retries: number }): void {
  // ...
}
```

### Variable Annotations

```typescript
// ✅ Type inferred from initializer (no annotation needed)
const name = 'John';  // string
const count = 0;      // number
const isActive = true;  // boolean

// ⚠️ Empty arrays need explicit type
const users: User[] = [];  // Can't infer type from empty array

// ✅ Type inferred from function return
const user = await fetchUser(id);  // Type inferred from fetchUser return type
```

---

## Variables

### const vs let (ESLint: `prefer-const`, `no-var`)

```typescript
// ❌ ESLint error: 'no-var'
var name = 'John';

// ❌ ESLint error: 'prefer-const' - never reassigned
let count = 0;
console.log(count);

// ✅ Use const for values that don't change
const name = 'John';
const count = 0;

// ✅ Use let only when reassignment is needed
let total = 0;
for (const item of items) {
  total += item.price;
}
```

### Unused Variables (ESLint: `no-unused-vars`)

```typescript
// ❌ ESLint error: 'userId' is defined but never used
function processUser(userId: string, userData: UserData): void {
  saveUser(userData);
}

// ✅ Prefix unused parameters with underscore
function processUser(_userId: string, userData: UserData): void {
  saveUser(userData);
}

// ✅ Or use object destructuring with rest
function processUser({ name, email }: UserData): void {
  saveUser({ name, email });
}
```

---

## Avoiding Any

### no-explicit-any (ESLint: warning)

```typescript
// ⚠️ ESLint warning: Unexpected any
function processData(data: any): any {
  return data.value;
}

// ✅ Use unknown for truly unknown types
function processData(data: unknown): string {
  if (typeof data === 'object' && data !== null && 'value' in data) {
    return String(data.value);
  }
  throw new Error('Invalid data format');
}

// ✅ Use generics for flexible but type-safe functions
function processData<T extends { value: string }>(data: T): string {
  return data.value;
}
```

### no-unsafe-assignment (ESLint: warning)

```typescript
// ⚠️ ESLint warning: Unsafe assignment of an `any` value
const response = JSON.parse(jsonString);
const userName: string = response.name;

// ✅ Type the parsed result
interface ApiResponse {
  name: string;
  email: string;
}
const response = JSON.parse(jsonString) as ApiResponse;
const userName: string = response.name;

// ✅ Better: Use a validation library like zod
import { z } from 'zod';
const ResponseSchema = z.object({ name: z.string(), email: z.string() });
const response = ResponseSchema.parse(JSON.parse(jsonString));
```

### no-unsafe-member-access (ESLint: warning)

```typescript
// ⚠️ ESLint warning: Unsafe member access .name on an `any` value
function getName(obj: any): string {
  return obj.name;
}

// ✅ Type the parameter properly
interface Named {
  name: string;
}
function getName(obj: Named): string {
  return obj.name;
}
```

### no-unsafe-return (ESLint: warning)

```typescript
// ⚠️ ESLint warning: Unsafe return of an `any` typed value
function parseConfig(json: string): Config {
  return JSON.parse(json);  // Returns any
}

// ✅ Validate and type the return
function parseConfig(json: string): Config {
  const parsed: unknown = JSON.parse(json);
  if (!isValidConfig(parsed)) {
    throw new ConfigError('Invalid configuration format');
  }
  return parsed;
}
```

---

## Logging

### no-console (ESLint: warning)

```typescript
// ⚠️ ESLint warning: Unexpected console statement
console.log('User created:', user);

// ✅ Use console.warn for warnings
console.warn('Deprecated API usage detected');

// ✅ Use console.error for errors
console.error('Failed to connect to database:', error);

// ✅ For production, use a proper logger
import { logger } from '@/utils/logger';
logger.info('User created', { userId: user.id });
logger.error('Database connection failed', { error });
```

---

## Optional Properties

### exactOptionalPropertyTypes (TypeScript)

```typescript
interface User {
  name: string;
  nickname?: string;  // Optional property
}

// ❌ TypeScript error with exactOptionalPropertyTypes
const user: User = {
  name: 'John',
  nickname: undefined,  // Error: can't explicitly set undefined
};

// ✅ Either omit the property entirely
const user: User = {
  name: 'John',
};

// ✅ Or provide a value
const user: User = {
  name: 'John',
  nickname: 'Johnny',
};

// ✅ If undefined is valid, declare it explicitly in the type
interface UserWithExplicitUndefined {
  name: string;
  nickname?: string | undefined;  // Now undefined is allowed
}
```

---

## Type vs Interface

### Prefer `type` for

```typescript
// ✅ Union types
type Status = 'pending' | 'active' | 'completed';

// ✅ Mapped types
type Readonly<T> = { readonly [K in keyof T]: T[K] };

// ✅ Utility type combinations
type UserWithRole = User & { role: Role };

// ✅ Function types
type Handler = (event: Event) => void;

// ✅ Tuple types
type Coordinate = [number, number];
```

### Prefer `interface` for

```typescript
// ✅ Object shapes that may be extended
interface User {
  id: string;
  name: string;
  email: string;
}

// ✅ Class contracts
interface Repository<T> {
  findById(id: string): Promise<T | null>;
  save(entity: T): Promise<T>;
}

// ✅ Declaration merging needs
interface Window {
  customProperty: string;
}
```

### Decision Framework

```
Is it a union/intersection/mapped type? → type
Will it be extended/implemented?       → interface
Is it a function signature?            → type
Is it a simple object shape?           → either (be consistent)
```

---

## Null Handling

### Explicit Null Checks

```typescript
// ✅ Explicit narrowing
function getUser(id: string): User | null {
  const user = users.get(id);
  if (!user) {
    return null;  // Explicit about missing data
  }
  return user;
}

// ✅ Early return pattern
function processUser(user: User | null): void {
  if (!user) {
    throw new UserNotFoundError();
  }
  // user is now narrowed to User
  console.log(user.name);
}
```

### Avoid

```typescript
// ❌ Non-null assertion without validation
const user = getUser(id)!;  // Dangerous

// ❌ Optional chaining without fallback consideration
const name = user?.profile?.name;  // What if undefined?

// ✅ Better: explicit fallback
const name = user?.profile?.name ?? 'Unknown';
```

### Array Access (with noUncheckedIndexedAccess)

```typescript
// With noUncheckedIndexedAccess: true, array access returns T | undefined

// ❌ Assumes element exists
const first = items[0];  // Type: Item | undefined
console.log(first.name); // Error: Object is possibly undefined

// ✅ Check first
const first = items[0];
if (first) {
  console.log(first.name);
}

// ✅ Or use assertion after validation
if (items.length > 0) {
  const first = items[0]!;  // OK: we verified length
}
```

---

## Async/Await

### no-floating-promises (ESLint: error)

Promises must be awaited, returned, or explicitly voided. Unhandled promises can lead to silent failures.

```typescript
// ❌ ESLint error: Promises must be awaited
async function processUser(id: string): Promise<void> {
  fetchUser(id);  // Error: floating promise
  sendNotification(id);  // Error: another floating promise
}

// ✅ Await all promises
async function processUser(id: string): Promise<void> {
  const user = await fetchUser(id);
  await sendNotification(user.email);
}

// ✅ Return the promise (when that's the intent)
function fetchUser(id: string): Promise<User> {
  return repository.findById(id);  // No async needed
}

// ✅ Parallel execution with Promise.all
async function processUsers(ids: string[]): Promise<void> {
  await Promise.all(ids.map(id => fetchUser(id)));
}

// ✅ Explicitly void if fire-and-forget is intentional
function handleClick(): void {
  void analytics.track('button_clicked');  // Intentionally not awaited
}

// ✅ Handle errors for fire-and-forget
function scheduleCleanup(): void {
  cleanup().catch(error => {
    console.error('Cleanup failed:', error);
  });
}
```

### Error Handling

```typescript
// ✅ Try-catch with typed errors
async function fetchData(): Promise<Data> {
  try {
    const response = await api.get('/data');
    return response.data;
  } catch (error) {
    if (error instanceof ApiError) {
      throw new DataFetchError('Failed to fetch data', { cause: error });
    }
    throw error;
  }
}

// ✅ Promise.allSettled for parallel operations that can fail independently
const results = await Promise.allSettled([
  fetchUser(id1),
  fetchUser(id2),
]);
```

---

## Enums vs Union Types

### Prefer Union Types

```typescript
// ✅ String literal union (preferred)
type Status = 'draft' | 'published' | 'archived';

// Benefits:
// - No runtime code generated
// - Better type inference
// - Works with JSON serialization

function updateStatus(status: Status): void {
  // ...
}
```

### Use Const Enum Sparingly

```typescript
// ⚠️ Only when you need numeric values AND iteration
const enum HttpStatus {
  OK = 200,
  NotFound = 404,
  ServerError = 500,
}

// Note: const enum is inlined at compile time
```

### Avoid Regular Enums

```typescript
// ❌ Regular enum generates runtime code
enum Status {
  Draft,
  Published,
  Archived,
}
// Generates: var Status; (function (Status) { ...
```

---

## Generics

### Name Generics Descriptively

```typescript
// ❌ Single letters for complex generics
function transform<T, U, V>(input: T, fn: (x: U) => V): V;

// ✅ Descriptive names
function transform<TInput, TIntermediate, TOutput>(
  input: TInput,
  fn: (x: TIntermediate) => TOutput
): TOutput;

// ✅ Single letter OK for simple, obvious cases
function identity<T>(value: T): T {
  return value;
}
```

### Constrain When Possible

```typescript
// ✅ Constrained generic
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// ✅ With default
function createContainer<T = unknown>(value: T): Container<T> {
  return { value };
}
```

---

## Imports

### Import Order

```typescript
// 1. Node built-ins
import { readFile } from 'fs/promises';
import path from 'path';

// 2. External packages
import React from 'react';
import { z } from 'zod';

// 3. Internal aliases (@/*)
import { Button } from '@/components/Button';
import { useAuth } from '@/hooks/useAuth';

// 4. Relative imports
import { helper } from './utils';
import type { Config } from './types';
```

### Type-Only Imports

```typescript
// ✅ Use type imports for types only
import type { User, Role } from './types';
import { UserService } from './services';

// ✅ Inline type imports
import { UserService, type User } from './user';
```

---

## Naming Conventions

| Item | Convention | Example |
|------|------------|---------|
| Variables/functions | snake_case | `get_user_by_id`, `is_active` |
| Classes | PascalCase | `UserService`, `HttpClient` |
| Interfaces | PascalCase | `User`, `Repository` |
| Types | PascalCase | `UserId`, `Status` |
| Enums | PascalCase | `HttpStatus` |
| Constants | SCREAMING_SNAKE | `MAX_RETRIES`, `API_URL` |
| Private fields | snake_case with `private` | `private cache` |
| Boolean variables | `is_`/`has_`/`can_` prefix | `is_active`, `has_permission` |
| Event handlers | `handle_` + event | `handle_click`, `handle_submit` |
| Async functions | verb describing action | `fetch_user`, `save_order` |

**Note on 3rd party frameworks:** Use snake_case for all custom code. When interacting with framework APIs, follow their conventions (e.g., React's `useState`, `onClick`, Express's `req.body`). This means your custom functions are snake_case, but framework callbacks and props use framework conventions.

---

## Error Classes

### only-throw-error (ESLint: error)

Only throw `Error` objects or subclasses. Never throw strings, objects, or other primitives.

```typescript
// ❌ ESLint error: Expected an error object to be thrown
throw 'Something went wrong';

// ❌ ESLint error: Expected an error object to be thrown
throw { message: 'Failed', code: 500 };

// ❌ ESLint error: Expected an error object to be thrown
throw null;

// ✅ Throw Error objects
throw new Error('Something went wrong');

// ✅ Better: Use custom error classes for domain errors
throw new ValidationError('Invalid email format', { email, field: 'email' });
throw new NotFoundError('User', userId);
throw new UnauthorizedError('Session expired');
```

### Custom Error Class Pattern

```typescript
// Base application error
class ApplicationError extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly statusCode: number = 500,
    public readonly context?: Record<string, unknown>
  ) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }

  toJSON(): Record<string, unknown> {
    return {
      error: this.code,
      message: this.message,
      ...(this.context && { details: this.context }),
    };
  }
}

// Domain-specific errors
class ValidationError extends ApplicationError {
  constructor(message: string, context?: Record<string, unknown>) {
    super(message, 'VALIDATION_ERROR', 400, context);
  }
}

class NotFoundError extends ApplicationError {
  constructor(entity: string, id: string) {
    super(`${entity} not found: ${id}`, 'NOT_FOUND', 404, { entity, id });
  }
}

class UnauthorizedError extends ApplicationError {
  constructor(message = 'Unauthorized') {
    super(message, 'UNAUTHORIZED', 401);
  }
}

// ✅ Usage
throw new ValidationError('Invalid email format', { email, field: 'email' });
```

---

## Comments

### When to Comment

```typescript
// ✅ Complex business logic
// Calculate compound interest using the formula: A = P(1 + r/n)^(nt)
// where P = principal, r = annual rate, n = compounds per year, t = years
function calculateCompoundInterest(
  principal: number,
  rate: number,
  compounds: number,
  years: number
): number {
  return principal * Math.pow(1 + rate / compounds, compounds * years);
}

// ✅ Non-obvious workarounds
// Using setTimeout(0) to defer execution until after React's batch update
setTimeout(() => setCount(count + 1), 0);

// ✅ TODO with context
// TODO(ticket-123): Refactor once new API is available
```

### JSDoc for Public APIs

```typescript
/**
 * Fetches a user by their unique identifier.
 *
 * @param id - The user's unique identifier
 * @returns The user if found, null otherwise
 * @throws {DatabaseError} If the database connection fails
 *
 * @example
 * const user = await fetchUser('user-123');
 * if (user) {
 *   console.log(user.name);
 * }
 */
export async function fetchUser(id: string): Promise<User | null> {
  // ...
}
```

---

## Summary

### ESLint Errors (Will Fail CI)

| Rule | What It Checks |
|------|----------------|
| `explicit-function-return-type` | Functions must have return types |
| `explicit-module-boundary-types` | Exports must have explicit types |
| `no-floating-promises` | Promises must be handled |
| `only-throw-error` | Only throw Error subclasses |
| `no-unused-vars` | No unused variables (use `_` prefix) |
| `prefer-const` | Use const when not reassigning |
| `no-var` | Never use var |

### ESLint Warnings (Should Fix)

| Rule | What It Checks |
|------|----------------|
| `no-explicit-any` | Avoid using `any` type |
| `no-unsafe-assignment` | Don't assign `any` to typed vars |
| `no-unsafe-member-access` | Don't access `any` members |
| `no-unsafe-return` | Don't return `any` from typed functions |
| `no-console` | Use console.warn/error, not console.log |

### TypeScript Compiler (Will Fail Build)

| Option | What It Checks |
|--------|----------------|
| `strict` | All strict type checking enabled |
| `noUncheckedIndexedAccess` | Array access returns `T \| undefined` |
| `exactOptionalPropertyTypes` | Can't assign `undefined` to optional props |

### Guidelines (Not Enforced)

| Rule | Recommendation |
|------|----------------|
| Prefer `type` for unions | Use type for unions, intersections, functions |
| Prefer `interface` for objects | Use interface for object shapes, class contracts |
| Use union types over enums | Prefer string literal unions over enums |
| Type-only imports | Use `import type` for type-only imports |
| Descriptive generic names | Use `TInput` over `T` for complex generics |
