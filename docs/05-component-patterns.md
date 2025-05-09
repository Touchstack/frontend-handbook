# Component Patterns

This guide outlines our standard component patterns and best practices for Next.js applications.

## Component Structure

### Folder Organization

Organize components into meaningful categories:

```
components/
├── ui/                # UI components (buttons, forms, inputs)
│   ├── Button.tsx
│   ├── Input.tsx
│   └── Form.tsx
├── layout/            # Layout components
│   ├── Header.tsx
│   ├── Footer.tsx
│   └── Sidebar.tsx
├── features/          # Feature-specific components
│   ├── auth/
│   ├── products/
│   └── dashboard/
└── shared/            # Shared components used across features
    ├── ErrorBoundary.tsx
    └── LoadingState.tsx
```

### Component File Structure

Each component should follow a consistent structure:

```typescript
// components/ui/Button.tsx

// 1. Imports
import { forwardRef } from 'react';
import { cn } from '@/lib/utils';

// 2. Types
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'outline' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  isLoading?: boolean;
}

// 3. Component Definition
const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant = 'primary', size = 'md', isLoading, children, ...props }, ref) => {
    // 4. Styles and classNames
    const baseStyles = 'rounded font-medium transition-colors focus:outline-none';
    
    const variantStyles = {
      primary: 'bg-blue-600 text-white hover:bg-blue-700',
      secondary: 'bg-gray-200 text-gray-800 hover:bg-gray-300',
      outline: 'border border-gray-300 bg-transparent hover:bg-gray-100',
      ghost: 'bg-transparent hover:bg-gray-100',
    };
    
    const sizeStyles = {
      sm: 'px-3 py-1.5 text-sm',
      md: 'px-4 py-2',
      lg: 'px-6 py-3 text-lg',
    };
    
    // 5. Component Logic
    return (
      <button
        ref={ref}
        className={cn(
          baseStyles,
          variantStyles[variant],
          sizeStyles[size],
          isLoading && 'opacity-70 cursor-not-allowed',
          className
        )}
        disabled={isLoading}
        {...props}
      >
        {isLoading ? (
          <span className="flex items-center justify-center">
            <LoadingSpinner size="sm" />
            <span className="ml-2">{children}</span>
          </span>
        ) : (
          children
        )}
      </button>
    );
  }
);

// 6. Display Name
Button.displayName = 'Button';

// 7. Export
export { Button };
```

## Component Design Patterns

### Composition Over Inheritance

Use component composition to create complex UIs:

```typescript
// Example of composition
export function Card({ children }: { children: React.ReactNode }) {
  return <div className="rounded-lg border shadow p-4">{children}</div>;
}

Card.Title = function CardTitle({ children }: { children: React.ReactNode }) {
  return <h3 className="text-lg font-semibold mb-2">{children}</h3>;
};

Card.Content = function CardContent({ children }: { children: React.ReactNode }) {
  return <div className="text-gray-600">{children}</div>;
};

Card.Footer = function CardFooter({ children }: { children: React.ReactNode }) {
  return <div className="mt-4 pt-2 border-t flex justify-end">{children}</div>;
};

// Usage
function ProductCard({ product }) {
  return (
    <Card>
      <Card.Title>{product.name}</Card.Title>
      <Card.Content>
        <p>{product.description}</p>
        <p className="font-bold">${product.price}</p>
      </Card.Content>
      <Card.Footer>
        <Button>Add to Cart</Button>
      </Card.Footer>
    </Card>
  );
}
```

### Props Spreading & Forwarding

Use prop spreading for extensibility and ref forwarding for DOM access:

```typescript
// Using forwardRef and spreading props
import { forwardRef } from 'react';

interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label: string;
  error?: string;
}

const Input = forwardRef<HTMLInputElement, InputProps>(
  ({ label, error, className, ...props }, ref) => {
    return (
      <div className="mb-4">
        <label className="block mb-1 font-medium">{label}</label>
        <input
          ref={ref}
          className={`w-full border rounded px-3 py-2 ${
            error ? 'border-red-500' : 'border-gray-300'
          } ${className}`}
          {...props}
        />
        {error && <p className="text-red-500 text-sm mt-1">{error}</p>}
      </div>
    );
  }
);

Input.displayName = 'Input';

export { Input };
```

### Controlled vs Uncontrolled Components

Support both patterns:

```typescript
// Supporting both controlled and uncontrolled
import { forwardRef, useState, useEffect } from 'react';

interface ToggleProps {
  checked?: boolean;
  defaultChecked?: boolean;
  onChange?: (checked: boolean) => void;
}

const Toggle = forwardRef<HTMLInputElement, ToggleProps>(
  ({ checked, defaultChecked, onChange }, ref) => {
    // For uncontrolled mode
    const [internalChecked, setInternalChecked] = useState(defaultChecked || false);
    
    // Sync with controlled value
    useEffect(() => {
      if (checked !== undefined) {
        setInternalChecked(checked);
      }
    }, [checked]);
    
    // Determine if component is controlled
    const isControlled = checked !== undefined;
    const isChecked = isControlled ? checked : internalChecked;
    
    const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
      const newChecked = e.target.checked;
      
      // Always update internal state for uncontrolled mode
      if (!isControlled) {
        setInternalChecked(newChecked);
      }
      
      // Call onChange if provided
      onChange?.(newChecked);
    };
    
    return (
      <input
        ref={ref}
        type="checkbox"
        checked={isChecked}
        onChange={handleChange}
      />
    );
  }
);

Toggle.displayName = 'Toggle';

export { Toggle };
```

## Server Components vs Client Components

### Server Components

Keep server components simple and focused on data fetching and rendering:

```typescript
// app/products/[id]/page.tsx (Server Component)
import { ProductDetails } from '@/components/products/ProductDetails';
import { ProductRecommendations } from '@/components/products/ProductRecommendations';
import { fetchProduct, fetchRecommendations } from '@/lib/api';

export default async function ProductPage({ 
  params 
}: { 
  params: { id: string } 
}) {
  // Parallel data fetching
  const [product, recommendations] = await Promise.all([
    fetchProduct(params.id),
    fetchRecommendations(params.id)
  ]);
  
  return (
    <div className="product-page">
      <ProductDetails product={product} />
      <ProductRecommendations 
        recommendations={recommendations} 
        currentProductId={params.id} 
      />
    </div>
  );
}
```

### Client Components

Use client components for interactivity:

```typescript
// components/products/AddToCartButton.tsx (Client Component)
"use client";

import { useState } from 'react';
import { Button } from '@/components/ui/Button';
import { addToCart } from '@/lib/api/cart';

interface AddToCartButtonProps {
  productId: string;
  initialStock: number;
}

export function AddToCartButton({ productId, initialStock }: AddToCartButtonProps) {
  const [isLoading, setIsLoading] = useState(false);
  const [quantity, setQuantity] = useState(1);
  
  const handleAddToCart = async () => {
    setIsLoading(true);
    try {
      await addToCart(productId, quantity);
      // Show success toast or feedback
    } catch (error) {
      // Handle error
    } finally {
      setIsLoading(false);
    }
  };
  
  return (
    <div>
      <div className="flex items-center mb-4">
        <button 
          onClick={() => setQuantity(q => Math.max(1, q - 1))}
          className="px-3 py-1 border"
        >
          -
        </button>
        <span className="mx-4">{quantity}</span>
        <button 
          onClick={() => setQuantity(q => Math.min(initialStock, q + 1))}
          className="px-3 py-1 border"
        >
          +
        </button>
      </div>
      <Button 
        onClick={handleAddToCart} 
        isLoading={isLoading}
        disabled={initialStock === 0}
      >
        {initialStock > 0 ? 'Add to Cart' : 'Out of Stock'}
      </Button>
    </div>
  );
}
```

## Best Practices

1. **Keep components focused**: Single responsibility principle
2. **Use composition**: Build complex UIs from simple components
3. **Colocation**: Keep related components together
4. **Custom hooks**: Extract complex logic to hooks
5. **Consistent naming**: Use PascalCase for components
6. **Forward refs**: Use forwardRef for components that wrap DOM elements
7. **Test your components**: Add unit/integration tests for critical components

## Component Checklist

- [ ] Uses TypeScript for type safety
- [ ] Has proper prop types and defaults
- [ ] Forwards refs when wrapping DOM elements
- [ ] Uses composition over inheritance
- [ ] Is properly documented (usage, props)
- [ ] Has appropriate error handling
- [ ] Labeled with "use client" directive if needed
- [ ] Follows accessibility best practices
