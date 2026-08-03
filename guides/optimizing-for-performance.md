---
url: https://better-auth.com/llms.txt/docs/guides/optimizing-for-performance
title: "Optimizing For Performance"
description: ""
access_date: 2026-08-03T19:43:07.705Z
current_date: 2026-08-03T19:43:07.705Z
---

A guide to optimizing your Better Auth application for performance.

In this guide, we’ll go over some of the ways you can optimize your application for a more performant Better Auth app.

## Caching

Caching is a powerful technique that can significantly improve the performance of your Better Auth application by reducing the number of database queries and speeding up response times.

### Cookie Cache

Calling your database every time `useSession` or `getSession` is invoked isn’t ideal, especially if sessions don’t change frequently. Cookie caching handles this by storing session data in a short-lived, signed cookie similar to how JWT access tokens are used with refresh tokens.

To turn on cookie caching, just set `session.cookieCache` in your auth config:

```
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  session: {
    cookieCache: {
      enabled: true,
      maxAge: 5 * 60, // Cache duration in seconds
    },
  },
});
```

Read more about [cookie caching](https://better-auth.com/docs/concepts/session-management#cookie-cache).

### Framework Caching

Here are examples of how you can do caching in different frameworks and environments:

#### Next

Since Next v15, we can use the `"use cache"` directive to cache the response of a server function.

```
export async function getUsers() {
    'use cache'
    const { users } = await auth.api.listUsers();
    return users
}
```

Learn more about [NextJS use cache directive](https://nextjs.org/docs/app/api-reference/directives/use-cache).

#### react-router

#### SolidStart

#### TanStack Query

## Background Tasks

On serverless platforms (Vercel, Cloudflare Workers), you can improve response times by deferring non-critical work—cleanup, analytics, rate limit updates, email sending—to run after the response is sent. Configure [`advanced.backgroundTasks`](https://better-auth.com/docs/reference/options#backgroundtasks) and use `ctx.context.runInBackground` or `ctx.context.runInBackgroundOrAwait` in hooks.

```
import { betterAuth } from "better-auth";
import { createAuthMiddleware } from "better-auth/api";
import { waitUntil } from "@vercel/functions";

export const auth = betterAuth({
  advanced: {
    backgroundTasks: { handler: waitUntil },
  },
  hooks: {
    after: createAuthMiddleware(async (ctx) => {
      if (ctx.path === "/sign-up/email") {
        ctx.context.runInBackground(logSignUp(ctx.context.newSession?.user.id));
      }
    }),
  },
});
```

See the [Context documentation](https://better-auth.com/docs/concepts/hooks#runinbackground) for more examples.

## SSR Optimizations

If you're using a framework that supports server-side rendering, it's usually best to pre-fetch the user session on the server and use it as a fallback on the client.

```
const session = await auth.api.getSession({
  headers: await headers(),
});
//then pass the session to the client
```

## Database optimizations

Optimizing database performance is essential to get the best out of Better Auth.

#### Recommended fields to index

| Table | Fields | Plugin |
| --- | --- | --- |
| users | `email` |  |
| accounts | `userId` |  |
| sessions | `userId`, `token` |  |
| verifications | `identifier` |  |
| invitations | `email`, `organizationId` | organization |
| members | `userId`, `organizationId` | organization |
| organizations | `slug` | organization |
| passkey | `userId` | passkey |
| twoFactor | `secret` | twoFactor |

## Bundle Size Optimization

If you're using custom adapters (like Prisma, Drizzle, or MongoDB), you can reduce your bundle size by using `better-auth/minimal` instead of `better-auth`. This version excludes Kysely, which is only needed when using direct database connections.

### Usage

Simply import from `better-auth/minimal` instead of `better-auth`:

#### Prisma

```
import { betterAuth } from "better-auth/minimal"; 
import { prismaAdapter } from "better-auth/adapters/prisma";
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

export const auth = betterAuth({
  database: prismaAdapter(prisma, {
    provider: "postgresql", // or "mysql", "sqlite"
  }),
});
```
