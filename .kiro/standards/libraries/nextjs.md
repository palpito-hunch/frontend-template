# Next.js Usage Standards

## Purpose

Next.js App Router patterns and best practices. These standards ensure consistent usage of Server Components, Client Components, API routes, and data fetching.

---

## Component Types

### Server Components (Default)

```typescript
// ✅ Server Component - no 'use client' directive
// Can use async/await, access server resources directly
async function UserProfile({ userId }: { userId: string }): Promise<JSX.Element> {
  const user = await prisma.user.findUnique({ where: { id: userId } });

  if (!user) {
    notFound();
  }

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

### Client Components

```typescript
'use client';

// ✅ Client Component - interactive, uses hooks
import { useState } from 'react';

export function Counter(): JSX.Element {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

### When to Use Each

| Use Server Component | Use Client Component |
|---------------------|---------------------|
| Fetching data | Event handlers (onClick, onChange) |
| Accessing backend resources | useState, useEffect, useReducer |
| Keeping secrets server-side | Browser APIs (localStorage, window) |
| Heavy dependencies | Interactivity and real-time updates |

---

## Data Fetching

### Server Components (Preferred)

```typescript
// ✅ Direct database access in Server Components
async function ProductList(): Promise<JSX.Element> {
  const products = await prisma.product.findMany({
    take: 20,
    orderBy: { createdAt: 'desc' },
  });

  return (
    <ul>
      {products.map(product => (
        <li key={product.id}>{product.name}</li>
      ))}
    </ul>
  );
}
```

### With Loading States

```typescript
// app/products/loading.tsx
export default function Loading(): JSX.Element {
  return <ProductListSkeleton />;
}

// app/products/page.tsx
export default async function ProductsPage(): Promise<JSX.Element> {
  const products = await getProducts();
  return <ProductList products={products} />;
}
```

### With Error Handling

```typescript
// app/products/error.tsx
'use client';

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}): JSX.Element {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

---

## API Routes (Route Handlers)

### Basic Pattern

```typescript
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function GET(request: NextRequest): Promise<NextResponse> {
  const users = await prisma.user.findMany();
  return NextResponse.json(users);
}

export async function POST(request: NextRequest): Promise<NextResponse> {
  const body = await request.json();
  const user = await prisma.user.create({ data: body });
  return NextResponse.json(user, { status: 201 });
}
```

### With Dynamic Routes

```typescript
// app/api/users/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server';

type Params = { params: Promise<{ id: string }> };

export async function GET(
  request: NextRequest,
  { params }: Params
): Promise<NextResponse> {
  const { id } = await params;
  const user = await prisma.user.findUnique({ where: { id } });

  if (!user) {
    return NextResponse.json(
      { error: 'User not found' },
      { status: 404 }
    );
  }

  return NextResponse.json(user);
}
```

### Error Handling Pattern

```typescript
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ApplicationError } from '@/errors';

export async function POST(request: NextRequest): Promise<NextResponse> {
  try {
    const body = await request.json();
    const user = await userService.createUser(body);
    return NextResponse.json(user, { status: 201 });
  } catch (error) {
    if (error instanceof ApplicationError) {
      return NextResponse.json(
        error.toJSON(),
        { status: error.statusCode }
      );
    }
    console.error('Unexpected error:', error);
    return NextResponse.json(
      { error: 'INTERNAL_ERROR', message: 'An unexpected error occurred' },
      { status: 500 }
    );
  }
}
```

---

## Server Actions

### Definition

```typescript
// app/actions/user.ts
'use server';

import { revalidatePath } from 'next/cache';

export async function createUser(formData: FormData): Promise<void> {
  const name = formData.get('name') as string;
  const email = formData.get('email') as string;

  await prisma.user.create({
    data: { name, email },
  });

  revalidatePath('/users');
}
```

### Usage in Forms

```typescript
// app/users/new/page.tsx
import { createUser } from '@/app/actions/user';

export default function NewUserPage(): JSX.Element {
  return (
    <form action={createUser}>
      <input name="name" required />
      <input name="email" type="email" required />
      <button type="submit">Create User</button>
    </form>
  );
}
```

---

## Caching and Revalidation

### Static Data

```typescript
// Cached indefinitely by default
async function getStaticData(): Promise<Data> {
  const res = await fetch('https://api.example.com/data');
  return res.json();
}
```

### Revalidate on Interval

```typescript
// Revalidate every hour
async function getProducts(): Promise<Product[]> {
  const res = await fetch('https://api.example.com/products', {
    next: { revalidate: 3600 },
  });
  return res.json();
}
```

### No Cache (Dynamic)

```typescript
// Always fetch fresh data
async function getCurrentUser(): Promise<User> {
  const res = await fetch('https://api.example.com/me', {
    cache: 'no-store',
  });
  return res.json();
}
```

### Manual Revalidation

```typescript
import { revalidatePath, revalidateTag } from 'next/cache';

// Revalidate specific path
revalidatePath('/products');

// Revalidate by tag
revalidateTag('products');
```

---

## Metadata

### Static Metadata

```typescript
// app/about/page.tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'About Us',
  description: 'Learn more about our company',
};
```

### Dynamic Metadata

```typescript
// app/products/[id]/page.tsx
import type { Metadata } from 'next';

type Props = { params: Promise<{ id: string }> };

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const { id } = await params;
  const product = await getProduct(id);

  return {
    title: product.name,
    description: product.description,
  };
}
```

---

## File Structure

```
app/
├── (auth)/                 # Route group (no URL segment)
│   ├── login/
│   │   └── page.tsx
│   └── register/
│       └── page.tsx
├── api/
│   └── users/
│       ├── route.ts        # GET /api/users, POST /api/users
│       └── [id]/
│           └── route.ts    # GET/PUT/DELETE /api/users/:id
├── users/
│   ├── page.tsx            # /users
│   ├── loading.tsx         # Loading UI
│   ├── error.tsx           # Error UI
│   └── [id]/
│       └── page.tsx        # /users/:id
├── layout.tsx              # Root layout
└── page.tsx                # Home page
```

---

## Summary

| Pattern | When to Use |
|---------|-------------|
| Server Components | Data fetching, backend access, no interactivity |
| Client Components | Interactivity, hooks, browser APIs |
| Route Handlers | REST API endpoints |
| Server Actions | Form submissions, mutations |
| loading.tsx | Streaming/suspense loading states |
| error.tsx | Error boundaries per route |
