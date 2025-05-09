# Server Components

Next.js 13+ introduced React Server Components, which fundamentally change how we build React applications. This guide outlines our approach to using Server Components effectively.

## Server Components vs Client Components

### Server Components (Default)

Server Components:

- Run only on the server and never on the client
- Have zero impact on JavaScript bundle size
- Can directly access backend resources
- Are not interactive (no event handlers)

```typescript
// app/users/page.tsx
// This is a Server Component by default
export default async function UsersPage() {
  // Fetch directly from the database or API
  const users = await fetchUsers();
  
  return (
    <div>
      <h1>Users</h1>
      <UserList users={users} />
    </div>
  );
}
```

### Client Components

Client Components:

- Run on both server (for initial render) and client
- Are marked with the "use client" directive
- Support interactivity and state
- Can use browser APIs and hooks

```typescript
"use client";

// components/interactive/Counter.tsx
import { useState } from 'react';

export default function Counter() {
  const [count, setState] = useState(0);
  
  return (
    <button onClick={() => setState(count + 1)}>
      Count: {count}
    </button>
  );
}
```

## Our Approach: Server-First

1. **Start with Server Components**: Begin building with Server Components by default
2. **Move to Client Components as needed**: Only convert to Client Components when you need:
   - Interactivity (event handlers, useState, useEffect)
   - Browser APIs
   - Custom hooks that use state/effects

3. **Create component boundaries**: When you need a Client Component, create a clear boundary:

```typescript
// UserProfile.tsx (Server Component)
import UserActions from './UserActions';

export default async function UserProfile({ userId }: { userId: string }) {
  // Server-side data fetching
  const user = await fetchUserDetails(userId);
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
      
      {/* Pass server data to Client Component */}
      <UserActions user={user} />
    </div>
  );
}
```

```typescript
// UserActions.tsx (Client Component)
"use client";

import { useState } from 'react';
import type { User } from '@/types';

export default function UserActions({ user }: { user: User }) {
  const [isEditing, setIsEditing] = useState(false);
  
  return (
    <div>
      <button onClick={() => setIsEditing(!isEditing)}>
        {isEditing ? 'Cancel' : 'Edit Profile'}
      </button>
      
      {isEditing && <EditForm initialData={user} />}
    </div>
  );
}
```

## Best Practices

1. **Fetch Data in Server Components**: Take advantage of server components to fetch data directly
2. **Pass Data as Props**: Send data from server to client components as props
3. **Keep Client Components Lean**: Client components should focus primarily on interactivity
4. **Hoist Client Directives**: Move "use client" to the highest reasonable level to minimize client components
5. **Async/Await in Server Components**: Utilize async/await pattern for data fetching in Server Components

## Common Pitfalls

1. ❌ Using "use client" unnecessarily (default to server components)
2. ❌ Fetching data in client components when it could be done on the server
3. ❌ Trying to use useState or useEffect in Server Components
4. ❌ Importing Server Components into Client Components (not supported)

## Performance Benefits

- Reduced JavaScript bundle size
- Faster initial page load
- Improved SEO
- Better performance on low-end devices
