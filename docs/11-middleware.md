# Middleware

This guide outlines our approach to using middleware in Next.js applications for route protection, redirects, and more.

## Overview

Next.js middleware allows you to run code before a request is completed, making it ideal for:

- Authentication & authorization
- Redirects & rewrites
- Request/response manipulation
- Feature flags
- A/B testing
- Localization
- Bot protection

## Basic Setup

Create a middleware file at the root of your project:

```typescript
// middleware.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  // Your middleware logic here
  return NextResponse.next();
}

// Specify which routes the middleware should run on
export const config = {
  matcher: [
    /*
     * Match all paths except:
     * 1. /api/auth (NextAuth routes)
     * 2. /_next (Next.js internals)
     * 3. /static (static files)
     * 4. /favicon.ico, /robots.txt (static files)
     */
    "/((?!api/auth|_next|static|favicon.ico|robots.txt).*)",
  ],
};
```

## Authentication Middleware

Use middleware to protect routes that require authentication:

```typescript
// middleware.ts
import { NextResponse } from "next/server";
import { getToken } from "next-auth/jwt";
import type { NextRequest } from "next/server";

export async function middleware(request: NextRequest) {
  const path = request.nextUrl.pathname;
  
  // Define public paths that don't require authentication
  const publicPaths = ["/login", "/register", "/", "/about"];
  const isPublicPath = publicPaths.some(publicPath => 
    path === publicPath || path.startsWith("/api/auth")
  );

  // Get the user's auth token using NextAuth.js
  const token = await getToken({
    req: request,
    secret: process.env.NEXTAUTH_SECRET,
  });

  // Redirect unauthenticated users to login
  if (!token && !isPublicPath) {
    // Store the original URL as a callback URL after login
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

export const config = {
  matcher: [
    "/((?!api/auth|_next|fonts|images|favicon.ico|robots.txt).*)",
  ],
};
```

## Role-Based Authorization

Implement role-based access control in middleware:

```typescript
// middleware.ts
import { NextResponse } from "next/server";
import { getToken } from "next-auth/jwt";
import type { NextRequest } from "next/server";

export async function middleware(request: NextRequest) {
  const path = request.nextUrl.pathname;
  
  // Get user token
  const token = await getToken({
    req: request,
    secret: process.env.NEXTAUTH_SECRET,
  });
  
  // Define public paths
  const publicPaths = ["/login", "/register", "/", "/about"];
  const isPublicPath = publicPaths.some(publicPath => 
    path === publicPath || path.startsWith("/api/auth")
  );
  
  // Check authentication
  if (!token && !isPublicPath) {
    const url = new URL("/login", request.url);
    url.searchParams.set("callbackUrl", path);
    return NextResponse.redirect(url);
  }
  
  // Role-based access control
  const userRole = token?.role as string;
  
  // Admin-only routes
  if (path.startsWith("/admin") && userRole !== "admin") {
    return NextResponse.redirect(new URL("/unauthorized", request.url));
  }
  
  // Member-only routes
  if (path.startsWith("/members") && !["admin", "member"].includes(userRole)) {
    return NextResponse.redirect(new URL("/unauthorized", request.url));
  }
  
  return NextResponse.next();
}

export const config = {
  matcher: [
    "/((?!api/auth|_next|fonts|images|favicon.ico|robots.txt).*)",
  ],
};
```

## Request Headers

Add or modify request headers with middleware:

```typescript
// middleware.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  // Clone the request headers
  const requestHeaders = new Headers(request.headers);
  
  // Add a custom header
  requestHeaders.set("x-custom-header", "my-custom-value");
  
  // Add security headers
  const response = NextResponse.next({
    request: {
      headers: requestHeaders,
    },
  });
  
  // Add response headers
  response.headers.set("X-Frame-Options", "DENY");
  response.headers.set("X-Content-Type-Options", "nosniff");
  response.headers.set("Referrer-Policy", "strict-origin-when-cross-origin");
  response.headers.set(
    "Content-Security-Policy",
    "default-src 'self'; script-src 'self'"
  );
  
  return response;
}
```

## Localization Middleware

Implement language detection and redirection:

```typescript
// middleware.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

// List of supported locales
const locales = ["en", "fr", "es"];
const defaultLocale = "en";

export function middleware(request: NextRequest) {
  // Get the pathname from request URL
  const { pathname } = request.nextUrl;
  
  // Check if pathname already has a locale
  const pathnameHasLocale = locales.some(
    locale => pathname.startsWith(`/${locale}/`) || pathname === `/${locale}`
  );
  
  if (pathnameHasLocale) return NextResponse.next();
  
  // Get preferred locale from cookie or Accept-Language header
  const preferredLocale = getPreferredLocale(request, locales, defaultLocale);
  
  // Redirect to the locale path
  const newUrl = new URL(`/${preferredLocale}${pathname}`, request.url);
  
  return NextResponse.redirect(newUrl);
}

function getPreferredLocale(
  request: NextRequest, 
  locales: string[], 
  defaultLocale: string
): string {
  // First check cookie
  const localeCookie = request.cookies.get("NEXT_LOCALE")?.value;
  if (localeCookie && locales.includes(localeCookie)) {
    return localeCookie;
  }
  
  // Then check Accept-Language header
  const acceptLanguage = request.headers.get("Accept-Language");
  if (acceptLanguage) {
    const languages = acceptLanguage.split(",").map(lang => lang.split(";")[0].trim());
    
    for (const lang of languages) {
      const locale = locales.find(l => 
        l === lang || l === lang.split("-")[0]
      );
      if (locale) return locale;
    }
  }
  
  // Fallback to default
  return defaultLocale;
}

export const config = {
  matcher: [
    // Skip existing locale prefixes
    "/((?!api|_next|fonts|images|.*\\..*).+)",
  ],
};
```

## Rate Limiting

Implement basic rate limiting:

```typescript
// middleware.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

// Simple in-memory store for rate limiting
// In production, use Redis or a similar service
const rateLimit = new Map<string, { count: number; timestamp: number }>();

// Config
const RATE_LIMIT_MAX = 60;  // Maximum requests
const RATE_LIMIT_WINDOW = 60 * 1000;  // Window size in ms (1 minute)

export function middleware(request: NextRequest) {
  // Only apply rate limiting to API routes
  if (!request.nextUrl.pathname.startsWith("/api")) {
    return NextResponse.next();
  }
  
  // Get IP to use as identifier
  const ip = request.ip || "anonymous";
  
  // Get current rate limit data for this IP
  const currentTime = Date.now();
  const rateData = rateLimit.get(ip) || { count: 0, timestamp: currentTime };
  
  // Reset if outside window
  if (currentTime - rateData.timestamp > RATE_LIMIT_WINDOW) {
    rateData.count = 0;
    rateData.timestamp = currentTime;
  }
  
  // Increment request count
  rateData.count++;
  
  // Update store
  rateLimit.set(ip, rateData);
  
  // Check rate limit
  if (rateData.count > RATE_LIMIT_MAX) {
    return new NextResponse(
      JSON.stringify({ error: "Too many requests" }),
      {
        status: 429,
        headers: {
          "Content-Type": "application/json",
          "X-RateLimit-Limit": RATE_LIMIT_MAX.toString(),
          "X-RateLimit-Remaining": "0",
          "Retry-After": Math.ceil((rateData.timestamp + RATE_LIMIT_WINDOW - currentTime) / 1000).toString(),
        },
      }
    );
  }
  
  // Add rate limit headers
  const response = NextResponse.next();
  response.headers.set("X-RateLimit-Limit", RATE_LIMIT_MAX.toString());
  response.headers.set("X-RateLimit-Remaining", (RATE_LIMIT_MAX - rateData.count).toString());
  
  return response;
}

export const config = {
  matcher: [
    "/api/:path*",
  ],
};
```

## Feature Flags

Implement feature flags with middleware:

```typescript
// middleware.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

// Define feature flags
const FEATURES = {
  NEW_DASHBOARD: {
    enabled: false,
    allowedUsers: ["user1@example.com", "user2@example.com"],
  },
  BETA_FEATURE: {
    enabled: true,
    rolloutPercentage: 25, // Enable for 25% of users
  },
};

export async function middleware(request: NextRequest) {
  // Get current user from session token
  const token = await getToken({ req: request });
  const userEmail = token?.email as string | undefined;
  
  // Check if accessing new dashboard
  if (request.nextUrl.pathname === "/dashboard" && userEmail) {
    // Check if new dashboard is enabled for this user
    const newDashboardEnabled = 
      FEATURES.NEW_DASHBOARD.enabled || 
      FEATURES.NEW_DASHBOARD.allowedUsers.includes(userEmail);
    
    if (newDashboardEnabled) {
      // Redirect to new dashboard
      return NextResponse.rewrite(new URL("/dashboard-new", request.url));
    }
  }
  
  // Percentage-based rollout for beta feature
  if (request.nextUrl.pathname.startsWith("/feature") && FEATURES.BETA_FEATURE.enabled) {
    // Generate a hash from user email or IP
    const userIdentifier = userEmail || request.ip || "anonymous";
    const hash = hashString(userIdentifier);
    const percentage = Math.abs(hash % 100);
    
    if (percentage < FEATURES.BETA_FEATURE.rolloutPercentage) {
      // User is in the rollout group
      return NextResponse.rewrite(new URL("/feature-beta", request.url));
    }
  }
  
  return NextResponse.next();
}

// Simple string hash function
function hashString(str: string): number {
  let hash = 0;
  for (let i = 0; i < str.length; i++) {
    const char = str.charCodeAt(i);
    hash = ((hash << 5) - hash) + char;
    hash = hash & hash; // Convert to 32bit integer
  }
  return hash;
}
```

## Best Practices

1. **Keep middleware lightweight**: Middleware runs on every request to matched routes, so keep it efficient
2. **Use edge runtime when possible**: Add `export const runtime = 'edge'` to benefit from edge computing
3. **Cache aggressively**: For operations that can be cached, use `Response.json()` with cache headers
4. **Log minimal information**: Avoid logging sensitive data in middleware
5. **Test locally first**: Use tools like `next dev` to test middleware locally before deployment
6. **Monitor performance**: Watch for performance impacts after adding middleware
7. **Combine middleware carefully**: Consider the order of operations when multiple middleware features are needed
8. **Version your middleware**: Use version control and deployment strategies for middleware changes

## Middleware Checklist

- [ ] Create a middleware.ts file at project root
- [ ] Define appropriate matchers for routes that need middleware
- [ ] Implement authentication protection for private routes
- [ ] Add authorization checks for role-restricted areas
- [ ] Set security headers for all responses
- [ ] Consider rate limiting for API routes
- [ ] Test middleware functionality thoroughly
- [ ] Monitor performance impact of middleware
