# Data Fetching

This guide outlines our standardized approach to data fetching in Next.js applications, following the SSR-first principle.

## Core Principles

1. **Fetch on the Server**: Always prioritize server-side data fetching
2. **Pass as Props**: Send data from server components to client components as props
3. **Hydration**: Use fetched server data as initial data for client-side data management

## Server-Side Data Fetching

### In Server Components

Server Components can directly fetch data using async/await:

```typescript
// app/products/page.tsx
export default async function ProductsPage() {
  // Native fetch with automatic deduplication
  const products = await fetch('https://api.example.com/products')
    .then(res => res.json());
  
  return (
    <div>
      <h1>Products</h1>
      <ProductGrid products={products} />
    </div>
  );
}
```

### Using Next.js Fetch with Caching

Take advantage of Next.js built-in caching:

```typescript
// app/products/[id]/page.tsx
export default async function ProductPage({ params }: { params: { id: string } }) {
  // With caching controls - revalidate every 60 seconds
  const product = await fetch(`https://api.example.com/products/${params.id}`, {
    next: { revalidate: 60 }
  }).then(res => res.json());
  
  return <ProductDetails product={product} />;
}
```

For static data:

```typescript
// Static data (cached at build time)
const product = await fetch(`https://api.example.com/products/${params.id}`, {
  cache: 'force-cache'
}).then(res => res.json());
```

For dynamic data:

```typescript
// Dynamic data (never cached)
const product = await fetch(`https://api.example.com/products/${params.id}`, {
  cache: 'no-store'
}).then(res => res.json());
```

## Client-Side Data Fetching

For client-side interactivity, we recommend:

- React Query (preferred)
- SWR

### With React Query

```typescript
// lib/api/products.ts
export async function fetchProduct(id: string) {
  const res = await fetch(`/api/products/${id}`);
  if (!res.ok) throw new Error('Failed to fetch product');
  return res.json();
}
```

```typescript
"use client";

// components/ProductActions.tsx
import { useQuery } from '@tanstack/react-query';
import { fetchProduct } from '@/lib/api/products';

export default function ProductActions({ 
  productId, 
  initialData 
}: { 
  productId: string;
  initialData: Product;
}) {
  // Use server-fetched data as initial data
  const { data: product } = useQuery({
    queryKey: ['product', productId],
    queryFn: () => fetchProduct(productId),
    initialData,
  });
  
  return (
    <div>
      <h2>{product.name}</h2>
      <button>Add to Cart</button>
    </div>
  );
}
```

### With SWR

```typescript
"use client";

// components/ProductStock.tsx
import useSWR from 'swr';

export default function ProductStock({ 
  productId, 
  initialStock 
}: { 
  productId: string;
  initialStock: number;
}) {
  // Use server-fetched data as fallback
  const { data: stock } = useSWR(
    `/api/products/${productId}/stock`, 
    fetcher, 
    { fallbackData: initialStock }
  );
  
  return <div>In stock: {stock}</div>;
}
```

## Data Fetching Pattern

1. **Fetch data in server components**
2. **Pass fetched data to client components**
3. **Use server data as initialData/fallback for client data libraries**

This approach provides:

- Fast initial page loads with SSR
- SEO benefits from server-rendered content
- Seamless transitions to client-side interactivity
- Reduced waterfall requests

## API Route Organization

Create API routes in `/app/api` with clear naming:

```typescript
// app/api/products/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const product = await fetchProductFromDatabase(params.id);
    return NextResponse.json(product);
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to fetch product' },
      { status: 500 }
    );
  }
}
```

## Common Pitfalls

1. ❌ Fetching data on the client when it could be done on the server
2. ❌ Not handling loading and error states
3. ❌ Not using proper caching strategies
4. ❌ Duplicating fetching logic between server and client
