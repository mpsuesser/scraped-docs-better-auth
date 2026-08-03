---
url: https://better-auth.com/llms.txt/docs/plugins/agent-auth
title: "Agent Auth"
description: ""
access_date: 2026-08-03T19:43:07.705Z
current_date: 2026-08-03T19:43:07.705Z
---

Agent identity, registration, discovery, and capability-based authorization for AI agents.

`AI Agents` `MCP` `Capabilities`

The Agent Auth plugin lets your Better Auth server act as an **Agent Auth provider**. It's a server implementation of the [Agent Auth Protocol](https://agentauthprotocol.com/).

It gives AI agents a standard way to discover your service, register themselves, request approval, and execute scoped capabilities using short-lived signed JWTs. It comes with adapters for **OpenAPI** and **MCP** — so you can turn an existing REST API or MCP server into an agent-auth-enabled service without writing capabilities by hand.

## Features

- **OpenAPI adapter** — derive capabilities, input/output schemas, and a proxy `onExecute` handler directly from an OpenAPI 3.x spec
- **MCP adapter** — expose agent auth as MCP tools so any MCP-compatible AI agent can discover and call your capabilities
- Discovery document at `/.well-known/agent-configuration`
- Capability listing, description, and execution (optional per-capability `location` URLs)
- Delegated and autonomous agent modes
- Device authorization and CIBA approval flows
- Short-lived signed JWTs with replay protection
- Audit/event hooks for approvals, grants, and execution

## Installation

### Install the packages

#### npm

```
npm install @better-auth/agent-auth
```

#### pnpm

#### yarn

#### bun

Client and CLI packages (optional):

```
npm install @auth/agent @auth/agent-cli
```

### Add the plugin to your auth config

Start by defining the capabilities your service exposes and an `onExecute` handler that performs the action for an authenticated agent.

```
import { betterAuth } from "better-auth";
import { agentAuth } from "@better-auth/agent-auth"; 

export const auth = betterAuth({
  plugins: [
    agentAuth({ 
      providerName: "Acme", 
      providerDescription: "Acme project and deployment APIs for AI agents.", 
      modes: ["delegated", "autonomous"], 
      capabilities: [ 
        { 
          name: "deploy_project", 
          description: "Deploy a project to production.", 
          input: { 
            type: "object", 
            properties: { 
              projectId: { type: "string" }, 
            }, 
            required: ["projectId"], 
          }, 
        }, 
        { 
          name: "list_projects", 
          description: "List projects the current user can access.", 
        }, 
      ], 
      async onExecute({ capability, arguments: args, agentSession }) { 
        switch (capability) { 
          case "list_projects": 
            return [{ id: "proj_123", name: "marketing-site" }]; 
          case "deploy_project": 
            return { 
              ok: true, 
              projectId: args?.projectId, 
              requestedBy: agentSession.user.id, 
            }; 
          default: 
            throw new Error(\`Unsupported capability: ${capability}\`); 
        } 
      }, 
    }), 
  ], 
});
```

### Expose the discovery document

The plugin provides `auth.api.getAgentConfiguration()`, but you should expose it from your app root at `/.well-known/agent-configuration`.

```
import { auth } from "@/lib/auth";
import { NextResponse } from "next/server";

export async function GET() {
  const configuration = await auth.api.getAgentConfiguration();
  return NextResponse.json(configuration);
}
```

### Migrate the database

Run the migration or generate the schema to add the agent, host, grant, and approval tables.

#### migrate

#### npm

#### generate

```
npx auth migrate
```

#### pnpm

#### yarn

#### bun

### Optional: add the Better Auth client plugin

If you want type-safe access to the plugin endpoints from a Better Auth client, add the client plugin too.

```
import { createAuthClient } from "better-auth/client";
import { agentAuthClient } from "@better-auth/agent-auth/client"; 

export const authClient = createAuthClient({
  plugins: [
    agentAuthClient(), 
  ],
});
```

## How It Works

The Agent Auth flow usually looks like this:

1. An agent discovers your provider from `/.well-known/agent-configuration`
2. The agent lists capabilities and decides what it needs
3. The agent registers with your server and requests capability grants
4. Your user approves the request through device authorization or CIBA
5. The agent signs short-lived JWTs (with an `aud` that matches the URL it calls) and invokes each granted capability at **`default_location`** or at that capability’s own **`location`**, if you set one

## Discovery

The discovery document tells agents how to interact with your server. The plugin includes provider metadata, supported modes, approval methods, absolute endpoint URLs, and a **`default_location`** field.

Important fields for execution:

- **`issuer`** — The provider’s base URL (Better Auth `baseURL`).
- **`endpoints`** — Absolute URLs for each route (for example `execute` points at `POST /capability/execute` on that base).
- **`default_location`** — The full URL of the default execute endpoint. It always matches `endpoints.execute`. Agents use this as the JWT **`aud`** when a capability does not define a custom URL, and as the request URL for those capabilities.

Expose it from your app root:

```
import { auth } from "@/lib/auth";
import { NextResponse } from "next/server";

export async function GET() {
  const configuration = await auth.api.getAgentConfiguration();
  return NextResponse.json(configuration);
}
```

## OpenAPI Adapter

If your service already has an OpenAPI spec, you can turn the entire API into an agent-auth provider in a few lines. **`createFromOpenAPI`** reads the spec and produces everything the plugin needs: capabilities (one per `operationId`), input/output JSON Schemas, a proxy **`onExecute`** handler, and optionally **`providerName`** / **`providerDescription`** from `info`.

```
import { betterAuth } from "better-auth";
import { agentAuth } from "@better-auth/agent-auth";
import { createFromOpenAPI } from "@better-auth/agent-auth/openapi";

const spec = await fetch("https://api.example.com/openapi.json").then((r) =>
  r.json(),
);

export const auth = betterAuth({
  plugins: [
    agentAuth({
      ...createFromOpenAPI(spec, {
        baseUrl: "https://api.example.com",
      }),
    }),
  ],
});
```

That is all it takes. Every operation with an `operationId` in the spec becomes a capability whose name is that id. Path, query, and header parameters plus the JSON request body are merged into a single `input` schema, and the 200/201 response body becomes `output`.

### Upstream authentication

The proxy handler calls your upstream API on behalf of the agent. Use **`resolveHeaders`** to inject the credentials each request needs (for example an internal service token or a user-scoped access token looked up from `agentSession`).

```
createFromOpenAPI(spec, {
  baseUrl: "https://api.example.com",
  async resolveHeaders({ agentSession }) {
    const token = await getAccessToken(agentSession.user.id);
    return { Authorization: \`Bearer ${token}\` };
  },
});
```

### Default host capabilities

Control which capabilities are auto-granted to new hosts. You can pass `true` (all), a single HTTP method string, an array of methods, or a callback that receives the full runtime context.

```
createFromOpenAPI(spec, {
  baseUrl: "https://api.example.com",
  defaultHostCapabilities: ["GET", "HEAD"],
});
```

### Approval strength per method

Map HTTP methods to **`approvalStrength`** so mutating operations require stronger user verification (for example WebAuthn) while reads use a normal session.

```
createFromOpenAPI(spec, {
  baseUrl: "https://api.example.com",
  approvalStrength: {
    GET: "session",
    POST: "webauthn",
    PUT: "webauthn",
    DELETE: "webauthn",
  },
});
```

### Per-capability location

When you set **`location`**, every derived capability gets that URL. Agents call it directly (with the agent JWT) instead of going through the default execute endpoint—useful when you want the agent to hit the real API URL and handle the session in your own middleware rather than proxying through `onExecute`.

```
createFromOpenAPI(spec, {
  baseUrl: "https://api.example.com",
  location: "https://api.example.com/agent/execute",
});
```

### Using the pieces individually

If you only need part of the pipeline, the adapter also exports the lower-level helpers:

- **`fromOpenAPI(spec)`** — returns `Capability[]` only (no handler, no host caps).
- **`createOpenAPIHandler(spec, opts)`** — returns only the `onExecute` proxy handler so you can pair it with hand-written capabilities or filter the spec yourself.

```
import {
  fromOpenAPI,
  createOpenAPIHandler,
} from "@better-auth/agent-auth/openapi";

const capabilities = fromOpenAPI(spec);
const onExecute = createOpenAPIHandler(spec, {
  baseUrl: "https://api.example.com",
});

agentAuth({ capabilities, onExecute });
```

## Capabilities

Capabilities are the contract between your application and an agent. Each capability has a name, a description, and optionally a JSON Schema `input` definition.

By default, agents call **`default_location`** from discovery (the execute URL) and the plugin runs **`onExecute`**. If you set **`location`** on a capability, agents call that absolute URL instead—for example an existing REST route—and **`onExecute` is not used** for those requests; you resolve the agent session with the helpers below and implement the handler yourself.

Use capabilities to expose narrow, reviewable actions instead of broad API access.

### Define capabilities

```
agentAuth({
  capabilities: [
    {
      name: "create_issue",
      description: "Create an issue in the current workspace.",
      input: {
        type: "object",
        properties: {
          title: { type: "string" },
          body: { type: "string" },
        },
        required: ["title"],
      },
    },
  ],
});
```

Optional **`location`** — agents call this URL with the agent JWT instead of the default execute URL:

```
{
  name: "create_issue",
  description: "Create an issue in the current workspace.",
  location: "https://api.example.com/v1/issues",
}
```

### Default execute vs custom location

- **No `location`** — Agents `POST` to **`default_location`** (`endpoints.execute`) with `{ capability, arguments }`. After the plugin validates the JWT and grant, it runs **`onExecute`**.
- **With `location`** — Agents call that URL (your REST handler, another service, an OpenAPI operation URL, etc.). **`onExecute` does not run** for that call. Resolve **`agentSession`** in your handler using the helpers below, then enforce grants and your business logic.

### Agent session outside onExecute

For custom **`location`** routes (or any non-execute handler), the agent still sends an **`Authorization: Bearer`** header with the agent JWT. Whatever framework you use, take the incoming **`Headers`** (e.g. **`request.headers`**, or your runtime’s equivalent) and pass them through—the verification path is the same: signature, **`aud`**, replay (**`jti`**), expiry, and (when present) request-binding claims.

**`auth.api.getAgentSession({ headers })`** runs that flow in-process and returns **`AgentSession`** or **`null`**. **`verifyAgentRequest(request, auth)`** does the same by forwarding the **`Request`** ’s headers to **`GET /agent/session`** via **`auth.handler`** —pick whichever fits your code shape; there is no Hono-vs-Next split, only “headers in, session out.”

```
import { auth } from "@/lib/auth";

export async function POST(request: Request) {
  const agentSession = await auth.api.getAgentSession({
    headers: request.headers,
  });
  if (!agentSession) {
    return new Response("Unauthorized", { status: 401 });
  }
  // Check grants, enforce constraints, run your handler…
}
```

```
// Equivalent when you already have \`Request\` + \`auth\` and prefer a helper:
import { verifyAgentRequest } from "@better-auth/agent-auth";
const agentSession = await verifyAgentRequest(request, auth);
```

**Checking grants and inputs**

After you have **`agentSession`**, inspect **`agentSession.agent.capabilityGrants`**. These are **active** DB grants **intersected** with the JWT’s **`capabilities`** claim (same as execute). For the capability this route implements, ensure there is a matching grant:

```
const CAP = "create_issue";
const allowed = agentSession.agent.capabilityGrants.some(
  (g) => g.capability === CAP && g.status === "active",
);
if (!allowed) {
  return new Response("Forbidden", { status: 403 });
}
```

If that grant has **`constraints`**, validate the request body or query the same way **`POST /capability/execute`** would—otherwise a client could bypass constraints by calling your custom URL. The plugin does not re-run execute’s constraint helpers on arbitrary routes; that logic stays in your handler (or call into shared code you extract from your **`onExecute`** path).

**What you get on the session**

- **`agentSession.user`** — Resolved user for the agent (delegated host user or **`resolveAutonomousUser`**).
- **`agentSession.agent`** — Id, name, mode, **`capabilityGrants`**, host id, metadata.
- **`agentSession.host`** — Host record when the agent is linked to a host.

Types are exported from **`@better-auth/agent-auth`** (for example **`AgentSession`**).

### JWT audience (aud)

The JWT **`aud`** must match what the server expects for the URL being called:

- **No per-capability `location`** — Use **`default_location`** / **`endpoints.execute`**, or issuer / base URL values the plugin already allows.
- **With `location`** — **`aud`** should be that same absolute URL. `GET /capability/list` includes `location` when set. Invalid `location` values in config fail at startup.

**Single capability in the JWT** — If `capabilities` lists exactly one id, **`aud`** may equal that capability’s **`location`** when set.

**Multiple capabilities in the JWT** — Per-capability **`location`** values are not accepted as **`aud`**; use the issuer, base path, or default execute endpoint instead.

Behind a reverse proxy, set **`trustProxy`** if you need **`Host`** / **`X-Forwarded-Proto`** to line up with **`aud`** validation.

### Filter visible capabilities

Use `resolveCapabilities` to show different capability sets to different callers, such as plan-gated, user-specific, or organization-specific capabilities.

### onExecute

Runs for capabilities that use the **default execute URL** (no per-capability **`location`**). The plugin verifies the JWT (including **`aud`**), attaches **`agentSession`**, checks the grant, then calls **`onExecute`**. Capabilities with a custom **`location`** never hit this path—you handle them in your own route using the session helpers above.

```
agentAuth({
  capabilities: [
    {
      name: "create_issue",
      description: "Create an issue in the current workspace.",
    },
  ],
  async onExecute({ capability, arguments: args, agentSession }) {
    if (capability !== "create_issue") {
      throw new Error("Unsupported capability");
    }

    return {
      ok: true,
      title: args?.title,
      createdBy: agentSession.user.id,
    };
  },
});
```

## Approval Flows

The plugin supports two approval methods:

- `device_authorization` for browser-based approval with a user code
- `ciba` for backchannel approval flows

By default, both are enabled. You can restrict or customize them with `approvalMethods` and `resolveApprovalMethod`.

```
agentAuth({
  approvalMethods: ["ciba", "device_authorization"],
  resolveApprovalMethod: ({ preferredMethod, supportedMethods }) => {
    if (preferredMethod && supportedMethods.includes(preferredMethod)) {
      return preferredMethod;
    }
    return "device_authorization";
  },
  deviceAuthorizationPage: "/device/capabilities",
});
```

## Events and Auditing

Use `onEvent` to capture important lifecycle events such as:

- agent creation and revocation
- host creation and enrollment
- capability requests and approvals
- capability execution

This hook is a good place to write audit logs or feed analytics pipelines.

## Configuration

The Agent Auth plugin supports many options. These are the ones you will usually start with:

Prop
