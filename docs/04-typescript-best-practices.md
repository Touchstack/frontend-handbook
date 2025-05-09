# TypeScript Best Practices

This guide outlines our TypeScript standards and best practices for Next.js applications.

## Type Definitions

### Centralized Types

Place shared types in a dedicated `/types` directory:

```typescript
// types/index.ts - Export all types from here
export * from './user';
export * from './product';
// etc.

// types/user.ts
export interface User {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user' | 'guest';
  createdAt: string;
}

// types/product.ts
export interface Product {
  id: string;
  name: string;
  description: string;
  price: number;
  category: ProductCategory;
  stock: number;
}

export type ProductCategory = 'electronics' | 'clothing' | 'books' | 'other';
```

### Component Props

Define props with interfaces for better documentation and type checking:

```typescript
interface UserProfileProps {
  user: User;
  isEditable?: boolean;
  onUpdate?: (updatedUser: Partial<User>) => Promise<void>;
}

export default function UserProfile({ 
  user, 
  isEditable = false,
  onUpdate
}: UserProfileProps) {
  // Component implementation
}
```

## Type Safety Standards

### Strict TypeScript Configuration

Use a strict `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

### Avoid Type Assertions

Prefer proper type checking over assertions:

```typescript
// ❌ Avoid
const user = data as User;

// ✅ Prefer
function isUser(data: unknown): data is User {
  return (
    typeof data === 'object' && 
    data !== null && 
    'id' in data && 
    'name' in data && 
    'email' in data
  );
}

if (isUser(data)) {
  // data is typed as User here
}
```

### Use Unknown Instead of Any

When type is truly unknown, use `unknown` instead of `any`:

```typescript
// ❌ Avoid
function processData(data: any) {
  return data.value; // No type checking
}

// ✅ Prefer
function processData(data: unknown) {
  if (typeof data === 'object' && data !== null && 'value' in data) {
    return data.value;
  }
  throw new Error('Invalid data format');
}
```

## API Type Safety

### API Response Types

Define types for API responses:

```typescript
// types/api.ts
export interface ApiResponse<T> {
  data: T;
  meta?: {
    page: number;
    pageSize: number;
    total: number;
  };
}

export interface ApiError {
  message: string;
  code: string;
  status: number;
}
```

### Route Handlers

Type API route handlers:

```typescript
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { User } from '@/types';

export async function GET(
  request: NextRequest
): Promise<NextResponse<ApiResponse<User[]> | ApiError>> {
  try {
    const users = await fetchUsers();
    return NextResponse.json({ data: users });
  } catch (error) {
    return NextResponse.json(
      { message: 'Failed to fetch users', code: 'FETCH_ERROR', status: 500 },
      { status: 500 }
    );
  }
}
```

## Form Handling

Use Zod for runtime validation with TypeScript integration:

```typescript
// lib/validations/user.ts
import { z } from 'zod';

export const userSchema = z.object({
  name: z.string().min(2).max(50),
  email: z.string().email(),
  password: z.string().min(8).max(100),
});

export type UserFormData = z.infer<typeof userSchema>;
```

```typescript
"use client";

// components/UserForm.tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { userSchema, UserFormData } from '@/lib/validations/user';

export default function UserForm() {
  const form = useForm<UserFormData>({
    resolver: zodResolver(userSchema),
    defaultValues: {
      name: '',
      email: '',
      password: '',
    },
  });
  
  // Form implementation
}
```

## Type Utilities

Create custom type utilities for common patterns:

```typescript
// types/utilities.ts

// Make specific properties required
export type RequireFields<T, K extends keyof T> = T & {
  [P in K]-?: T[P];
};

// Extract keys that have a specific type
export type KeysOfType<T, V> = {
  [K in keyof T]-?: T[K] extends V ? K : never;
}[keyof T];

// Convert undefined to null
export type NullableProps<T> = {
  [K in keyof T]: T[K] | null;
};
```

## TypeScript and Data Fetching

Type your API client functions:

```typescript
// lib/api/users.ts
import { User, ApiResponse, ApiError } from '@/types';

export async function fetchUser(id: string): Promise<User> {
  const res = await fetch(`/api/users/${id}`);
  
  if (!res.ok) {
    const error: ApiError = await res.json();
    throw new Error(error.message);
  }
  
  const { data }: ApiResponse<User> = await res.json();
  return data;
}
```

## Best Practices

1. **Always use TypeScript** for all new files
2. **Define explicit return types** for functions and API calls
3. **Use interfaces for objects** that will be extended
4. **Use type for unions** and simple object types
5. **Avoid `any`** - use `unknown` when necessary
6. **Enable strict mode** in tsconfig.json
7. **Use type guards** for runtime type checking
