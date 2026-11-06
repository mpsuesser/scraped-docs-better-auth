---
url: https://better-auth.com/llms.txt/docs/integrations/waku
title: "Waku"
description: ""
access_date: 2026-11-06T22:41:26.774Z
current_date: 2026-11-06T22:41:26.774Z
---

Integrate Better Auth with Waku.

Before you start, make sure you have a Better Auth instance configured. If you haven't done that yet, check out the [installation](https://better-auth.com/docs/installation). The examples on this page use the APIs of Waku `1.0.0-rc.4`.

## Installation

## Mount the handler

Create a catch-all API route at `src/pages/_api/api/auth/[...route].ts`:

```
import { auth } from "../../../../auth" // Adjust the path as necessary

export default function handler(request: Request): Promise<Response> {
  return auth.handler(request)
}
```

## Create a client

Create a client with `better-auth/react`:

```
import { createAuthClient } from "better-auth/react"

export const authClient = createAuthClient()
```

## Collect cookies from server code

Server actions and server components can't set cookies on Waku's response. Add a plugin that collects the cookies Better Auth sets, and a `getSession()` helper that reads them, so pages rendered later in the same request see the change:

```
import { betterAuth, type BetterAuthPlugin } from "better-auth"
import { createAuthMiddleware } from "better-auth/api"
import { applySetCookies } from "better-auth/cookies"
import { getContext } from "hono/context-storage"

export const auth = betterAuth({
  //...your config
  plugins: [wakuCookies()] // make sure this is the last plugin in the array
})

type Session = typeof auth.$Infer.Session

export type AuthEnv = {
  Variables: {
    betterAuthSetCookie: string[]
    sessionPromise?: Promise<Session | null>
  }
}

function wakuCookies() {
  return {
    id: "waku-cookies",
    hooks: {
      after: [
        {
          matcher: () => true,
          handler: createAuthMiddleware(async (ctx) => {
            // Requests to the auth handler set cookies on their own response
            if ("_flag" in ctx && ctx._flag === "router") return
            const setCookies = ctx.context.responseHeaders?.getSetCookie() ?? []
            if (!setCookies.length) return
            const c = getContext<AuthEnv>()
            // A single request can make several auth calls
            c.set("betterAuthSetCookie", [...c.var.betterAuthSetCookie, ...setCookies])
            // The cookies changed, so the next read fetches the session again
            c.set("sessionPromise", undefined)
          }),
        },
      ],
    },
  } satisfies BetterAuthPlugin
}

/**
 * Returns the current session, fetched once per request until cookies change.
 * Cookies set earlier in the request take precedence over the incoming ones.
 */
export function getSession() {
  const c = getContext<AuthEnv>()
  if (c.var.sessionPromise) return c.var.sessionPromise
  const headers = new Headers(c.req.raw.headers)
  applySetCookies(headers, c.var.betterAuthSetCookie)
  const sessionPromise = auth.api.getSession({ headers })
  c.set("sessionPromise", sessionPromise)
  return sessionPromise
}
```

## Write cookies to the response

Waku loads files in `src/middleware` automatically. This middleware provides the request context for the plugin, and adds the collected cookies once Waku's `Response` exists:

```
import { every } from "hono/combine"
import { contextStorage } from "hono/context-storage"
import { createMiddleware } from "hono/factory"
import type { AuthEnv } from "../auth"

const authMiddleware = () =>
  every(
    contextStorage(),
    createMiddleware<AuthEnv>(async (c, next) => {
      c.set("betterAuthSetCookie", [])
      await next()
      for (const cookie of c.var.betterAuthSetCookie) {
        c.header("set-cookie", cookie, { append: true })
      }
    }),
  )

export default authMiddleware
```

## Resolve the session before rendering

Waku streams server component responses, so headers are sent before components finish rendering. A [cookie cache](https://better-auth.com/docs/concepts/session-management#cookie-cache) or expiry refresh inside a layout or page would lose its cookies. Resolve the session in a handler interceptor, before rendering starts:

```
import { tryGetContext } from "hono/context-storage"
import { getRequest, type HandlerInterceptor } from "waku/router/server"
import { getSession } from "../../auth"

const sessionInterceptor: HandlerInterceptor = async (next) => {
  // Static renders at build time have no Hono context, and no session.
  if (!tryGetContext()) return next()
  // API routes are not streamed. Under \`/api/auth\`, a refreshed session
  // cookie would also override the one sign-out expires.
  const { pathname } = new URL(getRequest().url)
  if (pathname.startsWith("/api/")) return next()
  await getSession()
  return next()
}

export default sessionInterceptor
```

## Read the session

Call `getSession()` in pages, server actions, and API routes. Pages that read the session must use `render: "dynamic"`:

```
import { unstable_redirect as redirect } from "waku/router/server"
import { getSession } from "../auth"

export default async function DashboardPage() {
  const session = await getSession()

  if (!session) {
    redirect("/sign-in")
  }

  return (
    <div>
      <h1>Welcome {session.user.name}</h1>
    </div>
  )
}

export const getConfig = async () => {
  return { render: "dynamic" } as const
}
```

## Usage

### Sign in from the client

Sign users in with `authClient`, so the request goes through the auth handler and its [rate limiting](https://better-auth.com/docs/concepts/rate-limit):

```
"use client"

import { useState } from "react"
import { useRouter } from "waku"
import { authClient } from "../lib/auth-client"

export function SignInForm() {
  const router = useRouter()
  const [error, setError] = useState<string>()
  return (
    <form
      onSubmit={async (event) => {
        event.preventDefault()
        const form = new FormData(event.currentTarget)
        const { error } = await authClient.signIn.email({
          email: String(form.get("email")),
          password: String(form.get("password")),
        })
        if (error) return setError(error.message)
        router.push("/dashboard")
      }}
    >
      <input name="email" type="email" />
      <input name="password" type="password" />
      <button type="submit">Sign in</button>
      {error && <p>{error}</p>}
    </form>
  )
}
```

### Sign out from a server action

Call endpoints through `auth.api` with the request headers. The middleware adds the cookies they set to the response, and pages rendered later in the same request see the change:

```
"use server"

import { getRequest, unstable_redirect as redirect } from "waku/router/server"
import { auth } from "./auth"

export async function signOut() {
  await auth.api.signOut({ headers: getRequest().headers })
  redirect("/sign-in")
}
```

Use it as a form action, such as `<form action={signOut}>`. Calls through `auth.api` skip rate limiting, so keep sign-in and sign-up on the client.

### Redirect optimistically in middleware

To redirect before rendering, check for the session cookie in middleware. A client-side navigation requests `/RSC/R/dashboard.txt` instead of `/dashboard`, so match the route and build the redirect with Waku's router helpers:

```
import type { MiddlewareHandler } from "hono"
import { getSessionCookie } from "better-auth/cookies"
import {
  unstable_formatRouterRequest as formatRouterRequest,
  unstable_parseRouterRequest as parseRouterRequest,
} from "waku/router/server"

const redirectMiddleware = (): MiddlewareHandler => {
  return async (c, next) => {
    const route = parseRouterRequest(c.req.raw)
    const isProtected =
      route?.type === "route" &&
      (route.path === "/dashboard" || route.path.startsWith("/dashboard/"))
    // Only checks that the cookie exists. Pages still verify the session.
    if (isProtected && !getSessionCookie(c.req.raw)) {
      const location = formatRouterRequest(c.req.raw, "/sign-in")
      if (location) {
        // 303, so a form posted without JavaScript is not resent
        return c.redirect(location.pathname + location.search, 303)
      }
    }
    await next()
  }
}

export default redirectMiddleware
```

In production, pages rendered as `static` are served before middleware runs, so protected pages must use `render: "dynamic"`.

### Show the session on static pages

To keep a page static, move session-dependent UI into a lazy dynamic [slice](https://waku.gg/#slices) and render it with `<Slice id="account-menu" lazy fallback={<span />} />`. The page is built once, and the slice is rendered per request:

```
import { getSession } from "../../auth"

export default async function AccountMenu() {
  const session = await getSession()
  if (!session) {
    return <a href="/sign-in">Sign in</a>
  }
  return <a href="/dashboard">{session.user.name}</a>
}

export const getConfig = async () => {
  return { render: "dynamic" } as const
}
```
