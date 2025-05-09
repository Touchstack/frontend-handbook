# Performance Optimization

This guide covers strategies and best practices for optimizing performance in Next.js applications.

## Server-Side Optimization

### Caching Strategies

Utilize Next.js's built-in cache mechanisms:

```typescript
// Static data that doesn't change often (cached at build time)
const staticData = await fetch('https://api.example.com/data', {
  cache: 'force-cache'
});

// Revalidate data at specific intervals (ISR)
const revalidatedData = await fetch('https://api.example.com/data', {
  next: { revalidate: 3600 } // revalidate every hour
});

// Dynamic data (never cached)
const dynamicData = await fetch('https://api.example.com/data', {
  cache: 'no-store'
});
```

### Route Segment Config

Configure caching behavior for entire route segments:

```typescript
// app/products/[id]/page.tsx
export const dynamic = 'force-dynamic'; // Always fetch fresh data
// OR
export const revalidate = 3600; // Revalidate every hour
// OR
export const dynamic = 'force-static'; // Build-time generation only
```

### Parallel Data Fetching

Fetch data in parallel to reduce waterfall requests:

```typescript
// app/dashboard/page.tsx
export default async function DashboardPage() {
  // Fetch data in parallel
  const [
    userData,
    analyticsData,
    notificationsData
  ] = await Promise.all([
    fetchUserData(),
    fetchAnalyticsData(),
    fetchNotificationsData()
  ]);
  
  return (
    <Dashboard
      userData={userData}
      analyticsData={analyticsData}
      notificationsData={notificationsData}
    />
  );
}
```

## Client-Side Optimization

### Component Optimization

Use React's memoization features:

```typescript
// components/ExpensiveComponent.tsx
"use client";

import { memo, useMemo } from 'react';

interface ExpensiveComponentProps {
  data: ComplexData[];
  onItemSelect: (id: string) => void;
}

function ExpensiveComponent({ data, onItemSelect }: ExpensiveComponentProps) {
  // Memoize expensive calculations
  const processedData = useMemo(() => {
    return data.map(item => ({
      ...item,
      // expensive transformation
      processedValue: expensiveCalculation(item.value)
    }));
  }, [data]);
  
  return (
    <div>
      {processedData.map(item => (
        <div key={item.id} onClick={() => onItemSelect(item.id)}>
          {item.name}: {item.processedValue}
        </div>
      ))}
    </div>
  );
}

// Prevent re-renders when props haven't changed
export default memo(ExpensiveComponent);
```

### Event Handlers

Optimize event handlers with useCallback:

```typescript
"use client";

import { useState, useCallback } from 'react';

export default function ProductList({ products }) {
  const [selectedId, setSelectedId] = useState(null);
  
  // Memoize callback
  const handleSelect = useCallback((id: string) => {
    setSelectedId(id);
    // other operations
  }, []);
  
  return (
    <div>
      {products.map(product => (
        <ProductCard
          key={product.id}
          product={product}
          isSelected={product.id === selectedId}
          onSelect={handleSelect}
        />
      ))}
    </div>
  );
}
```

## Image Optimization

Use Next.js Image component:

```typescript
import Image from 'next/image';

export default function ProductImage({ product }) {
  return (
    <div className="relative h-64 w-full">
      <Image
        src={product.imageUrl}
        alt={product.name}
        fill
        sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
        placeholder="blur"
        blurDataURL={product.blurUrl}
        priority={product.isFeatured}
        className="object-cover"
      />
    </div>
  );
}
```

## Font Optimization

Use Next.js font optimization:

```typescript
// app/layout.tsx
import { Inter, Roboto_Mono } from 'next/font/google';

// Load fonts
const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-inter',
});

const robotoMono = Roboto_Mono({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-roboto-mono',
});

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en" className={`${inter.variable} ${robotoMono.variable}`}>
      <body>{children}</body>
    </html>
  );
}
```

## Code Splitting and Lazy Loading

### Component Lazy Loading

Lazy load components that aren't immediately needed:

```typescript
// app/page.tsx
import { Suspense, lazy } from 'react';
import LoadingSpinner from '@/components/ui/LoadingSpinner';

// Lazy load heavy components
const HeavyChart = lazy(() => import('@/components/dashboard/HeavyChart'));
const DataTable = lazy(() => import('@/components/DataTable'));

export default function DashboardPage() {
  return (
    <div>
      <h1>Dashboard</h1>
      
      {/* Critical content */}
      <DashboardSummary />
      
      {/* Lazy loaded content */}
      <Suspense fallback={<LoadingSpinner />}>
        <HeavyChart />
      </Suspense>
      
      <Suspense fallback={<p>Loading data table...</p>}>
        <DataTable />
      </Suspense>
    </div>
  );
}
```

### Dynamic Imports

Use dynamic imports for client components:

```typescript
"use client";

import { useState } from 'react';
import dynamic from 'next/dynamic';

// Dynamically import modal
const ProductModal = dynamic(() => import('@/components/products/ProductModal'), {
  loading: () => <p>Loading...</p>,
});

export default function ProductPage({ product }) {
  const [isModalOpen, setIsModalOpen] = useState(false);
  
  return (
    <div>
      <h1>{product.name}</h1>
      <button onClick={() => setIsModalOpen(true)}>View Details</button>
      
      {/* Only loaded when modal is opened */}
      {isModalOpen && (
        <ProductModal
          product={product}
          onClose={() => setIsModalOpen(false)}
        />
      )}
    </div>
  );
}
```

## Streaming and Progressive Rendering

Enable streaming with Suspense:

```typescript
// app/products/page.tsx
import { Suspense } from 'react';
import ProductGrid from '@/components/products/ProductGrid';
import ProductFilterSidebar from '@/components/products/ProductFilterSidebar';
import ProductsLoading from './loading';

export default function ProductsPage() {
  return (
    <div className="grid grid-cols-4 gap-6">
      <ProductFilterSidebar />
      
      <div className="col-span-3">
        <Suspense fallback={<ProductsLoading />}>
          {/* This component can fetch its own data */}
          <ProductGrid />
        </Suspense>
      </div>
    </div>
  );
}
```

## Bundle Optimization

### Analyze Bundles

Use `@next/bundle-analyzer` to identify large dependencies:

```bash
# Install the package
npm install --save-dev @next/bundle-analyzer

# In next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

module.exports = withBundleAnalyzer({
  // your Next.js config
});

# Run analysis
ANALYZE=true npm run build
```

### Tree-Shaking

Ensure proper imports for tree-shaking:

```typescript
// ❌ Avoid - imports entire library
import lodash from 'lodash';

// ✅ Better - imports only what's needed
import map from 'lodash/map';
import debounce from 'lodash/debounce';
```

## Performance Checklist

- [ ] Use Server Components for non-interactive parts
- [ ] Implement appropriate caching strategies
- [ ] Optimize images with next/image
- [ ] Lazy load non-critical components
- [ ] Memoize expensive calculations with useMemo
- [ ] Stabilize callbacks with useCallback
- [ ] Use Suspense for streaming and progressive loading
- [ ] Track and optimize Core Web Vitals
- [ ] Analyze bundle size and optimize large dependencies
- [ ] Enable HTTP/2 or HTTP/3 for your hosting environment
