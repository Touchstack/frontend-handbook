# Authentication with NextAuth.js

This guide outlines our approach to authentication in Next.js applications using NextAuth.js.

## Overview

We use NextAuth.js (now known as Auth.js) as our authentication solution for its flexibility, security, and easy integration with Next.js.

## Installation

```bash
npm install next-auth
```

## Basic Setup

### Auth Configuration

Create the auth configuration file:

```typescript
// app/api/auth/[...nextauth]/route.ts
import NextAuth from "next-auth";
import { authOptions } from "@/lib/auth";

const handler = NextAuth(authOptions);

export { handler as GET, handler as POST };
```

Define your auth options in a separate file for better organization:

```typescript
// lib/auth.ts
import { NextAuthOptions } from "next-auth";
import CredentialsProvider from "next-auth/providers/credentials";
import GoogleProvider from "next-auth/providers/google";
import { PrismaAdapter } from "@auth/prisma-adapter";
import { db } from "@/lib/db";
import { compare } from "bcrypt";

export const authOptions: NextAuthOptions = {
  adapter: PrismaAdapter(db),
  session: {
    strategy: "jwt",
  },
  pages: {
    signIn: "/login",
    signOut: "/logout",
    error: "/auth/error",
  },
  providers: [
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    }),
    CredentialsProvider({
      name: "Credentials",
      credentials: {
        email: { label: "Email", type: "email" },
        password: { label: "Password", type: "password" },
      },
      async authorize(credentials) {
        if (!credentials?.email || !credentials?.password) {
          return null;
        }

        const user = await db.user.findUnique({
          where: { email: credentials.email },
        });

        if (!user) {
          return null;
        }

        const isPasswordValid = await compare(
          credentials.password,
          user.password
        );

        if (!isPasswordValid) {
          return null;
        }

        return {
          id: user.id,
          email: user.email,
          name: user.name,
          role: user.role,
        };
      },
    }),
  ],
  callbacks: {
    async session({ session, token }) {
      if (token) {
        session.user.id = token.id as string;
        session.user.role = token.role as string;
      }
      return session;
    },
    async jwt({ token, user }) {
      if (user) {
        token.id = user.id;
        token.role = user.role;
      }
      return token;
    },
  },
};
```

### Environment Variables

Add the required environment variables to your `.env.local` file:

```bash
# NextAuth
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=your-generated-secret-key

# OAuth Providers
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret
```

Generate a secure NextAuth secret with:

```bash
openssl rand -base64 32
```

## Client Components

### Session Provider

Wrap your application with the `SessionProvider` in your root layout:

```tsx
// app/providers.tsx
"use client";

import { SessionProvider } from "next-auth/react";

interface AuthProviderProps {
  children: React.ReactNode;
}

export function AuthProvider({ children }: AuthProviderProps) {
  return <SessionProvider>{children}</SessionProvider>;
}
```

```tsx
// app/layout.tsx
import { AuthProvider } from "./providers";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <AuthProvider>{children}</AuthProvider>
      </body>
    </html>
  );
}
```

### Authentication Hooks

Use the NextAuth hooks in client components:

```tsx
// components/auth/login-form.tsx
"use client";

import { useState } from "react";
import { signIn } from "next-auth/react";
import { useRouter } from "next/navigation";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { toast } from "@/components/ui/use-toast";

export function LoginForm() {
  const router = useRouter();
  const [isLoading, setIsLoading] = useState(false);

  async function onSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    setIsLoading(true);

    const formData = new FormData(e.currentTarget);
    const email = formData.get("email") as string;
    const password = formData.get("password") as string;

    try {
      const result = await signIn("credentials", {
        email,
        password,
        redirect: false,
      });

      if (result?.error) {
        toast({
          title: "Login failed",
          description: "Invalid email or password",
          variant: "destructive",
        });
        return;
      }

      router.refresh();
      router.push("/dashboard");
    } catch (error) {
      toast({
        title: "Something went wrong",
        description: "Please try again later",
        variant: "destructive",
      });
    } finally {
      setIsLoading(false);
    }
  }

  return (
    <form onSubmit={onSubmit} className="space-y-4">
      <div className="space-y-2">
        <Label htmlFor="email">Email</Label>
        <Input id="email" name="email" type="email" required />
      </div>
      <div className="space-y-2">
        <Label htmlFor="password">Password</Label>
        <Input id="password" name="password" type="password" required />
      </div>
      <Button type="submit" className="w-full" disabled={isLoading}>
        {isLoading ? "Signing in..." : "Sign in"}
      </Button>
    </form>
  );
}
```

## Server Components

### Authentication Status

Access session data in server components:

```tsx
// app/dashboard/page.tsx
import { getServerSession } from "next-auth/next";
import { redirect } from "next/navigation";
import { authOptions } from "@/lib/auth";
import { DashboardHeader } from "@/components/dashboard/dashboard-header";
import { DashboardContent } from "@/components/dashboard/dashboard-content";

export default async function DashboardPage() {
  const session = await getServerSession(authOptions);

  if (!session) {
    redirect("/login?callbackUrl=/dashboard");
  }

  return (
    <div className="container mx-auto py-10">
      <DashboardHeader user={session.user} />
      <DashboardContent userId={session.user.id} />
    </div>
  );
}
```

## Route Protection

### Middleware

Create a middleware file to protect routes:

```typescript
// middleware.ts
import { NextResponse } from "next/server";
import { getToken } from "next-auth/jwt";
import { NextRequest } from "next/server";

export async function middleware(request: NextRequest) {
  const path = request.nextUrl.pathname;
  
  // Define public paths that don't require authentication
  const publicPaths = ["/login", "/register", "/", "/about"];
  const isPublicPath = publicPaths.some(publicPath => 
    path === publicPath || path.startsWith("/api/auth")
  );

  const token = await getToken({
    req: request,
    secret: process.env.NEXTAUTH_SECRET,
  });

  // Redirect unauthenticated users to login
  if (!token && !isPublicPath) {
    const url = new URL("/login", request.url);
    url.searchParams.set("callbackUrl", path);
    return NextResponse.redirect(url);
  }

  // Redirect authenticated users away from login/register pages
  if (token && (path === "/login" || path === "/register")) {
    return NextResponse.redirect(new URL("/dashboard", request.url));
  }

  return NextResponse.next();
}

// Specify which routes the middleware should run on
export const config = {
  matcher: [
    /*
     * Match all paths except:
     * 1. /api/auth (NextAuth routes)
     * 2. /_next (Next.js internals)
     * 3. /fonts, /images (static files)
     * 4. /favicon.ico, /robots.txt (static files)
     */
    "/((?!api/auth|_next|fonts|images|favicon.ico|robots.txt).*)",
  ],
};
```

## Role-Based Access Control

### Define User Roles

Extend your User type to include roles:

```typescript
// types/next-auth.d.ts
import { DefaultSession } from "next-auth";

declare module "next-auth" {
  interface Session {
    user: {
      id: string;
      role: string;
    } & DefaultSession["user"];
  }

  interface User {
    role: string;
  }
}

declare module "next-auth/jwt" {
  interface JWT {
    id: string;
    role: string;
  }
}
```

### RBAC Component

Create a component to handle role-based access:

```tsx
// components/auth/require-role.tsx
"use client";

import { useSession } from "next-auth/react";
import { redirect } from "next/navigation";

interface RequireRoleProps {
  children: React.ReactNode;
  allowedRoles: string[];
  fallback?: React.ReactNode;
}

export function RequireRole({ 
  children, 
  allowedRoles, 
  fallback 
}: RequireRoleProps) {
  const { data: session, status } = useSession();
  
  if (status === "loading") {
    return <div>Loading...</div>;
  }
  
  if (status === "unauthenticated") {
    redirect("/login");
    return null;
  }
  
  const userRole = session?.user.role;
  
  if (!userRole || !allowedRoles.includes(userRole)) {
    if (fallback) {
      return <>{fallback}</>;
    }
    
    return (
      <div className="flex flex-col items-center justify-center min-h-[50vh]">
        <h2 className="text-2xl font-bold">Access Denied</h2>
        <p className="text-muted-foreground">
          You don't have permission to access this page.
        </p>
      </div>
    );
  }
  
  return <>{children}</>;
}
```

### Usage in Pages

```tsx
// app/admin/page.tsx
import { RequireRole } from "@/components/auth/require-role";
import { AdminPanel } from "@/components/admin/admin-panel";

export default function AdminPage() {
  return (
    <RequireRole allowedRoles={["admin"]}>
      <div className="container mx-auto py-10">
        <h1 className="text-3xl font-bold mb-6">Admin Dashboard</h1>
        <AdminPanel />
      </div>
    </RequireRole>
  );
}
```

## Custom Login Page

Create a custom login page:

```tsx
// app/login/page.tsx
import { LoginForm } from "@/components/auth/login-form";
import { SocialLogin } from "@/components/auth/social-login";

export default function LoginPage({
  searchParams,
}: {
  searchParams: { callbackUrl?: string };
}) {
  return (
    <div className="container flex h-screen w-screen flex-col items-center justify-center">
      <div className="mx-auto flex w-full flex-col justify-center space-y-6 sm:w-[350px]">
        <div className="flex flex-col space-y-2 text-center">
          <h1 className="text-2xl font-semibold tracking-tight">
            Sign in to your account
          </h1>
          <p className="text-sm text-muted-foreground">
            Enter your email and password to continue
          </p>
        </div>
        <LoginForm callbackUrl={searchParams.callbackUrl} />
        <div className="relative">
          <div className="absolute inset-0 flex items-center">
            <span className="w-full border-t" />
          </div>
          <div className="relative flex justify-center text-xs uppercase">
            <span className="bg-background px-2 text-muted-foreground">
              Or continue with
            </span>
          </div>
        </div>
        <SocialLogin />
      </div>
    </div>
  );
}
```

## Authentication Best Practices

1. **Secure Secrets**: Store authentication secrets in environment variables
2. **HTTPS Only**: Enforce HTTPS in production
3. **Short Sessions**: Keep JWT sessions reasonable in length (default is 30 days)
4. **CSRF Protection**: NextAuth includes CSRF protection by default
5. **Database Adapter**: Use a database adapter for persistent sessions
6. **Custom Pages**: Create custom authentication pages for better UX
7. **Error Handling**: Provide clear error messages for authentication failures
8. **Audit Logs**: Log authentication events for security monitoring
9. **Password Standards**: Enforce strong password requirements
10. **Rate Limiting**: Implement rate limiting for login attempts

## Authentication Checklist

- [ ] Set up NextAuth.js with appropriate providers
- [ ] Configure secure environment variables
- [ ] Create custom login/register pages
- [ ] Implement route protection with middleware
- [ ] Set up role-based access control
- [ ] Add session validation in server components
- [ ] Create auth-related UI components (login form, social login buttons)
- [ ] Implement proper error handling for auth failures
- [ ] Test authentication flows thoroughly
- [ ] Set up proper session expiration and refresh logic
