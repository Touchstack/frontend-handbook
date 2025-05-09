# Next.js Project Structure

This guide outlines our recommended project structure for Next.js applications using the App Router.

## Directory Structure

```
my-app/
├── app/                  # App Router directory
│   ├── api/              # API routes
│   ├── (auth)/           # Route group for authentication-related pages
│   ├── (dashboard)/      # Route group for dashboard-related pages
│   ├── layout.tsx        # Root layout
│   └── page.tsx          # Homepage
├── components/           # Shared components
│   ├── ui/               # UI components (buttons, forms, etc.)
│   └── layouts/          # Layout components
├── lib/                  # Utility functions and shared logic
│   ├── api/              # API client utilities
│   ├── hooks/            # Custom React hooks
│   ├── utils/            # Helper functions
│   └── validations/      # Form and data validation schemas
├── public/               # Static assets
├── styles/               # Global styles
└── types/                # TypeScript type definitions
```

## Key Conventions

### App Router Organization

- Use **route groups** (parentheses folders) to organize related routes
- Keep related components close to their routes
- Place shared layouts in the appropriate level

```typescript
// app/(dashboard)/layout.tsx
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="dashboard-layout">
      <DashboardSidebar />
      <main>{children}</main>
    </div>
  );
}
```

### Component Organization

- Place components in logical categories
- Co-locate related components, hooks, and utilities
- Component files should use PascalCase naming

### Data Fetching

- Create server actions for data mutations in `/app/actions/`
- Place API utilities in `/lib/api/`

## Best Practices

1. **Route Organization**: Group routes logically with route groups
2. **Parallel Routes**: Use for complex layouts with independent navigation states
3. **Loading States**: Implement loading.tsx files for each route segment that fetches data
4. **Error Handling**: Add error.tsx files for graceful error handling

## Anti-Patterns to Avoid

1. ❌ Mixing old pages router and app router code
2. ❌ Deeply nested component directories without logical organization
3. ❌ Placing business logic directly in page or layout components
4. ❌ Storing sensitive information in client-side code
