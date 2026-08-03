---
url: https://better-auth.com/llms.txt/docs/integrations/tanstack
title: "Tanstack"
description: ""
access_date: 2026-08-03T19:43:07.705Z
current_date: 2026-08-03T19:43:07.705Z
---

Integrate Better Auth with TanStack Start.

This integration guide is assuming you are using TanStack Start.

Before you start, make sure you have a Better Auth instance configured. If you haven't done that yet, check out the [installation](https://better-auth.com/docs/installation).

## Quick Start

You can create a new TanStack Start project with Better Auth integrated using the following command. This CLI sets up a project with an auth instance configured with the plugin and mounted handlers.

#### npm

```
npm create @tanstack/start

◇  What add-ons would you like for your project?
│  Better Auth
```

#### pnpm

#### yarn

#### bun

## Usage

### Mount the handler

We need to mount the handler to a TanStack API endpoint/Server Route. Create a new file: `/src/routes/api/auth/$.ts`

```
import { auth } from '@/lib/auth'
import { createFileRoute } from '@tanstack/react-router'

export const Route = createFileRoute('/api/auth/$')({
    server: {
        handlers: {
            GET: async ({ request }:{ request: Request }) => {
                return await auth.handler(request)
            },
            POST: async ({ request }:{ request: Request }) => {
                return await auth.handler(request)
            },
        },
    },
})
```

### Usage tips

- We recommend using the client SDK or `authClient` to handle authentication, rather than server actions with `auth.api`.
- When you call functions that need to set cookies (like `signInEmail` or `signUpEmail`), you'll need to handle cookie setting for TanStack Start. Better Auth provides a `tanstackStartCookies` plugin to automatically handle this for you.

For React (TanStack Start with React):

```
import { betterAuth } from "better-auth";
import { tanstackStartCookies } from "better-auth/tanstack-start";

export const auth = betterAuth({
    //...your config
    plugins: [tanstackStartCookies()] // make sure this is the last plugin in the array
})
```

For Solid.js (TanStack Start with Solid):

```
import { betterAuth } from "better-auth";
import { tanstackStartCookies } from "better-auth/tanstack-start/solid";

export const auth = betterAuth({
    //...your config
    plugins: [tanstackStartCookies()] // make sure this is the last plugin in the array
})
```

Now, when you call functions that set cookies, they will be automatically set using TanStack Start's cookie handling system.

```
import { auth } from "@/lib/auth"

const signIn = async () => {
    await auth.api.signInEmail({
        body: {
            email: "user@email.com",
            password: "password",
        }
    })
}
```

### Protecting Resources

To protect resources that require authentication, use `beforeLoad` with a server function. This ensures authentication is checked on every navigation, including client-side navigation via `<Link>` components.

First, create server-side helpers to check the session:

```
import { createServerFn } from "@tanstack/react-start";
import { getRequestHeaders } from "@tanstack/react-start/server";
import { auth } from "@/lib/auth";

export const getSession = createServerFn({ method: "GET" }).handler(async () => {
    const headers = getRequestHeaders();
    const session = await auth.api.getSession({ headers });

    return session;
});

export const ensureSession = createServerFn({ method: "GET" }).handler(async () => {
    const headers = getRequestHeaders();
    const session = await auth.api.getSession({ headers });

    if (!session) {
        throw new Error("Unauthorized");
    }

    return session;
});
```

#### Protecting Routes

Use `beforeLoad` in your route definitions:

```
import { createFileRoute, redirect } from '@tanstack/react-router'
import { getSession } from '@/lib/auth.functions'

export const Route = createFileRoute('/dashboard')({
  beforeLoad: async () => {
    const session = await getSession();

    if (!session) {
      throw redirect({ to: "/login" });
    }

    return { user: session.user };
  },
  component: Dashboard,
})

function Dashboard() {
  const { user } = Route.useRouteContext();
  
  return <div>Welcome, {user.name}!</div>
}
```

#### Protecting Multiple Routes (Layout)

For protecting multiple routes, use a pathless layout route:

```
import { createFileRoute, redirect, Outlet } from '@tanstack/react-router'
import { getSession } from '@/lib/auth.functions'

export const Route = createFileRoute('/_protected')({
  beforeLoad: async ({ location }) => {
    const session = await getSession();

    if (!session) {
      throw redirect({
        to: "/login",
        search: { redirect: location.href },
      });
    }

    return { user: session.user };
  },
  component: () => <Outlet />,
})
```

Then nest protected routes under `_protected`:

dashboard.tsx

settings.tsx

\_protected.tsx

login.tsx

#### Protecting Server Functions

Use `ensureSession` helper to protect server functions:

```
import { createServerFn } from "@tanstack/react-start";
import { ensureSession } from "./auth.functions";

export const createPost = createServerFn({ method: "POST" })
  .inputValidator((data: { title: string }) => data)
  .handler(async ({ data }) => {
    const session = await ensureSession();
    const post = await db.posts.create({
      title: data.title,
      authorId: session.user.id,
    });
    
    return post;
  });
```
