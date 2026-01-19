# Prisma Usage Standards

## Purpose

Prisma-specific patterns and best practices for database operations. These rules prevent common pitfalls like N+1 queries, race conditions, and data inconsistencies.

---

## Transaction Requirements

### Always Use Transactions For

```typescript
// ✅ Multiple related writes
await prisma.$transaction(async (tx) => {
  const user = await tx.user.create({ data: userData });
  await tx.profile.create({ data: { userId: user.id, ...profileData } });
  return user;
});

// ✅ Read-then-write operations (prevents race conditions)
await prisma.$transaction(async (tx) => {
  const account = await tx.account.findUnique({ where: { id } });
  if (!account || account.balance < amount) {
    throw new InsufficientBalanceError(id, amount, account?.balance ?? 0);
  }
  return tx.account.update({
    where: { id },
    data: { balance: { decrement: amount } },
  });
});

// ✅ Operations requiring atomicity
await prisma.$transaction(async (tx) => {
  await tx.order.update({ where: { id }, data: { status: 'confirmed' } });
  await tx.inventory.update({
    where: { productId },
    data: { quantity: { decrement: orderQuantity } },
  });
});
```

### Skip Transactions For

```typescript
// ✅ Single read operations
const user = await prisma.user.findUnique({ where: { id } });

// ✅ Single write with no dependencies
const user = await prisma.user.create({ data: userData });

// ✅ Batch operations (already atomic)
await prisma.user.updateMany({
  where: { status: 'inactive' },
  data: { archivedAt: new Date() },
});
```

---

## N+1 Query Prevention

### Problem Pattern

```typescript
// ❌ N+1 query - executes 1 + N queries
const users = await prisma.user.findMany();
for (const user of users) {
  const posts = await prisma.post.findMany({
    where: { authorId: user.id },
  });
  // Process posts...
}
```

### Solution Patterns

```typescript
// ✅ Use include for relations
const users = await prisma.user.findMany({
  include: {
    posts: true,
  },
});

// ✅ Use select for specific fields
const users = await prisma.user.findMany({
  select: {
    id: true,
    name: true,
    posts: {
      select: {
        id: true,
        title: true,
      },
    },
  },
});

// ✅ Batch query with IN clause
const userIds = users.map(u => u.id);
const posts = await prisma.post.findMany({
  where: { authorId: { in: userIds } },
});
const postsByUser = groupBy(posts, 'authorId');
```

---

## Query Optimization

### Select Only Needed Fields

```typescript
// ❌ Loads all fields
const users = await prisma.user.findMany();

// ✅ Select specific fields
const users = await prisma.user.findMany({
  select: {
    id: true,
    email: true,
    name: true,
  },
});
```

### Use Pagination

```typescript
// ❌ Loads all records
const posts = await prisma.post.findMany();

// ✅ Paginate results
const posts = await prisma.post.findMany({
  take: 20,
  skip: page * 20,
  orderBy: { createdAt: 'desc' },
});

// ✅ Cursor-based pagination (better for large datasets)
const posts = await prisma.post.findMany({
  take: 20,
  cursor: lastPostId ? { id: lastPostId } : undefined,
  skip: lastPostId ? 1 : 0,
  orderBy: { id: 'asc' },
});
```

### Use Count Instead of Length

```typescript
// ❌ Loads all records just to count
const users = await prisma.user.findMany({ where: { active: true } });
const count = users.length;

// ✅ Use count query
const count = await prisma.user.count({ where: { active: true } });
```

---

## Error Handling

### Handle Known Errors

```typescript
import { Prisma } from '@prisma/client';

try {
  await prisma.user.create({ data: userData });
} catch (error) {
  if (error instanceof Prisma.PrismaClientKnownRequestError) {
    // Unique constraint violation
    if (error.code === 'P2002') {
      const field = (error.meta?.target as string[])?.[0];
      throw new ConflictError(`${field} already exists`, { field });
    }
    // Record not found
    if (error.code === 'P2025') {
      throw new NotFoundError('Record', 'unknown');
    }
  }
  throw error;
}
```

### Common Error Codes

| Code | Description | Handling |
|------|-------------|----------|
| P2002 | Unique constraint violation | Return 409 Conflict |
| P2025 | Record not found | Return 404 Not Found |
| P2003 | Foreign key constraint failure | Return 400 Bad Request |
| P2024 | Connection timeout | Retry or return 503 |

---

## Type Safety

### Use Generated Types

```typescript
import { Prisma, User } from '@prisma/client';

// ✅ Use generated input types
type CreateUserInput = Prisma.UserCreateInput;
type UpdateUserInput = Prisma.UserUpdateInput;

// ✅ Use generated select types
type UserWithPosts = Prisma.UserGetPayload<{
  include: { posts: true };
}>;

// ✅ Type-safe where clauses
const where: Prisma.UserWhereInput = {
  email: { contains: '@example.com' },
  active: true,
};
```

---

## Connection Management

### Use Single Instance

```typescript
// lib/prisma.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: process.env.NODE_ENV === 'development'
      ? ['query', 'error', 'warn']
      : ['error'],
  });

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}
```

---

## Summary

| Rule | Enforcement |
|------|-------------|
| Transactions for multiple writes | Required |
| Transactions for read-then-write | Required |
| N+1 query prevention | Required |
| Select specific fields | Recommended |
| Pagination for lists | Required |
| Handle known error codes | Required |
| Use generated types | Recommended |
| Single Prisma instance | Required |
