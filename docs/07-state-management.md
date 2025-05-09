# State Management

This guide outlines our approach to state management in Next.js applications, focusing on a server-first approach with client state integration.

## Types of State

In Next.js applications, we categorize state into four main types:

1. **Server State**: Data fetched from external sources (APIs, databases)
2. **UI State**: Temporary visual state (open/closed modals, active tabs)
3. **Form State**: User inputs and form submission state
4. **Application State**: Global state shared across components

## Server State

For server data, we follow these principles:

1. Fetch data on the server when possible
2. Pass data down as props to client components
3. Use React Query or SWR for client-side updates

### Server Components Data Flow

```typescript
// app/products/[id]/page.tsx (Server Component)
export default async function ProductPage({ params }: { params: { id: string } }) {
  // Fetch on the server
  const product = await fetchProduct(params.id);
  const relatedProducts = await fetchRelatedProducts(params.id);
  
  return (
    <div>
      {/* Pass data as props */}
      <ProductDetails product={product} />
      <RelatedProducts products={relatedProducts} />
      
      {/* For interactive content that needs client-side data */}
      <AddToCartSection product={product} />
    </div>
  );
}
```

### Client Components with React Query

For client-side data fetching and state management, use React Query:

```typescript
// lib/hooks/useProduct.ts
"use client";

import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { fetchProduct, updateProduct } from '@/lib/api/products';

export function useProduct(productId: string, initialData: Product) {
  const queryClient = useQueryClient();
  
  // Query with initial data from server
  const query = useQuery({
    queryKey: ['product', productId],
    queryFn: () => fetchProduct(productId),
    initialData,
  });
  
  // Mutation
  const mutation = useMutation({
    mutationFn: (updates: Partial<Product>) => 
      updateProduct(productId, updates),
    onSuccess: (updatedProduct) => {
      queryClient.setQueryData(['product', productId], updatedProduct);
    },
  });
  
  return {
    product: query.data,
    isLoading: query.isLoading,
    isError: query.isError,
    updateProduct: mutation.mutate,
    isUpdating: mutation.isPending,
  };
}
```

```typescript
// components/products/EditProductForm.tsx
"use client";

import { useProduct } from '@/lib/hooks/useProduct';

export default function EditProductForm({
  productId,
  initialData,
}: {
  productId: string;
  initialData: Product;
}) {
  const {
    product,
    isLoading,
    updateProduct,
    isUpdating
  } = useProduct(productId, initialData);
  
  // Form handling
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    // Get form data
    updateProduct({
      name: formData.name,
      price: formData.price,
    });
  };
  
  if (isLoading) return <LoadingSpinner />;
  
  return (
    <form onSubmit={handleSubmit}>
      {/* Form fields */}
      <button type="submit" disabled={isUpdating}>
        {isUpdating ? 'Saving...' : 'Save Changes'}
      </button>
    </form>
  );
}
```

## UI State

For UI state, manage locally within components when possible:

```typescript
"use client";

import { useState } from 'react';

export default function Tabs({ children, tabs }) {
  const [activeTab, setActiveTab] = useState(0);
  
  return (
    <div>
      <div className="tabs">
        {tabs.map((tab, index) => (
          <button
            key={index}
            className={activeTab === index ? 'active' : ''}
            onClick={() => setActiveTab(index)}
          >
            {tab.label}
          </button>
        ))}
      </div>
      <div className="tab-content">
        {children[activeTab]}
      </div>
    </div>
  );
}
```

For more complex UI state, use a custom hook:

```typescript
"use client";

// hooks/useDisclosure.ts
import { useState, useCallback } from 'react';

export function useDisclosure(initial = false) {
  const [isOpen, setIsOpen] = useState(initial);
  
  const open = useCallback(() => setIsOpen(true), []);
  const close = useCallback(() => setIsOpen(false), []);
  const toggle = useCallback(() => setIsOpen(prev => !prev), []);
  
  return { isOpen, open, close, toggle };
}

// Usage
function Modal({ children }) {
  const { isOpen, open, close } = useDisclosure();
  
  return (
    <>
      <button onClick={open}>Open Modal</button>
      {isOpen && (
        <div className="modal">
          <div className="modal-content">
            {children}
            <button onClick={close}>Close</button>
          </div>
        </div>
      )}
    </>
  );
}
```

## Form State

For form state, use React Hook Form with Zod validation:

```typescript
"use client";

// components/forms/UserForm.tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

// Define validation schema
const userSchema = z.object({
  name: z.string().min(2).max(50),
  email: z.string().email('Invalid email address'),
  age: z.number().min(18).optional(),
});

type UserFormData = z.infer<typeof userSchema>;

export default function UserForm({
  defaultValues,
  onSubmit,
}: {
  defaultValues?: Partial<UserFormData>;
  onSubmit: (data: UserFormData) => void;
}) {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<UserFormData>({
    resolver: zodResolver(userSchema),
    defaultValues: defaultValues || {
      name: '',
      email: '',
    },
  });
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label htmlFor="name">Name</label>
        <input id="name" {...register('name')} />
        {errors.name && <p className="error">{errors.name.message}</p>}
      </div>
      
      <div>
        <label htmlFor="email">Email</label>
        <input id="email" type="email" {...register('email')} />
        {errors.email && <p className="error">{errors.email.message}</p>}
      </div>
      
      <div>
        <label htmlFor="age">Age (optional)</label>
        <input
          id="age"
          type="number"
          {...register('age', { valueAsNumber: true })}
        />
        {errors.age && <p className="error">{errors.age.message}</p>}
      </div>
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Submitting...' : 'Submit'}
      </button>
    </form>
  );
}
```

## Application State

For global application state, we prefer a combination of:

1. React Context for UI-related global state
2. React Query for server state

### Context-Based State

```typescript
// context/ThemeContext.tsx
"use client";

import { createContext, useContext, useState, useEffect } from 'react';

type Theme = 'light' | 'dark' | 'system';

interface ThemeContextType {
  theme: Theme;
  setTheme: (theme: Theme) => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<Theme>('system');
  
  // Apply theme
  useEffect(() => {
    if (theme === 'system') {
      const systemTheme = window.matchMedia('(prefers-color-scheme: dark)').matches
        ? 'dark'
        : 'light';
      document.documentElement.classList.toggle('dark', systemTheme === 'dark');
    } else {
      document.documentElement.classList.toggle('dark', theme === 'dark');
    }
  }, [theme]);
  
  const value = { theme, setTheme };
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
}
```

### React Query Provider

```typescript
// app/providers.tsx
"use client";

import { useState } from 'react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
import { ThemeProvider } from '@/context/ThemeContext';

export function Providers({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(() => new QueryClient({
    defaultOptions: {
      queries: {
        staleTime: 60 * 1000, // 1 minute
        retry: 1,
      },
    },
  }));
  
  return (
    <QueryClientProvider client={queryClient}>
      <ThemeProvider>
        {children}
      </ThemeProvider>
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

```typescript
// app/layout.tsx
import { Providers } from './providers';

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

## State Management Decision Framework

When deciding how to manage state, follow this decision tree:

1. **Is this server data?**
   - Yes → Fetch on the server if possible, use React Query/SWR for client updates
   - No → Continue

2. **Is this state used by multiple components?**
   - Yes → Consider React Context
   - No → Continue

3. **Is this form state?**
   - Yes → Use React Hook Form
   - No → Continue

4. **Is this UI state?**
   - Yes → Use local useState or custom hooks

## Best Practices

1. **Server-first approach**: Fetch data on the server when possible
2. **Avoid premature global state**: Start with local state and lift up as needed
3. **Colocation**: Keep state close to where it's used
4. **Use React Query for server state**: Provides caching, refetching, and more
5. **Simple abstractions**: Create custom hooks for common patterns
6. **Type safety**: Ensure all state is properly typed
