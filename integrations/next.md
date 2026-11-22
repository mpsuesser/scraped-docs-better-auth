---
url: https://better-auth.com/llms.txt/docs/integrations/next
title: "Next"
description: ""
access_date: 2026-11-22T04:36:49.619Z
current_date: 2026-11-22T04:36:49.619Z
---

Integrate Better Auth with Next.js.

Better Auth can be easily integrated with Next.js. Before you start, make sure you have a Better Auth instance configured. If you haven't done that yet, check out the [installation](https://better-auth.com/docs/installation).

### Create API Route

We need to mount the handler to an API route. Create a route file inside `/api/auth/[...all]` directory. And add the following code:

```
import { auth } from "@/lib/auth";
import { toNextJsHandler } from "better-auth/next-js";

export const { GET, POST } = toNextJsHandler(auth);
```

For `pages` route, you need to use `toNodeHandler` instead of `toNextJsHandler` and set `bodyParser` to `false` in the `config` object. Here is an example:

```
import { toNodeHandler } from "better-auth/node"
import { auth } from "@/lib/auth"

// Disallow body parsing, we will parse it manually
export const config = { api: { bodyParser: false } }

export default toNodeHandler(auth.handler)
```

## Create a client

Create a client instance. You can name the file anything you want. Here we are creating `auth-client.ts` file inside the `lib/` directory.

```
import { createAuthClient } from "better-auth/react" // make sure to import from better-auth/react

export const authClient =  createAuthClient({
    //you can pass client configuration here
})
```

Once you have created the client, you can use it to sign up, sign in, and perform other actions. Some of the actions are reactive. The client uses [nano-store](https://github.com/nanostores/nanostores) to store the state and re-render the components when the state changes.

The client also uses [better-fetch](https://github.com/bekacru/better-fetch) to make the requests. You can pass the fetch configuration to the client.

## RSC and Server actions

The `api` object exported from the auth instance contains all the actions that you can perform on the server. Every endpoint made inside Better Auth is invocable as a function. Including plugins endpoints.

**Example: Getting Session on a server action**

```
import { auth } from "@/lib/auth"
import { headers } from "next/headers"

const someAuthenticatedAction = async () => {
    "use server";
    const session = await auth.api.getSession({
        headers: await headers()
    })
};
```

**Example: Getting Session on a RSC**

```
import { auth } from "@/lib/auth"
import { headers } from "next/headers"

export async function ServerComponent() {
    const session = await auth.api.getSession({
        headers: await headers()
    })
    if(!session) {
        return <div>Not authenticated</div>
    }
    return (
        <div>
            <h1>Welcome {session.user.name}</h1>
        </div>
    )
}
```

### Server Action Cookies

When you call a function that needs to set cookies, like `signOut` in a server action, cookies won’t be set. This is because server actions need to use the `cookies` helper from Next.js to set cookies.

To simplify this, you can use the `nextCookies` plugin, which will automatically set cookies for you whenever a `Set-Cookie` header is present in the response.

```
import { betterAuth } from "better-auth";
import { nextCookies } from "better-auth/next-js";

export const auth = betterAuth({
    //...your config
    plugins: [nextCookies()] // make sure this is the last plugin in the array
})
```

Now, when you call functions that set cookies, they will be automatically set.

```
"use server";
import { headers } from "next/headers";
import { auth } from "@/lib/auth"

export const signOut = async () => {
    await auth.api.signOut({
        headers: await headers()
    })
}
```

## Auth Protection

In Next.js proxy/middleware, it's recommended to only check for the existence of a session cookie to handle redirection to avoid blocking requests by making API or database calls.

### Next.js 16+ (Proxy)

Next.js 16 replaces "middleware" with "proxy". You can use the Node.js runtime for full session validation with database checks:

```
import { NextRequest, NextResponse } from "next/server";
import { headers } from "next/headers";
import { auth } from "@/lib/auth";

export async function proxy(request: NextRequest) {
    const session = await auth.api.getSession({
        headers: await headers()
    })

    if(!session) {
        return NextResponse.redirect(new URL("/sign-in", request.url));
    }

    return NextResponse.next();
}

export const config = {
  matcher: ["/dashboard"], // Specify the routes the middleware applies to
};
```

For cookie-only checks (faster but less secure), use `getSessionCookie`:

```
import { NextRequest, NextResponse } from "next/server";
import { getSessionCookie } from "better-auth/cookies";

export async function proxy(request: NextRequest) {
    const sessionCookie = getSessionCookie(request);

    // THIS IS NOT SECURE!
    // This is the recommended approach to optimistically redirect users
    // We recommend handling auth checks in each page/route
    if (!sessionCookie) {
        return NextResponse.redirect(new URL("/", request.url));
    }

    return NextResponse.next();
}

export const config = {
    matcher: ["/dashboard"], // Specify the routes the middleware applies to
};
```

### Next.js 15.2.0+ (Node.js Runtime Middleware)

From Next.js 15.2.0, you can use the Node.js runtime in middleware for full session validation with database checks:

```
import { NextRequest, NextResponse } from "next/server";
import { headers } from "next/headers";
import { auth } from "@/lib/auth";

export async function middleware(request: NextRequest) {
    const session = await auth.api.getSession({
        headers: await headers()
    })
    
    if(!session) {
        return NextResponse.redirect(new URL("/sign-in", request.url));
    }

    return NextResponse.next();
}

export const config = {
  runtime: "nodejs", // Required for auth.api calls
  matcher: ["/dashboard"], // Specify the routes the middleware applies to
};
```

### Next.js 13-15.1.x (Edge Runtime Middleware)

In older Next.js versions, middleware runs on the Edge Runtime and cannot make database calls. Use cookie-based checks for optimistic redirects:

#### For Next.js release 15.1.7 and below

If you need the full session object, you'll have to fetch it from the `/api/auth/get-session` API route. Since Next.js middleware doesn't support running Node.js APIs directly, you must make an HTTP request.

```
import { betterFetch } from "@better-fetch/fetch";
import type { auth } from "@/lib/auth";
import { NextRequest, NextResponse } from "next/server";

type Session = typeof auth.$Infer.Session;

export async function middleware(request: NextRequest) {
    const { data: session } = await betterFetch<Session>("/api/auth/get-session", {
        baseURL: request.nextUrl.origin,
        headers: {
            cookie: request.headers.get("cookie") || "", // Forward the cookies from the request
        },
    });

    if (!session) {
        return NextResponse.redirect(new URL("/sign-in", request.url));
    }

    return NextResponse.next();
}

export const config = {
    matcher: ["/dashboard"], // Apply middleware to specific routes
};
```

#### For Next.js release 15.2.0 and above

From Next.js 15.2.0, you can use the Node.js runtime in middleware for full session validation with database checks:

```
import { NextRequest, NextResponse } from "next/server";
import { headers } from "next/headers";
import { auth } from "@/lib/auth";

export async function middleware(request: NextRequest) {
    const session = await auth.api.getSession({
        headers: await headers()
    })

    if(!session) {
        return NextResponse.redirect(new URL("/sign-in", request.url));
    }

    return NextResponse.next();
}

export const config = {
  runtime: "nodejs",
  matcher: ["/dashboard"], // Apply middleware to specific routes
};
```

#### Cookie-based checks (recommended for all versions)

```
import { NextRequest, NextResponse } from "next/server";
import { getSessionCookie } from "better-auth/cookies";

export async function middleware(request: NextRequest) {
    const sessionCookie = getSessionCookie(request);

    // THIS IS NOT SECURE!
    // This is the recommended approach to optimistically redirect users
    // We recommend handling auth checks in each page/route
    if (!sessionCookie) {
        return NextResponse.redirect(new URL("/", request.url));
    }

    return NextResponse.next();
}

export const config = {
    matcher: ["/dashboard"], // Specify the routes the middleware applies to
};
```

Alternatively, you can use the `getCookieCache` helper to get the session object from the cookie cache.

```
import { getCookieCache } from "better-auth/cookies";

export async function middleware(request: NextRequest) {
    const session = await getCookieCache(request);
    if (!session) {
        return NextResponse.redirect(new URL("/sign-in", request.url));
    }
    return NextResponse.next();
}
```

### How to handle auth checks in each page/route

In this example, we are using the `auth.api.getSession` function within a server component to get the session object, then we are checking if the session is valid. If it's not, we are redirecting the user to the sign-in page.

```
import { auth } from "@/lib/auth";
import { headers } from "next/headers";
import { redirect } from "next/navigation";

export default async function DashboardPage() {
    const session = await auth.api.getSession({
        headers: await headers()
    })

    if(!session) {
        redirect("/sign-in")
    }

    return <h1>Welcome {session.user.name}</h1>
}
```

## Next.js 16 Compatibility

Better Auth is fully compatible with Next.js 16. The main change is that "middleware" is now called "proxy". See the [Auth Protection](#auth-protection) section above for Next.js 16+ proxy examples.

### Migration Guide

Use Next.js codemod for automatic migration:

```
npx @next/codemod@canary middleware-to-proxy .
```

Or manually:

- Rename `middleware.ts` → `proxy.ts`
- Change function name: `middleware` → `proxy`

All Better Auth methods work identically. See the [Next.js migration guide](https://nextjs.org/docs/app/api-reference/file-conventions/proxy#migration-to-proxy) for details.
