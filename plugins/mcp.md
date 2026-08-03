---
url: https://better-auth.com/llms.txt/docs/plugins/mcp
title: "Mcp"
description: ""
access_date: 2026-08-03T19:43:07.705Z
current_date: 2026-08-03T19:43:07.705Z
---

MCP provider plugin for Better Auth

`OAuth` `MCP`

The **MCP** plugin lets your app act as an OAuth provider for MCP clients. It handles authentication and makes it easy to issue and manage access tokens for MCP applications.

## Installation

### Add the Plugin

Add the MCP plugin to your auth configuration and specify the login page path.

```
import { betterAuth } from "better-auth";
import { mcp } from "better-auth/plugins"; 

export const auth = betterAuth({
    plugins: [
        mcp({ 
            loginPage: "/sign-in" // path to your login page
        }) 
    ]
});
```

### Generate Schema

Run the migration or generate the schema to add the necessary fields and tables to the database.

#### migrate

#### npm

#### generate

```
npx auth migrate
```

#### pnpm

#### yarn

#### bun

The MCP plugin uses the same schema as the OIDC Provider plugin. See the [OIDC Provider Schema](https://better-auth.com/docs/plugins/oidc-provider#schema) section for details.

## Usage

### OAuth Discovery Metadata

Better Auth already handles the `/api/auth/.well-known/oauth-authorization-server` route automatically but some client may fail to parse the `WWW-Authenticate` header and default to `/.well-known/oauth-authorization-server` (this can happen, for example, if your CORS configuration doesn't expose the `WWW-Authenticate`). For this reason it's better to add a route to expose OAuth metadata for MCP clients:

```
import { oAuthDiscoveryMetadata } from "better-auth/plugins";
import { auth } from "../../../lib/auth";

export const GET = oAuthDiscoveryMetadata(auth);
```

### OAuth Protected Resource Metadata

Better Auth already handles the `/api/auth/.well-known/oauth-protected-resource` route automatically but some client may fail to parse the `WWW-Authenticate` header and default to `/.well-known/oauth-protected-resource` (this can happen, for example, if your CORS configuration doesn't expose the `WWW-Authenticate`). For this reason it's better to add a route to expose OAuth metadata for MCP clients:

```
import { oAuthProtectedResourceMetadata } from "better-auth/plugins";
import { auth } from "@/lib/auth";

export const GET = oAuthProtectedResourceMetadata(auth);
```

### MCP Session Handling

You can use the helper function `withMcpAuth` to get the session and handle unauthenticated calls automatically.

```
import { auth } from "@/lib/auth";
import { createMcpHandler } from "@vercel/mcp-adapter";
import { withMcpAuth } from "better-auth/plugins";
import { z } from "zod";

const handler = withMcpAuth(auth, (req, session) => {
    // session contains the access token record with scopes and user ID
    return createMcpHandler(
        (server) => {
            server.tool(
                "echo",
                "Echo a message",
                { message: z.string() },
                async ({ message }) => {
                    return {
                        content: [{ type: "text", text: \`Tool echo: ${message}\` }],
                    };
                },
            );
        },
        {
            capabilities: {
                tools: {
                    echo: {
                        description: "Echo a message",
                    },
                },
            },
        },
        {
            redisUrl: process.env.REDIS_URL,
            basePath: "/api",
            verboseLogs: true,
            maxDuration: 60,
        },
    )(req);
});

export { handler as GET, handler as POST, handler as DELETE };
```

You can also use `auth.api.getMcpSession` to get the session using the access token sent from the MCP client:

```
import { auth } from "@/lib/auth";
import { createMcpHandler } from "@vercel/mcp-adapter";
import { z } from "zod";

const handler = async (req: Request) => {
     // session contains the access token record with scopes and user ID
    const session = await auth.api.getMcpSession({
        headers: req.headers
    })
    if(!session){
        //this is important and you must return 401
        return new Response(null, {
            status: 401
        })
    }
    return createMcpHandler(
        (server) => {
            server.tool(
                "echo",
                "Echo a message",
                { message: z.string() },
                async ({ message }) => {
                    return {
                        content: [{ type: "text", text: \`Tool echo: ${message}\` }],
                    };
                },
            );
        },
        {
            capabilities: {
                tools: {
                    echo: {
                        description: "Echo a message",
                    },
                },
            },
        },
        {
            redisUrl: process.env.REDIS_URL,
            basePath: "/api",
            verboseLogs: true,
            maxDuration: 60,
        },
    )(req);
}

export { handler as GET, handler as POST, handler as DELETE };
```

## Configuration

The MCP plugin accepts the following configuration options:

Prop

### OIDC Configuration

The plugin supports additional OIDC configuration options through the `oidcConfig` parameter:

Prop

## Remote MCP Client

The examples above use `withMcpAuth` which requires the Better Auth instance to be in the **same process** as the MCP server. If your MCP server runs as a separate service (different repo, different runtime, different language), you can use the **MCP Client** — a lightweight, framework-agnostic HTTP client that validates Bearer tokens against a remote Better Auth server.

### Setup

### Create the client

Point it at your Better Auth server's URL (the same `baseURL` + `basePath` from your auth config):

```
import { createMcpAuthClient } from "better-auth/plugins/mcp/client"

const mcpAuth = createMcpAuthClient({ 
    authURL: "http://localhost:3000/api/auth"
})
```

### Protect your MCP routes

Use the `handler` wrapper for Web Standard `Request` / `Response` (works with Deno, Bun, Cloudflare Workers, etc.):

```
const handler = mcpAuth.handler(async (req, session) => { 
    // session.userId, session.scopes, session.clientId
    return new Response(JSON.stringify({
        jsonrpc: "2.0",
        result: { userId: session.userId },
        id: 1
    }))
}) 

Deno.serve(handler)
```

### Mount discovery endpoints

MCP clients need to discover your OAuth server. Mount these at the root of your MCP server:

```
const discovery = mcpAuth.discoveryHandler() 
const protectedResource = mcpAuth.protectedResourceHandler("http://localhost:4000") 

// GET /.well-known/oauth-authorization-server → proxied from Better Auth
// GET /.well-known/oauth-protected-resource → points MCP clients to Better Auth
```

These proxy and cache the metadata from your Better Auth server, so MCP clients can discover the authorization, token, and registration endpoints automatically.

### Framework Adapters

#### Hono

```
import { Hono } from "hono"
import { mcpAuthHono } from "better-auth/plugins/mcp/client/adapters"

const app = new Hono()
const { middleware, discoveryRoutes } = mcpAuthHono({ 
    authURL: "http://localhost:3000/api/auth"
}) 

// Mount OAuth discovery endpoints (required by MCP spec)
discoveryRoutes(app, "http://localhost:4000") 

// Protect MCP routes
app.use("/mcp/*", middleware) 

app.post("/mcp", (c) => {
    const session = c.get("mcpSession") 
    // session.userId, session.scopes, etc.
})
```

#### Express

#### Official MCP SDK

#### mcp-use

### Options

Prop

### Session Object

The session object returned by `verifyToken` and passed to handlers contains:

Prop

## Schema

The MCP plugin uses the same schema as the OIDC Provider plugin. See the [OIDC Provider Schema](https://better-auth.com/docs/plugins/oidc-provider#schema) section for details.
