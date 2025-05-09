# Frontend Handbook

This repository serves as the comprehensive frontend handbook for our organization. It outlines our best practices, coding standards, and architectural decisions for frontend development.

## Purpose

- Establish consistent coding patterns and practices across all frontend projects
- Onboard new developers quickly with clear guidelines
- Document architectural decisions and their rationales
- Provide practical examples of implementation patterns

## Tech Stack

- **Framework**: Next.js (App Router)
- **Language**: TypeScript
- **Data Fetching**:
  - Server Components: Native fetch with Next.js caching
  - Client Components: React Query / SWR with server-fetched initial data
- **Styling**: Tailwind CSS with shadcn/ui components and Framer Motion animations
- **State Management**: React Context + React Query/SWR
- **Authentication**: NextAuth.js with middleware for route protection

## Core Principles

1. **SSR-First Approach**: Optimize for server-side rendering to improve SEO and initial load performance
2. **Server Components by Default**: Prefer server components over client components when possible
3. **TypeScript Everywhere**: Strong typing for better developer experience and code quality
4. **Data Fetching on the Server**: Fetch data on the server and pass as props to child components
5. **Component-Based Architecture**: Build UIs from small, reusable components
6. **Consistent Error Handling**: Create proper error.tsx fallbacks for all routes
7. **Loading State Management**: Provide loading.tsx files for all routes with data fetching
8. **Route Protection**: Use middleware for authentication and authorization

## Naming Conventions

- **Component Names**: PascalCase (e.g., `ProductCard`)
- **File and Directory Names**: kebab-case (e.g., `product-card.tsx`)
- **CSS Classes**: Use Tailwind utility classes directly

## How to Use This Handbook

Each section in this handbook contains:

- Explanations of best practices
- Code examples
- Common pitfalls to avoid
- Links to relevant documentation

Explore the specific sections in the `/docs` directory to learn more about each topic:

1. [Project Structure](./docs/01-project-structure.md)
2. [Server Components](./docs/02-server-components.md)
3. [Data Fetching](./docs/03-data-fetching.md)
4. [TypeScript Best Practices](./docs/04-typescript-best-practices.md)
5. [Component Patterns](./docs/05-component-patterns.md)
6. [Performance Optimization](./docs/06-performance-optimization.md)
7. [State Management](./docs/07-state-management.md)
8. [Styling Guidelines](./docs/08-styling-guidelines.md)
9. [Loading States and Error Handling](./docs/09-loading-error-handling.md)
10. [Authentication with NextAuth.js](./docs/10-authentication.md)
11. [Middleware](./docs/11-middleware.md)
