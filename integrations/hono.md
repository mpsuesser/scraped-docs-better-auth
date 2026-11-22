---
url: https://better-auth.com/llms.txt/docs/integrations/hono
title: "Hono"
description: ""
access_date: 2026-11-22T12:47:47.467Z
current_date: 2026-11-22T12:47:47.467Z
---

Integrate Better Auth with Hono.

Both Hono and Better Auth use the Web Standard `Request` and `Response` APIs, so you can mount Better Auth directly in a Hono application without an adapter.

Before you begin, make sure you have a Better Auth instance configured. If not, follow the [installation guide](https://better-auth.com/docs/installation).

## Setup

### Create a Hono App

If you are starting a new project, create a Hono application and select the template for your runtime or platform:

#### npm

```
npx create-hono@latest
```

#### pnpm

#### yarn

#### bun

If you already have a Hono application, you can skip this step. See the [create-hono documentation](https://hono.dev/docs/guides/create-hono) for available templates and options, or Hono's [Getting Started guides](https://hono.dev/docs/getting-started/basic) for runtime-specific setup and deployment.

### Mount the Auth Handler

Add the highlighted route to your existing Hono application:

```
import { Hono } from "hono";
import { auth } from "./auth";

const app = new Hono();

app.all("/api/auth/*", (c) => auth.handler(c.req.raw)); 

export default app;
```

That's all you need to connect Better Auth to Hono!

If you're using another runtime, keep your existing entry point and add the same `app.all()` route.

`app.all()` forwards every HTTP method to Better Auth using the raw Web Standard `Request` from `c.req.raw`. Better Auth validates the method and returns a `Response` that Hono sends directly. Register the auth route before any catch-all route that could handle the request first.

### Additional Configuration

#### Cloudflare Workers

See the [Cloudflare guide](https://better-auth.com/docs/guides/cloudflare) for the Workers configuration Better Auth needs.

## CORS

To allow cross-origin authentication requests, register Hono's [CORS middleware](https://hono.dev/docs/middleware/builtin/cors) before the Better Auth route:

```
import { Hono } from "hono";
import { cors } from "hono/cors"; 
import { auth } from "./auth";

const app = new Hono();

app.use( 
    "/api/auth/*", 
    cors({ 
        origin: "http://localhost:3001", 
        credentials: true, 
    }), 
); 

app.all("/api/auth/*", (c) => auth.handler(c.req.raw));

export default app;
```

Add the same origin to Better Auth's `trustedOrigins` configuration:

```
import { betterAuth } from "better-auth";

export const auth = betterAuth({
    trustedOrigins: ["http://localhost:3001"], 
});
```

When `credentials` is enabled, configure an explicit CORS origin instead of `*` and add the same origin to Better Auth's `trustedOrigins`. See [Cookies](https://better-auth.com/docs/concepts/cookies) for cross-domain and cross-subdomain cookie behavior.

## Middleware

You can also use custom middleware to make the current session available through Hono's context:

```
import { createMiddleware } from "hono/factory";
import { auth } from "./auth";

type Env = {
    Variables: {
        session: typeof auth.$Infer.Session | null;
    };
};

export const sessionMiddleware = createMiddleware<Env>(async (c, next) => {
    const session = await auth.api.getSession({
        headers: c.req.raw.headers,
    });

    c.set("session", session);

    await next();
});
```

Add the middleware to any route that needs access to the current session:

```
import { Hono } from "hono";
import { HTTPException } from "hono/http-exception";
import { sessionMiddleware } from "./session-middleware";

const app = new Hono();

app.get("/hello", sessionMiddleware, (c) => {
    const session = c.get("session");

    if (!session) {
        throw new HTTPException(401);
    }

    return c.json({ message: \`Hello, ${session.user.name}.\` });
});

export default app;
```

This example applies the middleware to a single route. For other patterns, see Hono's [Middleware guide](https://hono.dev/docs/guides/middleware).

## Hono RPC

The Better Auth client uses `credentials: "include"` by default. If you use Hono RPC to call authenticated Hono routes from another origin, configure the RPC client to include credentials as well:

#### Client

```
import { hc } from "hono/client";
import type { AppType } from "./server";

const client = hc<AppType>("http://localhost:8787", {
    init: { 
        credentials: "include", 
    }, 
});

const response = await client.hello.$get();
```

#### Server

This configuration is only required for cross-origin requests. For more information, see Hono's [RPC cookies guide](https://hono.dev/docs/guides/rpc#cookies).
