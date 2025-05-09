# Loading States and Error Handling

This guide outlines our approach to loading states and error handling in Next.js applications.

## Loading States

We create dedicated loading states using Next.js App Router's loading.tsx files for all pages with data fetching.

### Loading Convention

Each route segment that performs data fetching should have a corresponding `loading.tsx` file:

```
app/
├── dashboard/
│   ├── page.tsx         # Main dashboard page with data fetching
│   └── loading.tsx      # Loading UI for dashboard
├── products/
│   ├── page.tsx         # Products listing page
│   ├── loading.tsx      # Loading UI for products listing
│   └── [id]/
│       ├── page.tsx     # Product detail page
│       └── loading.tsx  # Loading UI for product detail
```

### Loading UI Implementation

Create skeleton loaders that match the shape of your actual content:

```tsx
// app/products/loading.tsx
import { Skeleton } from "@/components/ui/skeleton";

export default function ProductsLoading() {
  return (
    <div className="container mx-auto py-10">
      <Skeleton className="h-10 w-[250px] mb-6" />
      
      <div className="grid grid-cols-1 gap-6 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4">
        {Array.from({ length: 8 }).map((_, index) => (
          <div key={index} className="rounded-lg border p-4">
            <Skeleton className="h-40 w-full rounded-md" />
            <Skeleton className="h-6 w-3/4 mt-4" />
            <Skeleton className="h-4 w-1/2 mt-2" />
            <div className="flex justify-between mt-4">
              <Skeleton className="h-8 w-20" />
              <Skeleton className="h-8 w-8 rounded-full" />
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}
```

### Loading UI Best Practices

1. **Match content structure**: Create skeleton loaders that match the layout of your actual content
2. **Show progress**: For longer operations, consider progress indicators
3. **Keep it simple**: Don't overload loading states with unnecessary animations
4. **Consistent sizing**: Ensure skeletons match the approximate size of the content they replace

### Creating Skeleton Components

Create reusable skeleton components:

```tsx
// components/ui/skeleton.tsx
import { cn } from "@/lib/utils";

interface SkeletonProps {
  className?: string;
}

export function Skeleton({ className }: SkeletonProps) {
  return (
    <div
      className={cn(
        "animate-pulse rounded-md bg-muted",
        className
      )}
    />
  );
}
```

## Error Handling

We implement error.tsx files for all routes to provide graceful error states.

### Error Convention

Each route segment should have a corresponding `error.tsx` file:

```
app/
├── dashboard/
│   ├── page.tsx         # Main dashboard page
│   ├── loading.tsx      # Loading UI
│   └── error.tsx        # Error UI for dashboard
├── products/
│   ├── page.tsx         # Products listing page
│   ├── loading.tsx      # Loading UI
│   ├── error.tsx        # Error UI for products listing
│   └── [id]/
│       ├── page.tsx     # Product detail page
│       ├── loading.tsx  # Loading UI
│       └── error.tsx    # Error UI for product detail
```

### Error UI Implementation

Create client components that handle different error scenarios:

```tsx
// app/products/[id]/error.tsx
"use client";

import { useEffect } from "react";
import { Button } from "@/components/ui/button";
import { AlertCircle } from "lucide-react";

interface ErrorProps {
  error: Error & { digest?: string };
  reset: () => void;
}

export default function ProductError({ error, reset }: ErrorProps) {
  useEffect(() => {
    // Log the error to an error reporting service
    console.error(error);
  }, [error]);

  return (
    <div className="container mx-auto py-10 flex flex-col items-center justify-center space-y-4 text-center">
      <div className="flex h-20 w-20 items-center justify-center rounded-full bg-destructive/10">
        <AlertCircle className="h-10 w-10 text-destructive" />
      </div>
      <h2 className="text-3xl font-bold tracking-tight">Something went wrong!</h2>
      <p className="text-muted-foreground max-w-[500px]">
        We couldn't load the product information. Please try again later.
      </p>
      <div className="flex gap-2">
        <Button onClick={() => reset()}>Try again</Button>
        <Button variant="outline" onClick={() => window.history.back()}>
          Go back
        </Button>
      </div>
      <p className="text-sm text-muted-foreground">
        Error reference: {error.digest}
      </p>
    </div>
  );
}
```

### Global Error Handler

Implement a global error handler at the root level:

```tsx
// app/global-error.tsx
"use client";

import { Button } from "@/components/ui/button";

export default function GlobalError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <html>
      <body>
        <div className="flex h-screen flex-col items-center justify-center gap-6">
          <h1 className="text-4xl font-bold">Something went wrong!</h1>
          <p className="text-muted-foreground max-w-[500px] text-center">
            We apologize for the inconvenience. Our team has been notified about this issue.
          </p>
          <Button onClick={() => reset()}>Try again</Button>
          <p className="text-sm text-muted-foreground">
            Error reference: {error.digest}
          </p>
        </div>
      </body>
    </html>
  );
}
```

### Error Handling Best Practices

1. **Be specific**: Tailor error messages to the specific context
2. **Provide actions**: Give users clear next steps (reset, go back, contact support)
3. **Log errors**: Capture error details for debugging
4. **Include error codes**: Add error references to help with debugging
5. **Maintain brand consistency**: Style error pages to match your app's design

### Not Found Pages

Create a custom 404 page for routes that don't exist:

```tsx
// app/not-found.tsx
import Link from "next/link";
import { Button } from "@/components/ui/button";

export default function NotFound() {
  return (
    <div className="container mx-auto py-20 flex flex-col items-center justify-center space-y-6 text-center">
      <h1 className="text-9xl font-bold">404</h1>
      <h2 className="text-3xl font-bold tracking-tight">Page not found</h2>
      <p className="text-muted-foreground max-w-[500px]">
        Sorry, we couldn't find the page you're looking for.
      </p>
      <div className="flex gap-2">
        <Button asChild>
          <Link href="/">Go home</Link>
        </Button>
        <Button variant="outline" onClick={() => window.history.back()}>
          Go back
        </Button>
      </div>
    </div>
  );
}
```

Also create specific not-found pages for dynamic routes:

```tsx
// app/products/[id]/not-found.tsx
import Link from "next/link";
import { Button } from "@/components/ui/button";

export default function ProductNotFound() {
  return (
    <div className="container mx-auto py-10 flex flex-col items-center justify-center space-y-4 text-center">
      <h2 className="text-3xl font-bold tracking-tight">Product not found</h2>
      <p className="text-muted-foreground max-w-[500px]">
        The product you're looking for doesn't exist or has been removed.
      </p>
      <div className="flex gap-2">
        <Button asChild>
          <Link href="/products">Browse products</Link>
        </Button>
        <Button variant="outline" onClick={() => window.history.back()}>
          Go back
        </Button>
      </div>
    </div>
  );
}
```

## Suspense for Component-Level Loading

For more granular loading states, use React Suspense:

```tsx
// app/dashboard/page.tsx
import { Suspense } from "react";
import DashboardStats from "@/components/dashboard/dashboard-stats";
import RecentActivity from "@/components/dashboard/recent-activity";
import { DashboardStatsLoading } from "@/components/dashboard/dashboard-stats-loading";
import { RecentActivityLoading } from "@/components/dashboard/recent-activity-loading";

export default function DashboardPage() {
  return (
    <div className="container mx-auto py-10 space-y-8">
      <h1 className="text-3xl font-bold">Dashboard</h1>
      
      <div className="grid gap-6 md:grid-cols-2">
        <Suspense fallback={<DashboardStatsLoading />}>
          <DashboardStats />
        </Suspense>
        
        <Suspense fallback={<RecentActivityLoading />}>
          <RecentActivity />
        </Suspense>
      </div>
    </div>
  );
}
```

## Loading and Error UI Checklist

- [ ] Create a `loading.tsx` file for each route with data fetching
- [ ] Create an `error.tsx` file for each route for error handling
- [ ] Implement a global error handler
- [ ] Create custom 404 pages for not-found routes
- [ ] Use Suspense for component-level loading states
- [ ] Ensure loading UI matches the structure of actual content
- [ ] Provide clear error messages and recovery actions
- [ ] Log errors for debugging
- [ ] Maintain consistent styling across loading and error states
